数据库服务
**********

注册服务
========

配置内容 ::

    'services' => array(
        'XDatabase' => array(
            'class' => '\\X\\Service\\XDatabase\\Service',
            'enable' => true,
            'delay' => true,
            'params' => array(
                'migration_table_name' => 'xdatabase_dbmigration_histories',
                'databases' => array (
                    'example' => array (
                        'dsn' => 'mysql:host=localhost;dbname=suanhetao-service-view',
                        'username' => 'username',
                        'password' => 'password',
                        'charset' => 'UTF8',
                    ),
                ),
            ),
        ),
    ),

初始化服务
==========

数据库服务不用初始化，当配置成功后即可直接使用。

SQL 创建器
==========

服务允许直接以字符串的形式执行SQL语句， 例如 ::

    $sql = "DROP TABLE example";
    $isSuccess = X\Service\XDatabase\Service::getService()->get()->exec($sql);

在不考虑更换数据库类型的情况下，这种方式是最好的。 但是如果后期数据库从 mysql
迁移到 sqlite 后，就会存在sql语句不兼用的问题，这个时候就推荐使用 ``SQLBuilder``
来创建SQL语句。 虽然 ``SQLBuilder`` 不能完全解决SQL不兼容的问题，但是能够减轻
一部分的迁移工作， 尤其是字段，表名的转义等。 例如 ::

    Mysql : SELECT `from` FROM `table`

    SQLITE : SELECT [from] FROM [table]

SQLBuilder 的目的不仅仅是解决上面的问题， 对于部分内置函数， SQLBuilder提供了统一
的调用方式，不用再关心切换数据库类型后的迁移方式。

使用SQLBuilder创建查询语句时，和普通的字符串方式不相同的是可变部分作为参数来实现， 
例如 ::

    $dbname = 'example';
    $sql = \X\Service\XDatabase\Core\SQL\Builder::build($dbname)->select()
          ->expression('name', 'nickname')
          ->expression('age', 'realage')
          ->expression('sex')
          ->from('example_students')
          ->where(array(
              'name' => 'michael',
              'sex' => 'm',
          ))
          ->offset(10)
          ->limit(10)
          ->orderBy('age', 'DESC')
          ->toString();

最终输出的SQL语句为 ::

    SELECT 
      `name` AS `nickname`, 
      `age` AS `realage`, 
      `sex`
    FROM `example_students` 
    WHERE 
      `name`='michael' 
      AND `sex` = 'm' 
    ORDER BY `age` DESC 
    OFFSET 10 
    LIMIT 10

SQLBuilder 在生成SQL语句时传入的数据库名 ``$dbname`` 是用来告诉builder使用哪个
数据来生成， 因为不同的数据库引擎最终生成的SQL语句是不同的，默认情况下SQLBuilder
使用当前的数据库来生成。

Active Record
=============

AR 是将数据表中的数据映射成一个对象，对该对象修改后将会同步到数据库中。 例如 ::

    use X\Module\Example\Model\User;
    
    # 通过主键查询
    $user = User::model()->findByPrimaryKey(1);

    # 通过属性查询
    $user = User::model()->find(array('name'=>'michael'));

    # 通过属性查询多条
    $users = User::model()->findAll(array('sex'=>'m'));

    # 修改
    $user->name = 'new-name';
    $user->save();

    # 创建
    $user = new User();
    $user->name = 'another michael';
    $user->save();

    # 删除
    $user = User::model()->findByPrimaryKey(1);
    $user->delete();

AR类的定义 ::

    <?php 
    namespace X\Module\Example\Model;
    use X\Service\XDatabase\Core\ActiveRecord\XActiveRecord;

    /**
     * @property string $name
     * @property string sex 
     */
    class User extends XActiveRecord {
       /**
        * 设置数据库配置名称，表明该AR类操作时使用的数据库链接
        */
        public function getDatabaseName() {
            return 'example';
        }

       /**
        * 配置属性描述信息
        */
       protected function describe() {
           return array(
               'name' => 'VARCHAR (256)',
               'sex' => 'VARCHAR (1)',
           );
       }
    
       /**
        * 配置数据表名称
        */
       protected function getTableName() {
           return 'users';
       }
    }



表管理
======

Table Manager 是进行表管理的快捷方式， 可以在不用写SQL语句的情况下对表数据或者结构进行操作。 
例如 ::

    use X\Service\XDatabase\Core\Table\Manager;
    
    $tableName = 'example_table';
    
    # 删除表
    Manager::open($tableName)->drop();
    
    # 更新表数据
    Manager::open($tableName)->truncate();
    
    # 更新表数据
    Manager::open($tableName)->update('new-value', array('sex'=>'m'), 10, 5);
    
    # 解锁表
    Manager::open($tableName)->unlock();
    
    # 重命名数据表
    Manager::open($tableName)->rename('new_table_name');
    
    # 锁表
    Manager::open($tableName)->lock('READ');
    
    # 插入一条数据
    Manager::open($tableName)->insert(array('id'=>1,'name'=>'michael'));
    
    # 获取表信息
    $tableInfo = Manager::open($tableName)->getInformation();
    
    # 创建表 
    Manager::create(
        'another_table',                   # 表名
        array('id'=>'int', 'age'=>'int'),  # 表列定义
        'id'                               # 主键
    );

在操作表的时候， 如果表不存在， 则只能调用 ``create()`` 方法创建表之后继续操作， 
如果表存在， 则需要调用 ``open()`` 方法打开表后再操作。

