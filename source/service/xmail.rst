邮件服务
********

注册服务
========

  配置内容 ::

    'services' => array(
        'XMail'=>array (
            'enable' => true,
            'class' => 'X\\Service\\XMail\\Service',
            'delay' => true,
            'params' => array(
                # 配置邮件服务器列表
                'handlers' => array(
                    'default' => array(
                        'handler' => 'smtp',
                        'host' => 'smtp.163.com',
                        'port' => '25',
                        'from' => 'user@example.com',
                        'from_name' => 'Example',
                        'auth_required' => true,
                        'username' => 'user@example.com',
                        'password' => 'password',
                    ),
                    'another' => array(
                        'handler' => 'smtp',
                        'host' => 'smtp.163.com',
                        'port' => '25',
                        'from' => 'another@example.com',
                        'from_name' => 'Example',
                        'auth_required' => true,
                        'username' => 'another@example.com',
                        'password' => 'password',
                    ),
                ),
            ),
        ),
    ),

- 邮件服务支持配置多个邮件服务器。
- 当发送邮件时，如果没有指定服务器名称， 则会使用名称为 ``default`` 的邮件服务器。


发送邮件
========

配置完成后，服务不再需要初始化， 可以直接使用, 例如 ::

    use X\Service\XMail\Service;

    $mailService = Service::getService();
        
    # 创建一个 Mail 实例
    $mail = $mailService->create('Demo Mail');
    # 设置邮件发件人, ‘res-name’ 是收件人的名称， 可以不填。
    $mail->addAddress('res@example.com', 'res-name');
    # 设置邮件内容
    $mail->setContent("This is a demo email");
    # 发送邮件，成功将返回true.
    if ( $mail->send() ) {
       echo "成功";
    } else {
       echo $mail->ErrorInfo;
    }
        
    
    $mail = $mailService->create('Demo Mail');
    # 使用名称为 another 的邮件服务器发送邮件。
    $mail->setHandler('another');
    $mail->addAddress('568109749@qq.com', 'GGGG');
    $mail->setContent("This is a demo email");
    $mail->send();


