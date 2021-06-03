## 微信支付回调处理



###### 建议使用EasyWechat

```php
/**
     * @Function notify #获取微信回调的数据
     *
     * @return mixed
     * @autthor：Alert
     */
    public function notify()
    {
        // 获取微信回调的数据
        $notifiedData = file_get_contents('php://input'); //php7+使用此方法 否则使用：$xml = $GLOBALS['HTTP_RAW_POST_DATA'];
        //XML格式转换
        $xmlObj = simplexml_load_string($notifiedData, 'SimpleXMLElement', LIBXML_NOCDATA);

        return json_decode(json_encode($xmlObj), true);
    }

    /**
     * @Function callbackToUrlParams #签名校验
     *
     * @param $Parameters
     *
     * @return string
     * @autthor：Alert
     */
    public function callbackToUrlParams($Parameters)
    {
        $buff = '';
        foreach ($Parameters as $k => $v) {
            if ('sign' != $k && '' != $v && !is_array($v)) {
                $buff .= $k.'='.$v.'&';
            }
        }

        return trim($buff, '&');
    }

    /**
     * @Function appgetSign #签名校验
     *
     * @param $data
     *
     * @return string
     * @autthor：Alert
     */
    public function appgetSign($data)
    {
        //签名步骤一：按字典序排序参数
        ksort($data);
        $String = $this->callbackToUrlParams($data);
        //签名步骤二：在string后加入KEY
        $String = $String.'&key='.WxConfig::KEY;
        //签名步骤三：MD5加密
        $String = md5($String);
        //签名步骤四：所有字符转为大写
        return strtoupper($String);
    }

    /**
     * @Function returnXml #返回微信xml
     * @autthor：Alert
     */
    public function returnXml()
    {
        header('Content-type:text/xml;');
        $xml = "<?xml version='1.0' encoding='UTF-8'?>\n";
        $xml .= "<xml>\n";
        $xml .= "<return_code>SUCCESS</return_code>\n";
        $xml .= "<return_msg>OK</return_msg>\n";
        $xml .= "</xml>\n";
        echo $xml;
    }

    /**
     * @Function actionSelfPayNotify #逻辑处理
     * @autthor：Alert
     */
    public function actionSelfPayNotify()
    {
        $notify = $this->notify();
        file_put_contents('./wxpay.log', json_encode($notify));
        if ('SUCCESS' == $notify['return_code'] && 'SUCCESS' == $notify['result_code']) {
            if ($notify['sign'] === $this->appgetSign($notify)) {
                // 总订单号
                //处理订单逻辑
                $trade_no = $notify['out_trade_no'];
            }
        }
    }
```
