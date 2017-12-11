OAuth 2.0 服务
**************

注册服务
========

  配置内容 ::

    'services' => array(
        'OAuth2' => array(
            'class' => '\\X\\Service\\OAuth2\\Service',
            'enable' => true,
            'delay' => true,
            'params' => array(
                # 存储方式
                'storage_handler' => 'Pdo',
                # 存储配置
                'storage_params'=>array(
                    'dsn'=>'mysql:dbname=demo_db_name;host=db_host', 
                    'username'=>'db_user', 
                    'password'=>'db_password'
                ),
                # 授权方式
                'grant_types' => array(
                    'AuthorizationCode',
                    'ClientCredentials',
                    'RefreshToken',
                    'UserCredentials'
                ),
                # oauth 2.0 配置
                'option' => array(
                    'use_jwt_access_tokens'        => false,
                    'store_encrypted_token_string' => true,
                    'use_openid_connect'       => false,
                    'id_lifetime'              => 3600,
                    'access_lifetime'          => 3600,
                    'www_realm'                => 'Service',
                    'token_param_name'         => 'access_token',
                    'token_bearer_header_name' => 'Bearer',
                    'enforce_state'            => true,
                    'require_exact_redirect_uri' => true,
                    'allow_implicit'           => false,
                    'allow_credentials_in_request_body' => true,
                    'allow_public_clients'     => true,
                    'always_issue_new_refresh_token' => false,
                    'unset_refresh_token_after_use' => true,
                ),
            ),
        ),
    ), 


初始化
======

- 初始化数据库，SQL ::

    CREATE TABLE oauth_clients (
      client_id             VARCHAR(80)   NOT NULL,
      client_secret         VARCHAR(80),
      redirect_uri          VARCHAR(2000),
      grant_types           VARCHAR(80),
      scope                 VARCHAR(4000),
      user_id               VARCHAR(80),
      PRIMARY KEY (client_id)
    );

    CREATE TABLE oauth_access_tokens (
      access_token         VARCHAR(40)    NOT NULL,
      client_id            VARCHAR(80)    NOT NULL,
      user_id              VARCHAR(80),
      expires              TIMESTAMP      NOT NULL,
      scope                VARCHAR(4000),
      PRIMARY KEY (access_token)
    );

    CREATE TABLE oauth_authorization_codes (
      authorization_code  VARCHAR(40)     NOT NULL,
      client_id           VARCHAR(80)     NOT NULL,
      user_id             VARCHAR(80),
      redirect_uri        VARCHAR(2000),
      expires             TIMESTAMP       NOT NULL,
      scope               VARCHAR(4000),
      id_token            VARCHAR(1000),
      PRIMARY KEY (authorization_code)
    );

    CREATE TABLE oauth_refresh_tokens (
      refresh_token       VARCHAR(40)     NOT NULL,
      client_id           VARCHAR(80)     NOT NULL,
      user_id             VARCHAR(80),
      expires             TIMESTAMP       NOT NULL,
      scope               VARCHAR(4000),
      PRIMARY KEY (refresh_token)
    );

    CREATE TABLE oauth_users (
      username            VARCHAR(80),
      password            VARCHAR(80),
      first_name          VARCHAR(80),
      last_name           VARCHAR(80),
      email               VARCHAR(80),
      email_verified      BOOLEAN,
      scope               VARCHAR(4000),
      PRIMARY KEY (username)
    );

    CREATE TABLE oauth_scopes (
      scope               VARCHAR(80)     NOT NULL,
      is_default          BOOLEAN,
      PRIMARY KEY (scope)
    );

    CREATE TABLE oauth_jwt (
      client_id           VARCHAR(80)     NOT NULL,
      subject             VARCHAR(80),
      public_key          VARCHAR(2000)   NOT NULL
    );

- 插入演示数据 ::

    INSERT INTO oauth_clients 
      (client_id, client_secret, redirect_uri) 
    VALUES 
      ("testclient", "testpass", "http://fake/");

获取 Access Token
=================

  在请求资源之前，需要获取一个 access token， 然后才能够调用资源接口 ::
  
    $service = \X\Service\OAuth2\Service::getService();
    $service->generateAccessToken()->send();

  假设调用该接口的url为 ``http://example.com/module=oauth2&action=token``,
  则结果将会输出 ::

    {
        "access_token":"03807cb390319329bdf6c777d4dfae9c0d3b3c35",
        "expires_in":3600,
        "token_type":"bearer",
        "scope":null
    }


处理资源请求
============

  在请求资源接口时， 需要将上一步请求获取的access token放入请求的参数里面， 
  并且在处理请求的时候需要判断请求是否已经有效 ::

    $service = \X\Service\OAuth2\Service::getService();
    if ( !$service->verifyResourceRequest() ) {
        echo json_encode(array(
            'success' => false,
            'message' => 'authoriation required',
        ));
    }
    
    echo json_encode(array(
        'success' => true, 
        'message' => 'You accessed my APIs!', 
        'data'=>array('ver'=>'1.0.0')
    ));

  假设调用该接口的URL为 ``http://example.com/module=api&action=version`` ，
  并且将 ``access_token`` 作为POST参数传入，
  调用成功后输出 ::

    {
        "success" : true,
        "message" : "You accessed my APIs!",
        "data"    : {
            "ver" : "1.0.0"
        }
    } 
  
