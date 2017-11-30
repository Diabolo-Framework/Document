全局配置
********

范例
====

基础配置 ::

    array(                              # 配置运行信息
        'document_root' => __DIR__,     # 项目根路径
        'module_path' => array(),       # 模块搜索路径
        'service_path' => array(),      # 服务搜索路径
        'library_path' => array(),      # 公共库搜索路径
        'modules' => array(),           # 引用模块列表
        'services' => array(),          # 引用服务列表
        'params' => array(),            # 其他自定义参数
    );

项目说明 
========

- ``document_root``  
  
  项目根目录路径， 当使用 ``X::system()->getPath()`` 的时候会依据该目录计算出绝对路径。
  例如，设置 ``document_root`` 为 **/myproject/demo/** ::

    echo X\Core\X::system()->getPath("View/Index.php");
  
  将会输出 ::
  
    /myproject/demo/View/Index.php

- ``module_path``
  
  设置模块搜索路径，当加载模块的时候将在这些目录中寻找模块。
  列表中的路径地址必须是绝对路径， 并且可以不在当前项目中。
  例如::

    'module_path' => array(
        '/myproject/demo/Module',
        '/usr/local/diabolo/Module',
    )

  这样， 框架将会按照上面的顺序寻找模块，找到后将停止寻找。

- ``service_path``

  设置服务搜索路径，当加载服务的时候将在这些目录中寻找服务。
  列表中的路径地址必须是绝对路径， 并且可以不在当前项目中。
  例如::

    'service_path' => array(
        '/myproject/demo/Service',
        '/usr/local/diabolo/Service',
    )

  这样， 框架将会按照上面的顺序寻找服务，找到后将停止寻找。

- ``library_path``

  设置第三方库的搜索路径，当加载lib的时候将在这些目录中寻找。
  列表中的路径地址必须是绝对路径， 并且可以不在当前项目中。
  例如::

    'library_path' => array(
        '/myproject/demo/Library',
        '/usr/local/diabolo/Library',
    )

  这样， 框架将会按照上面的顺序寻找库，找到后将停止寻找。

- ``modules``
  
  配置项目运行所需的模块列表。
  例如::

    'modules' => array(
        'Demo' => array(
            'enable' => true,
            'default' => true,
            'params' => array(
                'param001' => 'value of param001',
            ),
        ),
    )
  
  模块数组的键名是模块名

    * ``enable`` : true|false 配置是否启用该模块
    * ``default`` : true|false 配置是否为默认模块，当参数没有指定运行参数的时候，默认的模块将会被运行
    * ``params`` : 模块的配置项目, 在模块的代码中可以使用 ``\X\Module\Demo::getModule()->getConfiguration()->get("param001")`` 获取值 **"value of param001"**

- ``services``

  配置项目运行所需的服务列表
  例如::

    'services' => array(
        'Demo' => array(
            'class' => '\\X\\Service\\Demo\\Service',
            'enable' => true,
            'delay' => true,
            'params' => array(
                'param001' => 'value of param001',
            ),
        ),
    )

  服务数组的键名是服务名

    * ``class`` 服务类名称
    * ``enable``  true|false 配置是否启用该服务
    * ``delay`` true|false 是否延迟加载， 延迟加载将在仅仅服务调用的时候加载服务，如果没有调用，则服务不会被启动。如果为false，则当项目启动的时候自动加载该服务。
    *  ``params`` : 服务的配置项目, 在服务的代码中可以使用 ``\X\Service\Demo::getService()->getConfiguration()->get("param001")`` 获取值 **"value of param001"**

- ``params``

  全局配置, 在代码中可以使用 ``\X\Core\X::system()->getConfiguration()->get("params")->get("param001")`` 获取值 **"value of param001"**
  例如::

    'params' => array(
        'param001' => 'value of param001',
    ),

