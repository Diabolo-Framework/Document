错误处理服务
************

服务配置
========

  配置内容 ::

    'services' => arrary(
        'XError'=>array (
            'enable' => true,
            'class' => 'X\\Service\\XError\\Service',
            'delay' => false,
            'params' => array(
                'types' => E_ALL, # 错误类型
                'handlers' => array( # 错误处理器
                    array(
                        'handler' => 'Email',           # 发生错误时发送邮件 
                        'mail_handler' => 'default',    # 邮件处理名称
                        'subject' => 'Example Subject', # 邮件标题        
                        'recipients' => array( # 收件人列表
                            'NameOfRecipient'=>'address@example.com'
                         ),
                        'template' => 'default', # 邮件模板
                        'isHtml' => false, # 是否启用HTML格式发送
                    ),
                    array(
                        'handler' => 'FunctionCall', # 函数调用
                        'callback' => array(ErrorHandler::class, 'handle'), # 错误回调函数
                    ),
                    array(
                        'handler' => 'Url', # 发生错误时请求URL
                        'url' => 'http://www.example.com?action=handleError', # URL地址 
                        'parameters' => array( # 请求参数
                            'code' => '?',     # 错误代码
                            'message' => '?',  # 错误消息
                            'file' => '?',     # 错误文件
                            'line' => '?',     # 错误行号
                            'user' => 'user',  # 自定义参数
                            'password'=>'password', # 自定义参数
                        ),
                        'gotoUrl' => true, # 是否跳转到该URL
                        'method' => 'post', # 请求方法
                    ),
                    array(
                        'handler' => 'View', # 发生错误时渲染该视图
                        'path' => 'View/Error.php', # 视图路径
                    ),
                ),
            ),
        ),
    );

- ``types`` 错误类型，用于配置该服务处理的错误类型，默认为 ``E_ALL``, 
  取值参考 : http://php.net/manual/zh/errorfunc.constants.php

- ``handlers`` 错误处理器列表， 该服务支持配置多个错误处理器， 当错误发生时， 
  处理器将会被按照所配置的顺序进行执行。目前支持的处理类型包括 : 
  ``Email``, ``FunctionCall``, ``Url``, ``View`` 这四种。

- ``Email`` 邮件处理， 邮件处理需要启用 ``XMail`` 服务来进行邮件的发送。

  * ``mail_handler``  在 ``XMail`` 中配置的 ``handler`` 名称。
  * ``template`` 邮件模板， 默认为 ``default``, 如果不使用默认模板， 
    则配置的路径需要以项目的根目录来配置视图路径， 例如 :: ``View\Error.php`` 。

- ``FunctionCall`` 回调处理， 回调处理的回调函数接受一个 $error 参数用于获取错误信息， 例如 ::

    public static function handle( $error ) {
        var_dump($error);
    }

- ``Url`` URL处理， URL处理是在错误发生后调用该URL进行处理的操作。

  * ``parameters`` 请求URL时传递的参数列表， 参数列表中对应的 ``code`` ， ``message`` , 
    ``file`` , ``line``, 将会被赋值为错误信息中的对应值。

  * ``gotoUrl`` 是不是要跳转到对应的url， 如果该参数为 ``true``, 则处理器将会输出一个界面并
    然后跳转到指定url， 否则将直接请求该url，不做跳转。
 
  * 在 ``cli`` 模式下， 不能够使用 ``gotoUrl`` 功能。

- ``View`` 视图处理， 当错误发生后， 将渲染该视图。

错误处理
========

 该服务不需要从外部调用，配置完成后将会自动处理错误。

