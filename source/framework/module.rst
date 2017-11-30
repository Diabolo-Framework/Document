模块
****

创建模块
========

- 模块目录结构 ::

    project
    `- Module
        |- ModuleName
        |   `- Module.php
        `- Example
            `- Module.php 

- 创建存放模块的目录，并将该目录添加到配置文件的 ``module_path`` 中。
- 创建模块文件夹，文件夹名称与模块名相同。
- 创建模块类文件，文件名固定为 ``Module.php`` , 其内容如下::

    <?php
    namespace X\Module\Example;
    use X\Core\Module\XModule;
    
    class Module extends XModule {
        /**
         * {@inheritDoc}
         * @see \X\Core\Module\XModule::run()
         */
        public function run($parameters = array()) {
            echo "This is an example module.";

        }
    }

  模块的实现类必须要定义 ``run()`` 方法，该方法是模块运行时执行实际的请求处理。 
  比如根据参数调用操作处理方法，然后渲染视图输出等。

- 注册模块，将模块信息写入 ``modules`` ::
    
    'modules' => array(
        'Example' => array(
            'enable' => true,
            'default' => true,
            'params' => array(
                'pretty_name' => '测试模块',
            ),
        ),
    ),

  ``params`` 可以为空，这里是方便演示所以加了一个参数。

运行模块
========
  
  框架通过获取 ``module`` 参数的值来加载指定的模块并运行，
  例如 ::
  
    web 方式 : http://example.com/index.php?module=example
    cli 方式 : php index.php --module=example
  
  上面两种方式最终都会输出 ``This is an example module.`` .

获取模块信息
============

- 获取模块下文件路径 ::

    # 在 模块类 中
    echo $this->getPath("Resource/Template.txt");
    
    # 在其他文件中
    \X\Module\Example\Module::getModule()->getPath("Resource/Template.txt");

  假如项目地址为 ``/home/project/example`` , 那么输出的路径为 ::

    /home/project/example/Module/Resource/Template.txt

- 获取模块的配置信息 ::

     # 在 模块类 中
     echo $this->getConfiguration()->get("pretty_name");

     # 在其他文件中
     \X\Module\Example\Module::getModule()->getConfiguration()->get("pretty_name");

  将输出 ``测试模块``

