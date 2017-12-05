动作服务
********

动作服务用于从请求参数中获取动作名称，然后从一组动作中实例化动作并执行的服务。

注册服务
========

配置内容::

    'services' => array(
        'XAction' => array(
            'class' => '\\X\\Service\\XAction\\Service',
            'enable' => true,
            'delay' => true,
            'params' => array(
                'ActionParamName' => 'action',
                'Error404Handler' => array(Error404::class, 'handle404Error'),
                'Error500Handler' => array(Error500::class, 'handle500Error'),
                'CommonViewPath' => 'View',
            ),
        ),
    ),

- ``ActionParamName`` 动作参数名，用来告诉服务从哪里获取动作名称， 
  例如这里为 ``action`` , 那么，当web请求的参数中包含 ``action`` 的时候，
  服务将从取该参数的值作为动作名称。 当请求来自命令行时， 参数来自 ``--action`` 。 
  例如 ::

    GET http://example.com/index.php?module=example&action=show/hello

  或者 ::
 
    php index.php --module=example --action=show/hello

  都将调用 ``show/hello`` 动作。

- ``Error404Handler`` 404错误处理器， 取值可以为普通字符串或者回调函数， 
  如果配置为字符串，那么当发生404错误的时候，则显示该字符串; 
  如果该值能够被调用， 则404错误发生的时候执行该回调。
  404能够在任何地方触发，例如 ::

    \X\Service\XAction\\Service::getService()->triggerErrorHandler('404');

  不过，推荐仅仅在动作中触发， 例如 ::

    $this->throw404IfTrue(true);

- ``Error500Handler`` 500错误处理器, 取值可以为普通字符串或者回调函数， 
  如果需要禁用该功能，则赋值为 ``null`` 。
  当该值不为 ``null`` 时，服务将注册一个错误处理器，并在发生错误的时候调用该值。

- ``CommonViewPath`` 视图放置路径，该路径是相对于模块或者整个项目的。 
  这里配置为 ``View`` , 则项目目录如下 ::

    project
    |- Module
    |   `- Example
    |       `- View
    |           |- Particle
    |           |   `- Index.php
    |           `- Layout
    |
    `- View
       | - Particle
       |    `- Header.php
       ` - Layout
           `- Default.php

初始化服务
==========

动作服务的初始化主要是向服务中添加分组，一般该操作在模块类中进行， 例如 ::

    <?php
    namespace X\Module\Example;
    use X\Core\Module\XModule;
    use X\Service\XAction\Service as XActionService;

    /**
     * 模块基类, 当前项目的所有模块都应当继承该类。
     * <li>动作类都放到 Action 文件夹</li>
     * <li>模块私有视图文件都放到 View 文件夹， 布局存放在 View/Layout, 片段存放在 View/Particle 下面</li>
     */
    class Module extends XModule {
        /**
         * 使用 XAction 服务来分发动作。
         */
        public function run($parameters = array()) {
             $actionService = XActionService::getService();
             
             # 组名必须唯一
             $group = 'example';
             # 注册分组
             $actionService->addGroup($group, '\\X\\Module\\Example\\Action');
             # 设置该分组的默认动作
             $actionService->setGroupOption($group, 'defaultAction', 'index');
             # 设置分组的视图路径
             $actionService->setGroupOption($group, 'viewPath', $this->getPath('View/')); 
             # 设置运行参数
             $actionService->getParameterManager()->merge($parameters);
             # 运行分组
             return $actionService->runGroup($group);
       }
    }

- 组名 组名在动作分组中需要唯一
- 注册分组时的命名空间名称是动作的基础命名空间名称， 例如动作为 ``user/login`` ，则最终的动作类为 ``\X\Module\Example\Action\User\Login`` 。
- 分组的默认动作是当参数中没有指定动作名称的时候执行的动作。
- 视图路径是指该分组的视图路径。
- 当运行分组时，服务将在指定的分组中寻找动作并执行。

实现动作
========

动作类是实际用来处理请求的操作类。
例如 ::

    <?php 
    namespace X\Module\Example\Action\User;
    use X\Service\XAction\Handler\WebPageAction;

    class Index extends WebPageAction {
        /** 布局 */
        protected $layout = '/SingleColumn';
        /** 标题 */
        protected $title = '用户登录';
        
        # 该方法用于接收参数并执行动作处理
        protected function runAction( $userName ) {
            $this->addParticle('/header');

            $this->addParticle('login', array(
                'default' => $userName,  # 视图数据
            ));

            $this->addParticle('/footer');
            $this->display();
        }
    }

- 该动作继承 ``WebPageAction`` 用于处理一个网页请求并渲染界面。 
  目前支持 ``WebPageAction`` , ``CommandAction`` 和 ``AjaxAction`` 这三类动作。

- 对于视图的名称， 例如 ``/SingleColumn`` 或者 ``/header``, ``login`` 之类，
  如果以 ``/`` 开头，则将会使用服务中配置的 ``CommonViewPath`` 来获取， 否则
  使用该动作所在分组的视图路径来获取。

- ``runAction()`` 方法用于最终的动作处理， 该方法的参数值将在初始化动作服务时获取。
  以之前的初始化方式为例, 如果要获取 ``$userName`` 的值， 则请求应该如下 ::
  
    GET http://example.com/index.php?module=example&action=user/login&userName=example

  当方法的参数没有默认值的时候，表示该参数必传否则将会出错。 有默认值的时候，如果
  请求中不带该参数，则取默认值。


Web视图
=======

web视图为php文件，视图分为布局和片段，布局用来展示网页的大体概貌， 片段是整个视图的一小部分，
并且由布局视图来管理在哪里显示。
web视图最终输出的内容是 ``html`` 中的 ``body`` 部分， ``head`` 部分由视图类来管理。


在布局文件中， ``$this`` 是 ``X\Service\XAction\Component\WebView\Html`` 的实例。
在片段文件中， ``$this`` 是 ``X\Service\XAction\Component\WebView\ParticleView`` 的实例。

普通视图文件 ::

    <div>
        当前时间为：<?php echo date('Y-m-d H:i:s'); ?>
    </div>

在布局文件中，通过以下方式增加资源 ::

    <?php
    /* @var $this \X\Service\XAction\Component\WebView\Html */
    
    /* @var $link \X\Service\XAction\Component\WebView\LinkManager */
    $link = $this->getLinkManager();
    
    # 注册CSS文件
    $link->addCSS('bootstrap', '/lib/bootstrap/dist/css/bootstrap.css');

    /* @var $script \X\Service\XAction\Component\WebView\ScriptManager */
    $script = $this->getScriptManager();
    # 注册 Js 文件
    $script->add('jquery', '/lib/jquery/dist/jquery.min.js');

在片段文件中，通过以下方式增加资源 ::

    <?php
    /* @var $this \X\Service\XAction\Component\WebView\ParticleView */
    $html = $this->getManager()->getParent();

    /* @var $link \X\Service\XAction\Component\WebView\LinkManager */
    $link = $html->getLinkManager();
    
    # 注册CSS文件
    $link->addCSS('bootstrap', '/lib/bootstrap/dist/css/bootstrap.css');

    /* @var $script \X\Service\XAction\Component\WebView\ScriptManager */
    $script = $html->getScriptManager();
    # 注册 Js 文件
    $script->add('jquery', '/lib/jquery/dist/jquery.min.js');


