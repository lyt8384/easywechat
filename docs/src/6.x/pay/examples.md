# 示例

> 👏🏻 欢迎点击本页下方 "帮助我们改善此页面！" 链接参与贡献更多的使用示例！

<details>
    <summary>JSAPI 下单</summary>

> 官方文档：<https://pay.weixin.qq.com/wiki/doc/apiv3/apis/chapter3_1_1.shtml>

```php
$response = $app->getClient()->postJson("v3/pay/transactions/jsapi", [
   "mchid" => "1518700000", // <---- 请修改为您的商户号
   "out_trade_no" => "native12177525012012070352333'.rand(1,1000).'",
   "appid" => "wx6222e9f48a0xxxxx", // <---- 请修改为服务号的 appid
   "description" => "Image形象店-深圳腾大-QQ公仔",
   "notify_url" => "https://weixin.qq.com/",
   "amount" => [
        "total" => 1,
        "currency" => "CNY"
    ],
    "payer" => [
        "openid" => "o4GgauInH_RCEdvrrNGrnxxxxxx" // <---- 请修改为服务号下单用户的 openid
    ]
]);

\dd($response->toArray(false));
```

</details>

<details>
    <summary>Native 下单</summary>

```php
$response = $app->getClient()->postJson('v3/pay/transactions/native', [
    'mchid' => (string)$app->getMerchant()->getMerchantId(),
    'out_trade_no' => 'native20210720xxx',
    'appid' => 'wxe2fb06xxxxxxxxxx6',
    'description' => 'Image形象店-深圳腾大-QQ公仔',
    'notify_url' => 'https://weixin.qq.com/',
    'amount' => [
        'total' => 1,
        'currency' => 'CNY',
    ]
]);

print_r($response->toArray(false));
```

</details>

<details>
    <summary>查询订单（商户订单号）</summary>

```php

$outTradeNo = 'native20210720xxx';
$response = $app->getClient()->get("v3/pay/transactions/out-trade-no/{$outTradeNo}", [
    'query'=>[
        'mchid' =>  $app->getMerchant()->getMerchantId()
    ]
]);

print_r($response->toArray());
```

</details>

<details>
    <summary>查询订单（微信订单号）</summary>

```php
$transactionId = '217752501201407033233368018';
$response = $app->getClient()->get("pay/transactions/id/{$transactionId}", [
    'query'=>[
        'mchid' =>  $app->getMerchant()->getMerchantId()
    ]
]);

print_r($response->toArray());
```

</details>

<details>
    <summary>Laravel 中处理微信支付回调</summary>

> 记得需要将此类路由关闭 csrf 验证。

```php
// 假设你设置的通知地址notify_url为: https://easywechat.com/payment_notify

// 注意：通知地址notify_url必须为https协议

Route::post('payment_notify', function () {
    // $app 为你实例化的支付对象，此处省略实例化步骤
    $server = $app->getServer();

    // 处理支付结果事件
    $server->handlePaid(function ($message) {
        // $message 为微信推送的通知结果，详看微信官方文档

        // 微信支付订单号 $message['transaction_id']
        // 商户订单号 $message['out_trade_no']
        // 商户号 $message['mchid']
        // 具体看微信官方文档...
        // 进行业务处理，如存数据库等...
    });

    // 处理退款结果事件
    $server->handleRefunded(function ($message) {
        // 同上，$message 详看微信官方文档
        // 进行业务处理，如存数据库等...
    });

    return $server->serve();
});
```

</details>
  
<details>
   <summary>付款（V2）</summary>

```php
$response = $api->post('/mmpaymkttransfers/promotion/transfers', [
    'body' => [
        'mch_appid' => $app->getConfig()['app_id'],     //注意在配置文件中加上app_id
        'mchid' => $app->getConfig()['mch_id'],         //商户号
        'partner_trade_no' => '202203081646729819743',  // 商户订单号，需保持唯一性(只能是字母或者数字，不能包含有符号)
        'openid' => 'ogn1H45HCRxVRiEMLbLLuABbxxxx',     //用户openid
        'check_name' => 'FORCE_CHECK',                  // NO_CHECK：不校验真实姓名, FORCE_CHECK：强校验真实姓名
        're_user_name'=> '用户真实姓名',                  // 如果 check_name 设置为 FORCE_CHECK 则必填用户真实姓名
        'amount' => '100',                              //金额
        'desc' => '理赔',                                // 企业付款操作说明信息。必填
    ],
    'local_cert' => $app->getConfig()['certificate'], //v2证书绝对路径
    'local_pk' => $app->getConfig()['private_key'],   //v2证书密钥绝对路径
]);

print_r($response->toArray());
```

</details>

<details>
   <summary>JSAPI下单（服务商）</summary>

> 官方文档：<[https://pay.weixin.qq.com/wiki/doc/apiv3/apis/chapter3_1_1.shtml](https://pay.weixin.qq.com/docs/partner/apis/partner-jsapi-payment/partner-jsons/partner-jsapi-prepay.html)>

```php
 $response = $app->getClient()->postJson("v3/pay/partner/transactions/jsapi", [
            "sp_appid" => $appId, // 服务商应用ID
            "sp_mchid" => '********', // 服务商户号
            'sub_mchid' => '*********', // 子商户号/二级商户号
            "sub_appid" => '********', // 子商户/二级商户应用ID(选填)
            "description" => $this->payDesc($from), // 商品描述
            "out_trade_no" => $order['pay_sn'], // 商户订单号
            "notify_url" => $this->config['notify_url'], // 通知地址
            "amount" => [
                "total" => intval($order['order_amount'] * 100), // 总金额
            ], // 订单金额信息
            "payer" => [
                "sp_openid" => $this->auth['openid'], // 用户服务标识，户在服务商AppID下的唯一标识
                "sub_openid" => $this->auth['openid'] // 用户子标识，用户在子商户AppID下的唯一标识。若传sub_openid，那sub_appid必填。下单前需获取到用户的OpenID
            ], // 支付者,(sp_openid 和 sub_openid 二选一)
            'attach' => $from
        ]);

print_r($response->toArray());
```

</details>

<details>
    <summary>敏感信息加密</summary>

> 官方文档：<https://pay.weixin.qq.com/doc/v3/merchant/4013053257>
> 使用默认公钥 ID

```php
$utils = $app->getUtils();
$response = $app->getClient()->withSerialHeader()->postJson("v3/applyment4sub/applyment/", [
   "business_code" => "12345678",
   'contact_info'  => [
        'contact_name'      => $utils->createRsaEncrypt('张三'),
        //...
    ],
    //...
]);

print_r($response->toArray());
```

或指定公钥 ID

```php
$utils = $app->getUtils();
$response = $app->getClient()->withSerialHeader("PUB_KEY_ID_123456")->postJson("v3/applyment4sub/applyment/", [
   "business_code" => "12345678",
   'contact_info'  => [
        'contact_name'      => $utils->createRsaEncrypt("张三","PUB_KEY_ID_123456"),
        //...
    ],
    //...
]);

print_r($response->toArray());
```

</details>
  
<!--
<details>
    <summary>标题</summary>
内容
</details>
-->
