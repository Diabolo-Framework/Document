事件
****

事件注册
========

目前事件注册只能通过代码手动注册，不支持配置的方式注册, 例如为事件 ``event001`` 注册处理器 ::

    $eventManager = \X\Core\X::system()->getEventManager();
    
    # 注册事件处理器
    $eventManager->registerHandler('event001', function( $param1 ) {
        return "THIS IS CALLBACK RESPONSE";
    });

    # 注册事件结果
    $eventManager->registerHandler('event001', "THIS IS A TEST RESULT");

当事件触发的时候，如果处理器是一个可调用的处理器，那么该处理器将会被执行，其返回结果将会返回给事件触发者，如果处理器不可以执行，那么，将直接返回该数据给触发者。

事件触发
========

事件需要手动触发， 例如触发上面的 ``event001`` ::

   $eventManager = \X\Core\X::system()->getEventManager();
   $result = $eventManager->trigger('event001', 'par001');

则 ``$result`` 为 ::

    array(
        "THIS IS CALLBACK RESPONSE",
        "THIS IS A TEST RESULT",
    );

事件触发时能够传递任意多个参数，参数将会传递给事件处理器，以上面这个例子来说，
事件触发的时候传递一个参数值为 ``"par001"``, 那么， 第一个事件处理器中的 ``$param1`` 的值就是 ``"par001"`` 。

事件响应
========

事件触发完成后，返回的数据是包含每个处理器的返回结果数组。如果处理器没有返回，则对应未知的返回值为 ``null`` 。

