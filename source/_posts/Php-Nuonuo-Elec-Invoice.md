---
title: PHP接入诺诺电子发票
date: 2021-03-16 09:36:31
tags:
- PHP
- API
category:
- PHP
---
# 前期准备11111

首先开票公司必须使用的是航天信息的开票系统，有航天信息提供的开票软件客户端、金税盘等。然后要在诺诺电子发票服务平台注册企业账号，注意是企业账号，不是个人账号，地址：[https://fp.jss.com.cn](https://fp.jss.com.cn/) ，如果要开电票的话（诺诺发票服务平台也可以开普票纸票和专票），可能还要去税务局找航信公司人员办理电子签章等，航信一般在当地税务局有窗口办理业务。


注册和签章做完后，可以登录诺诺电子发票服务平台工作台。
<!-- more -->
这里还是简单介绍一下诺诺的各平台吧。
1.电子发票服务平台，诺诺提供的在线开票和发票管理平台，这里可以直接开票，前提是你已经办好了上述的一些手续，电子发票服务平台的工作台，可以直接开票，注意，这里不光可以开电子发票，还可以开纸质普票，专票，可以选择开票类型。

2.诺诺开放平台，如果你不使用诺诺提供的电子发票服务平台，或者要将电子发票引入到你自己的业务系统，那就要使用诺诺开放平台，地址:[https://open.jss.com.cn](https://open.jss.com.cn/) ，账号是通用的，可直接登录诺诺开放平台，在应用管理中，创建应用，一般应用类型是自用型应用，然后会分别给一个正式环境和沙箱环境的appkey和APP Secret，开放平台提供了开发文档和sdk，使用都很简单。

这里要注意，先去看文档找接口，那最重要的就是开票接口，平台的文档里没有开票接口文档，只有在注册了账号，创建了应用后，查看应用中提供了开票接口的文档……

3.诺诺开票软件，诺诺发票，诺诺电子发票等客户端，这些都是财务使用的，也许有的
财务只用其中一个或2个，所有这些客户端都需要插上金税盘。


# 开票流程
## 普通的开票：
公司财务直接在开票软件中开了，开票时填写开票信息等，如果开通了电子发票，开票软件中也可以开电子发票，开完后不用打印，直接将电子发票发送到开票信息中提供的手机和邮箱中了，也会提供电子发票的下载地址。

使用电子发票服务平台开票：
1.在公司财务电脑（有金税盘的）上打开诺诺发票，开启开票服务，然后数据就会在发票服务平台和公司财务的开票软件之间同步，下图是诺诺发票客户端开启发票服务的状态。
2.登录电子发票服务平台工作台，点击发票填开，填写发票信息，发票类型。提交开票。提交的时候会请求公司的财务电脑的诺诺发票同步数据，如果请求不成功，会提示相应的错误，如果请求成功，开票状态是待开。
3.如果是纸票和专票，在公司财务电脑上打开开票软件，发票管理或者开票界面导入中，有从电子发票服务平台提交过来的开票信息，导入打印开票。开票完成后，数据会同步到电子发票服务平台中，刷新的话发票状态已经变成开票完成了。如果是电子发票，同样会请求公司财务电脑诺诺发票，不过不需要手动打印开票，而是请求电子签章，完成后同样将数据同步回去。
所以公司的诺诺发票软件要一直保持开启开票服务的状态，不然无法从电子发票服务平台（也包括开放平台api）提交发票，但是有时候财务人员会拿金税盘到税务去办相关的业务，金税盘不插的话，诺诺发票和开票软件等客户端肯定无法正常使用，而且财务的电脑也不是服务器，也不可能一直保持开机状态，这种情况可能需要使用分机，办理了分机的话，会有一个分机税盘，然后要配置分机，这个可以下载一个诺言客户端咨询客服，客服会通过远程给协助配置。

## 开放平台开票：
搞清楚电子发票服务平台后，那使用开放平台api就简单了，自己的业务系统中使用开发平台接口所完成的功能其实就是电子发票服务平台提供的功能的简化版，只是把一些功能通过api加到自己的业务系统中了，所以使用开放平台api开票，首先要调试电子发票服务平台能不能开票成功，如果电子发票服务平台开票不成功，api接口也不会调试成功！

所以，不管是电子发票服务平台，还是开放平台，最终开票请求都通过诺诺提交到了公司财务的开票电脑上了，开票电脑完成开票后返回开票的结果。

### PHP实现
这个就很简单了，主要使用三个接口：获取access_token，开票和查询，下载php的sdk。直接上代码：
```php
require_once "../nuonuo/lib/Api.php";
```
#### 获取access_token：
```php
$appKey    = C('invoice.appKey');
$appSecret = C('invoice.appSecret');
try {
        $token = Api::getMerchantToken($appKey, $appSecret);
} catch (Exception $e) {
        return false;
}
if (!isJson($token)) {
        return false;
}
$token = json_decode($token);
if (!isset($token->access_token) || !isset($token->expires_in)) {
        return false;
}
return $token->access_token;
```
注意：access_token的调用限制是30天50次，需要本地缓存起来，在有效期内重复使用。

#### 开票请求：
```php
public function request() {
        $appKey    = C('invoice.appKey');
        $appSecret = C('invoice.appSecret');
        $token     = self::getMerchantToken();
        $taxnum    = C('invoice.salerTaxNum');
        $url       = C('invoice.url');
        $method    = "nuonuo.electronInvoice.requestBilling";
        $senid     = $this->senid;
        $body      = json_encode(
            array(
                'order' => array(
                    'buyerName' => stripslashes($this->buyerName),
                    'buyerTaxNum' => $this->buyerTaxNum,
                    'buyerTel' => $this->buyerTel,
                    'buyerAddress' => stripslashes($this->buyerAddress),
                    'buyerAccount' => stripslashes($this->buyerAccount),
                    'orderNo' => $this->orderNo,
                    'invoiceDate' => $this->invoiceDate,
                    'clerk' => $this->clerk,
                    'payee' => C('invoice.payee'),
                    'checker' => C('invoice.checker'),
                    'salerTaxNum' => $this->salerTaxNum,
                    'salerTel' => $this->salerTel,
                    'salerAddress' => $this->salerAddress,
                    'salerAccount' => $this->salerAccount,
                    'invoiceType' => $this->invoiceType,
                    'pushMode' => $this->pushMode,
                    'buyerPhone' => $this->buyerPhone,
                    'email' => $this->email,
                    'invoiceLine' => $this->invoiceLine,
                    'invoiceDetail' => array(
                        'goodsName' => $this->goodsName,
                        'goodsCode' => $this->goodsCode,
                        'num' => $this->num,
                        'price' => $this->price,
                        'taxIncludedAmount' => $this->taxIncludedAmount,
                        'withTaxFlag' => $this->withTaxFlag,
                        'taxRate' => $this->taxRate,
                    )
                )
            )
        );
        try {
            $res = Api::sendPostSyncRequest($url, $senid, $appKey, $appSecret, $token, $taxnum, $method, $body);
        } catch (Exception $e) {
            $this->message = $e->getMessage();
            return false;
        }
        if (!isJson($res)) {
            $this->message = '发票请求失败';
            return false;
        }
        $res = json_decode($res);
        if ($res->code !== 'E0000') {
            $this->message = $res->describe;
            return false;
        }
        if (!isset($res->result->invoiceSerialNum)) {
            $this->message = '未收到发票流水号';
            return false;
        }
        $this->invoiceSerialNum = $res->result->invoiceSerialNum;
        $this->update_field(array('invoiceSerialNum'));
        return true;
    }
```
注意：请求成功后，保存返回的发票流水号，用来查询开票结果。
#### 查询结果：
```php
public static function queryInvoice(array $serialNos) {
        $appKey    = C('invoice.appKey');
        $appSecret = C('invoice.appSecret');
        $token     = self::getMerchantToken();
        $taxnum    = C('invoice.salerTaxNum');
        $url       = C('invoice.url');
        $method    = "nuonuo.ElectronInvoice.queryInvoiceResult";
        $senid     = uniqid();

        $body = json_encode(array(
            'serialNos' => $serialNos,
        ));
        try {
            $res = Api::sendPostSyncRequest($url, $senid, $appKey, $appSecret, $token, $taxnum, $method, $body);
        } catch (Exception $e) {
            return false;
        }
        if (!isJson($res)) {
            return false;
        }
        $res = json_decode($res);
        if ($res->code !== 'E0000') {
            return false;
        }
        if (!isset($res->result)) {
            return false;
        }
        return $res->result;
    }
```
注意：发票结果查询需要作为定时任务一直运行，然后将查询的结果更新到本地，可批量查询。
最新的接口中，请求发票的时候可以提交一个callbackUrl参数，这个url只有开票成功的时候才会回调，但官方网站没有提供回调请求的文档，个人认为这个参数还是不要使用了，因为光使用回调不使用定时查询的话是不够的，其他状态的发票也需要更新，如开票失败，签章失败，作废等，既然避免不了使用定时任务来更新发票状态，那就没必要做回调了。
