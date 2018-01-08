日志服务
********

注册服务
========

  配置内容 ::

    'services' => array(
        'XLog' => array(
            'enable' => true,
            'class' => XLogService::class,
            'delay' => false,
            'params' => array(
                'level' => 'trace',
                'cache' => true,
                'handler' => 'file',
                /* 其他配置 */
            ),
        ),
    ),

- ``level`` 日志记录级别，当程序中记录的日志级别低于该配置项目的值时，日志内容将直接丢弃。目前支持以下几种, 以取值从低到高的顺序排列 

  * ``trace`` 当设置该级别的时候， 所有级别的日志都将记录。
  * ``debug`` 调试
  * ``info``  信息
  * ``warn``  警告
  * ``error`` 错误
  * ``fatal`` 致命错误

- ``cache`` 是否缓存日志，当缓存日志开启的时候，日志仅仅会在请求结束后写入存储设备， 否则将会在调用记录日志时立即写入。

- ``handler`` 日志记录方式，目前支持以下几种 :

  * ``file`` 将日志写入文件，扩展参数如下 ::

      'handler' => 'file', # 使用文件记录
      'path' => '/tmp/diabolo.demo.log', # 日志存储地址
      'enableDailyFile' => true, # 是否每天创建一个日志文件

  * ``database`` 将日志写入数据库, 扩展参数如下 ::

      'handler' => 'database', # 使用数据库记录
      'dsn' => 'mysql:host=localhost;dbname=test', # 数据库链接
      'user' => 'user',  # 数据库用户名，如果为空则为null
      'password' => 'pass', # 数据库密码
      'table' => 'runtimelog', # 日志表名
      'attrs' => array( # 日志表列映射
          'time' => '$time', 
          'type' => '$type',
          'content' => '$content',
      ),

    日志表列映射时，如果值以 ``$`` 开头， 则表明使用日志环境变量，
    否则将会直接赋值给该列。 目前日志环境变量支持以下取值 ::

      $time 日志记录时间，使用"时间戳.毫秒"的格式
      $type 日志记录类型， 例如：info, error 等
      $content 日志记录内容。
 

记录日志
========

在服务启动后，日志记录可以直接使用 ``X\Service\XLog\Logger`` 类来进行操作。 
例如 ::

    use X\Service\XLog\Logger;
    
    Logger::debug("This is a debug message");
    Logger::error("This is an error message");
    Logger::fatal("This is a fatal message");
    Logger::info("This is an info message");
    Logger::trace("This is a trace message");
    Logger::warn("This is a warning message");
