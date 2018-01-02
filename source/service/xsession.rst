会话管理服务
************

注册服务
========

  配置内容 ::

    'services' => array(
        'XSession' => array(
            'enable' => true,
            'class' => 'X\\Service\\XSession\\Service',
            'delay' => false,
            'params' => array(
                'autoStart' => true,
                'name' => 'My-Custom-Session-Name',
                'holders' => array('cookie', 'get', 'post', 'request'),
                'cookie' => array(
                    'lifetime'=>3600,
                    'path'=>'/',
                    'domain'=>'',
                    'secure'=> false,
                    'httponly'=>false
                ),
                'storage' => null,
            ),
        ),
    ),

- ``autoStart`` 是否自动启动，默认为 ``true`` ，自动启动时将在服务启动的时候自动启动会话，
  不用手动调用 ``seesion_start()`` 或者 ``$service->startSession()`` 来启动。

- ``name`` 会话参数名称，说明会话在初始化时从哪个参数获取会话id。如果你需要获取上传文件的进度信息， 你需要将该名称同步到php.ini或者apache或者其他web服务器的配置文件中，否则无法获取到文件上传信息(@see http://php.net/manual/en/session.upload-progress.php#119631)。

- ``holders`` 这个指明会话ID从哪里获取去，如果没有设置将采用PHP.ini中的设置, 在配置中的变量中， 第一个找到的会话ID将会被使用。

- ``cookie`` 配置cookie信息。 

- ``storage`` 配置会话存储方式，如果为null, 将使用默认的存储方式。 目前支持以下的存储方式 :

  * mongodb 使用 mongodb 存储， 配置如下 ::

        # mongodb
        'storage' => array(
            'type' => 'mongodb',
            # 数据库链接
            'uri' => 'mongodb://127.0.0.1:27017',
            #'uri' => 'mongodb://username:password@host:port'
            # 数据库名称
            'database' => 'diabolo',
            # collection 名称
            'collection' => 'sessions',
            # 有效时间秒数
            'lifetime' => 3600,
        ),

  * radis 使用 redis 存储， 配置如下 ::

        # redis
        'storage' => array(
            'type' => 'redis',
            # 服务器地址
            'host' => '127.0.0.1',
            # 端口号
            'port' => 6379,
            # 数据库索引号
            'database' => 1,
            # 链接密码
            'password' => 'redis-password',
            # 有效期
            'lifetime' => 3600,
            # 键名前缀
            'prefix' => 'SESSION:',
        ),

  * memcached 使用memcached 存储， 配置如下 ::

        # memcached
        'storage' => array(
            'type' => 'memcached',
            # memcached地址
            'host' => '127.0.0.1',
            # 端口
            'port' => 11211,
            # 键名前缀
            'prefix' => 'SESSION:',
            # 过期时间秒数
            'lifetime' => 3600,
        ),

  * database 使用数据库存储， 配置如下 ::

        'storage' => array(
            'type' => 'database',
            # 数据库链接
            'dsn' => 'mysql:host=database.host;port=3306;dbname=databasename',
            # 会话存储使用的表名
            'table' => 'sessions',
            # 数据库用户名
            'user' => 'username',
            # 数据库密码
            'password' => 'password',
            # 在写入表时，处理额外的存储字段
            'serializeHandler' => array(SessionSerializeHandler::class, 'serialize'),
            # 会话有效期的秒数
            'lifetime' => 3600
        ),
    
    数据库表结构 ::

        CREATE TABLE `sessions` (
            `ID`  varchar(64) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL , -- 必须字段， 用于存储会话id
            `EXPIRED_AT`  datetime NOT NULL , -- 必须字段，用于存储会话过期时间
            `RAW`  text CHARACTER SET utf8 COLLATE utf8_general_ci NULL , -- 必须字段， 用于存储会话内容
            `USER_ID`  int(11) NULL DEFAULT NULL , -- 自定义字段， 由 serializeHandler 处理后的数据信息
            PRIMARY KEY (`ID`)
        )
        ENGINE=InnoDB
        DEFAULT CHARACTER SET=utf8 COLLATE=utf8_general_ci
        ROW_FORMAT=DYNAMIC;

    ``serializeHandler`` 用于在存储会话的时候为额外的字段赋值 ::

        class SessionSerializeHandler {
            public static function serialize( ) {
                return array(
                    'USER_ID' => 100,
                );
            }
        }

会话管理
========

- ``startSession()``  启动会话
- ``close($save=ture)`` 关闭当前会话，默认保存变更的会话信息
- ``destory()`` 销毁当前会话
- ``clean($lifetime=0)`` 清理过期的会话数据

Flash 使用
==========

flash 在存储方式上和普通会话变量没有区别，但是，在获取指定flash之后， 该flash将会被自动删除。 ::

    $service = Service::getService();
    $service->flashAdd('demo-flash', "This is a demo flash");
    $service->flashHas('demo-flash'); # true
    $content = $service->flashGet('demo-flash'); 
    echo $content; # "This is a demo flash"
    $service->flashHas('demo-flash'); # false

