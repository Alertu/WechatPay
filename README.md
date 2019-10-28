微信支付处理回调  
  
  
     //    获取微信回调的数据   
     public function notify() 
     {
        
        // 获取微信回调的数据
        $notifiedData = file_get_contents('php://input');//php7+使用此方法 否则使用：$xml = $GLOBALS['HTTP_RAW_POST_DATA'];
        //XML格式转换
        $xmlObj = simplexml_load_string($notifiedData, 'SimpleXMLElement', LIBXML_NOCDATA);
        $xmlObjs = json_decode(json_encode($xmlObj),true);
        return $xmlObjs;
    }
    //签名校验
    public function callbackToUrlParams($Parameters){
        $buff = "";
        foreach ($Parameters as $k => $v){
            if($k != "sign" && $v != "" && !is_array($v)){
                $buff .= $k . "=" . $v . "&";
            }
        }
        $buff = trim($buff, "&");
        return $buff;
    }
    //签名校验
    public  function appgetSign($data){
        //签名步骤一：按字典序排序参数
        ksort($data);
        $String = $this->callbackToUrlParams($data);
        //签名步骤二：在string后加入KEY
        $String = $String."&key=".WxConfig::KEY;
        //签名步骤三：MD5加密
        $String = md5($String);
        //签名步骤四：所有字符转为大写
        $result_ = strtoupper($String);
        return $result_;
    }
    //返回微信xml
       public  function returnXml(){
        header("Content-type:text/xml;");
        $xml = "<?xml version='1.0' encoding='UTF-8'?>\n";
        $xml .= "<xml>\n";
        $xml .= "<return_code>SUCCESS</return_code>\n";
        $xml .= "<return_msg>OK</return_msg>\n";
        $xml .= "</xml>\n";
        echo  $xml;
    }
    public function actionSelfPayNotify()
    {
        $notify=$this->notify();
        file_put_contents('./wxpay.log',json_encode($notify));
        if ($notify['return_code'] == "SUCCESS" && $notify['result_code'] == "SUCCESS")
        {
            if ($notify['sign'] === $this->appgetSign($notify)) {
                // 总订单号
                //处理订单逻辑
                $trade_no = $notify['out_trade_no'];
                }
        }
    }
