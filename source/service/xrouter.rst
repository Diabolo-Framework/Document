路由服务
********

注册服务
========

配置内容 ::

    'XRouter' => array(
        'class' => '\\X\\Service\\XRouter\\Service',
        'enable' => true,
        'delay' => false,
        'params' => array(
            'router' => XActionRouter::class,
            'fakeExt' => 'html',
        ),
    ),

- ``fakeExt`` 虚拟扩展名，用于在生成URL或者路由的时候添加或者去掉后缀。
- ``router`` 路由处理类名称，目前支持以下几种路由 :

  * ``X\Service\XRouter\Router\MapUrlRouter`` URL映射方式， 配置如下 ::
      
      'router' => X\Service\XRouter\Router\MapUrlRouter::class,
      'regexAlias' => array(
          'module' => '\w*?',
          'action'=>'\w*?',
          'id' => '[a-z0-9]{16}',
          'example' => '.*?',
      ),
      'rules' => array(
          # 请求URL => 路由后的URL
          '/{module}_{id}/{action}$' => 'module={module}&action={action}&{module}={id}',
          '/{module:$example}/' => 'module={module}&action=index', 
          '/{module:@\w}/{target:$id}' => 'module={mdoule}&action=detail&{module}={id}', 
          '/help'=>'index.php?module=dionysos&action=help',
      ),

    - ``regexAlias`` 正则表达式匹配列表， 用于在规则中使用
    - ``rules`` 匹配规则, 在规则中，使用 ``{`` 和 ``}`` 包含起来的将会被转换为正则表达式用于匹配url， 
      格式为: ``{anytext}``, ``{anytext:$alias}``, ``{anytext:@regex}`` 这几种。
      
      在格式 ``{anytext}`` 中， 如果 ``anytext`` 是 ``regexAlias`` 中的键名， 则该格式效果同 ``{anytext:$alias}``

  * ``X\Service\XRouter\Router\XActionRouter`` XAction服务方式, 配置如下 ::

      'router' => X\Service\XRouter\Router\XActionRouter::class,
      'fakeExt' => 'html',
      'mainModuleName' => 'mainmodule',
      'hideMainModuleName' => true,
      'defaultAction' => 'index',
    
    - ``mainModuleName`` 主模块名称
    - ``hideMainModuleName`` 是否隐藏主模块名称，例如 ``/mainmodule/index`` 将会变为 ``/index``, 隐藏了主模块的名称。
    - ``defaultAction`` 默认的动作名称，当解析不到动作的时候将会被使用。

    XAction路由不强制要求启用XAction服务， 该路由器匹配和生成URL基于以下规则 
   
    - 模块和动作将会组成请求的路径， 例如 : ``/module/action/path``
    - 如果在参数列表中包含与路径相同名称的参数， 则会被链接到路径中， 
      例如 : ``index.php?module=module&action=action/path&module=001`` => ``/module-001/action/path``
    - 不在请求路径的参数将会原封不动的添加到url中，
      例如 : ``index.php?module=main&action=search&text=123`` => ``/main/search?text=123``

生成URL
=======

当路由器使用的是XActionRouter时，该路由支持根据原生url生成格式化后的url。
例如 ::

    # fakeExt = html
    use X\Service\XRouter\Router\XActionRouter;
    XActionRouter::generate("/index.php?module=food&action=user/partner/edit&food=123&partner=426&from=team")
    # 输出 : food-123/user/partner-426/edit.html?from=team
    
    XActionRouter::action('food','user/partner/edit', array('food'=>123,'partner'=>4564,'xxx'=>222))
    # 输出 : food-123/user/partner-4564/edit.html?xxx=222

