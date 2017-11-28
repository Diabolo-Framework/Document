开始使用
********

准备
====
- 进入到项目根目录
- 创建Demo模块::

    $ mkdir Module       # 创建用于存放模块的目录
    $ mkdir Module/Demo  # 创建Demo模块目录
    $ vi Module/Demo/Module.php # 编辑模块类 

  文件内容::

    <?php
    namespace X\Module\Demo; # 模块命名空间
    use X\Core\Module\XModule; # 引入模块基础类
    class Module extends XModule {
        # 当该模块运行的时候，将会调用该方法
        public function run($parameters = array()) {
            echo "This is a demo module.";
        }
    }

- 创建启动文件，例如index.php， 然后在该文件中初始化并运行::

    <?php
    define("DIABOLO_PATH", __DIR__);    # 定义框架路径
    require DIABOLO_PATH.'/Core/X.php'; # 导入框架
    $config = array(                    # 配置运行信息
        'document_root' => __DIR__,     # 项目根路径
        'module_path' => array(),       # 模块搜索路径
        'service_path' => array(),      # 服务搜索路径
        'library_path' => array(),      # 公共库搜索路径
        'modules' => array(             # 引用模块列表
            'Demo' => array(            # 定义模块
                'enable' => true,       # 启用模块
                'default' => true,      # 设置为默认模块
                'params' => array(),    # 模块配置参数
            ),
        ),
        'services' => array(),          # 引用服务列表
        'params' => array(),            # 其他自定义参数
    );
    X\Core\X::start($config)->run();    # 启动并运行

- 最终的目录结构::

    project
    | -- Core               # Diabolo存放目录
    ` -- Module             # 模块存放目录
         ` - Demo           # Demo 模块目录
             ` - Module.php # 模块主文件

启动
====
使用浏览器访问项目地址 ``http://127.0.0.1/`` ，输出::

    This is a demo module.

