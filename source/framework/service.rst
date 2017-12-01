服务
****

服务创建
========

- 服务目录结构 ::

    project
    `- Service
        |- ServiceName
        |   `- Service.php
        `- Example
            `- Service.php

- 创建存放服务的目录，并将该目录添加到配置文件的 ``service_path`` 中。
- 创建模块文件夹，文件夹名称与模块名称相同。
- 创建模块类文件，文件名固定为 ``Service.php`` ，其内容如下 ::

    <?php
    namespace X\Service\Example;
    
    class Service extends \X\Core\Service\XService {
        /** 定义服务名称 */
        protected static $serviceName = 'Example';
    
        /** 定义服务接口 */
        public function doSomething() {
            return "something";
        }
    }

- 注册服务，将服务信息写入 ``services`` ::

    'services' => array(
        'Example' => array(
            'class' => '\\X\\Service\\Example\\Service',
            'enable' => true,
            'delay' => true,
            'params' => array(
                'param001' => 'value of param001',
            ),
        ),
    )

  ``params`` 可以为空，这里是方便演示所以加了一个参数。


调用服务
========

  在项目运行时，服务只会被实例化一次，可以通过 ``getService()`` 方法获取服务实例。 ::

    echo X\Service\Example\Service::getService()->doSomething();

  将输出 ``something`` 。

``start()`` 与 ``stop()`` 方法
==============================

  如果希望在服务启动/停止的时候执行一些操作， 则需要重写 ``start()`` 或者 ``stop()`` 方法。 
  例如， 如果我们想知道请求大概运行了多久。 ::

    class Service extends \X\Core\Service\XService {
        /** 记录开始时间 */
        private $startTime = null;
        
        /** 服务启动时记录开始时间 */
        public function start() {
            parent::start(); 

            $this->startTime = time();
        }

        /** 服务结束的时候输出花费的时间 */
        public function stop() {
            echo time() - $this->startTime();

            parent::stop();
        }
    }

  当然， 这需要在注册服务的时候，将 ``delay`` 设置为 ``false`` 来禁用掉延迟加载。

