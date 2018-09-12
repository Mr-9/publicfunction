
<!-- TOC -->

- [说明文档](#说明文档)
- [function.php](#functionphp)
    - [ma() “跳转地址”](#ma-跳转地址)
    - [returnAPPJSON() “返回json数组”](#returnappjson-返回json数组)
    - [dbOperate() “跨域操作数据库有配置文件”](#dboperate-跨域操作数据库有配置文件)
    - [_dbOperate() “跨域操作数据库，输入数据库信息”](#_dboperate-跨域操作数据库输入数据库信息)
    - [Jpush() “极光推送”](#jpush-极光推送)
    - [showExcel() “导出excel表格”](#showexcel-导出excel表格)
    - [post_curl() 模拟POST请求方法](#post_curl-模拟post请求方法)
    - [encodePass() “密码加盐”](#encodepass-密码加盐)
    - [checkPass() “密码校验”](#checkpass-密码校验)
    - [returnHtmldecode() “html实体化decode方法”](#returnhtmldecode-html实体化decode方法)
    - [time_to_date() “时间戳转时间”](#time_to_date-时间戳转时间)
    - [pageFormatBiz() “分页格式化数据”](#pageformatbiz-分页格式化数据)
    - [sendSms() “发送短信”](#sendsms-发送短信)
    - [checkVerifyCode() “验证码校验”](#checkverifycode-验证码校验)
    - [ch2arr() “中文拆数组”](#ch2arr-中文拆数组)
    - [getFirstCharter() “php获取中文字符拼音首字母”](#getfirstcharter-php获取中文字符拼音首字母)
    - [getMapName() “查询当前id所属父级类别的名称”](#getmapname-查询当前id所属父级类别的名称)
    - [getMapId() “得到当前id的父类id数组集合”](#getmapid-得到当前id的父类id数组集合)
    - [getTree() “递归获取列表”](#gettree-递归获取列表)
- [PublicController.class.php](#publiccontrollerclassphp)
    - [pageS() “封装的tp的分页”](#pages-封装的tp的分页)
    - [appPageS() “封装的tp分页，根据传参返回列表”](#apppages-封装的tp分页根据传参返回列表)
    - [EditOk() “增加编辑函数”](#editok-增加编辑函数)
    - [Del() “删除”](#del-删除)
    - [DelAll “批量删除”](#delall-批量删除)
    - [detail() “详情”](#detail-详情)
    - [addAll()批量增加](#addall批量增加)

<!-- /TOC -->
# 说明文档

# function.php  

## ma() “跳转地址”
```php
/**
* @param $text 提示文字
* @param $url  跳转路径
*/
    function ma($text, $url){
        echo "<script>
                alert('".$text."');
                location.href='".$url."';
            </script>
                ";
        exit;
    }
```    
## returnAPPJSON() “返回json数组”
```php
/**
    * 返回json数据
    * @param $ret
    * @param string $msg
    * @param array $data
    */
function returnAPPJSON($ret, $msg='', $data=array()){
        $res['ret'] = $ret.'';
        $res['msg'] = $msg;
        $res['data'] = (($data==null)?array():$data);   // $data存在则返回数组不存在返回空数组
        //$data?$res['data'] = $data:'';                // $data存在则返回不存在不返回
        exit(json_encode($res));
}
```

## dbOperate() “跨域操作数据库有配置文件”
```php
    /**
     * 数据库操作   dbOperate("manufacturer",'',1,3,$where);
     * @param unknown $table 表名
     * @param unknown $tablePrefix  表前缀
     * @param string $db_link  数据库链接数
     * @param unknown $operate_type 0:查询 1：增加，2：修改 3：删除
     * @param unknown $where  条件 适用于查询，修改和删除
     * @param string $data  增加或者修改的数据
     * @return 返回查询列表或操作结果
     */
    function dbOperate($table,$tablePrefix='',$db_link=0,$operate_type = 0,$where='',$data="") {
        try{

        // 得到数据库连接
        if (!empty($db_link)) {
            $db_link = 'DB_CONFIG'.$db_link;
            $model = M($table, $tablePrefix,$db_link);
        } else {
            $model = M($table,$tablePrefix);

        }

        switch ($operate_type) {
            case 0:
                // 查询
                $list = $model->where($where)->select();
                break;
            case 1 :
                // 增加
                if (empty($data)) {
                    return 0;
                }
                $result = $model->data($data)->add();


                break;
            case 2 :
                if (empty($where) || empty($data)) {
                    return 0;
                }
                // 修改
                $result = $model->where($where)->save($data);
                break;
            case 3:
                if (empty($where)) {
                    return 0;
                }
                // 删除
                $result = $model->where($where)->delete();
                break;

        }
        // 返回数据
        if (!empty($operate_type)) {
            if ($result !==false) {
                return $result;
            } else {
                return -1;
            }
        } else {
            return $list;
        }
        }catch(Exception $e){
            return false;
        }

    }
```

## _dbOperate() “跨域操作数据库，输入数据库信息”
```php
    /**
     * 数据库操作
     * @param unknown $table 表名
     * @param unknown $tablePrefix  表前缀
     * @param string $db_link  数据库链接数
     * @param unknown $operate_type 0:查询 1：增加，2：修改 3：删除
     * @param unknown $where  条件 适用于查询，修改和删除
     * @param string $data  增加或者修改的数据
     * @return 返回查询列表或操作结果
     */
    function _dbOperate($table,$db_link=0,$operate_type = 0,$where='',$data="") {
        try{
        $tablePrefix=isset($db_link['DB_PREFIX'])?$db_link['DB_PREFIX']:'';
        // 得到数据库连接
        if (!empty($db_link)) {
    //        $db_link = 'mysql://root:@127.0.0.1/test#utf8'
            $db_link = 'mysql://'.$db_link['DB_USER'].':'.$db_link['DB_PWD'].'@'.$db_link['DB_HOST'].'/'.$db_link['DB_NAME'].'#utf8';
            $model = M($table, $tablePrefix,$db_link);
        } else {
            $model = M($table);
        }
        switch ($operate_type) {
            case 0:
                // 查询
                $list = $model->where($where)->select();
                break;
            case 1 :
                // 增加
                if (empty($data)) {
                    return 0;
                }
                $result = $model->data($data)->add();


                break;
            case 2 :
                if (empty($where) || empty($data)) {
                    return 0;
                }
                // 修改
                $result = $model->where($where)->save($data);
                break;
            case 3:
                if (empty($where)) {
                    return 0;
                }
                // 删除
                $result = $model->where($where)->delete();
                break;

        }
        // 返回数据
        if (!empty($operate_type)) {
            if ($result !==false) {
                return $result;
            } else {
                return -1;
            }
        } else {
            return $list;
        }
        }catch(Exception $e){
            return false;
        }

    }
```

## Jpush() “极光推送”

```php
    /**
     *	极光推送
     * @param unknown $alias   极光注册ID
     * @param unknown $title		标题
     * @param unknown $content		内容
     * @param unknown $ky	APP段判断方法	array('function'=>'unsetorder','order_number'=>$orderdata['order_number'])
     * @param number $type  	来源类型1 客服  2车辆  3  仓库
     * @param string $sound     声音
     * @return boolean
     * $reg_id = M('member')->where(array('member_id'=>$r))->getField('registration_id');
     * foreach (explode(',',$reg_id) as $kk=>$vv){
     * Jpush($vv, '新订单', '您有新订单',array('type'=>'1'),1);
     *}
     */
    function Jpush($app_key,$master_secret,$alias,$title,$content,$ky=array(),$sound='iOS sound'){
        //极光推送
        try {
            Vendor('Jpush.JPush');
            $client = new JPush($app_key, $master_secret);

            $result = $client->push()
            // 		->setPlatform('all')
            // 		->addAllAudience()
            // 		->setNotificationAlert('Hello, JPush')
            // 		->send();
            ->setPlatform(array('ios', 'android'))
            ->addRegistrationId($alias)
            ->addIosNotification($title,'default', JPush::DISABLE_BADGE,true,'category',$ky)
            ->setMessage($content,$title,'string',$ky)
            ->setOptions(null,86400, null,false)
            ->send();//dump($result);exit;
            return true;
        } catch (Exception $e) {
            print $e;
        }
        // return 'Result=' . json_encode($result).$br;

    }
```
## showExcel() “导出excel表格”
```php
    /**
    * 导出Excel表格（单工作表）
    * @param array $file    标识列         例：$file = array('news_title','news_content'……)
    * @param array $title   列名           例：$title = array('标题','内容'……)
    * @param array $head    位置  'A'开始    长度取决于$val   例：$head = array('A','B'……)
    * @param array(array) $val  值  二维数组
    * @param string $name
    * $name = '出库统计';
    * $file = array('chkout_no','user_name','time','num','value');
    * $title = array('出库单号','用户','日期','商品种类','总价值');
    * $head = array('A','B','C','D','E');
    * self::showExcel($file, $title, $head, $list, $name);
    */
    function showExcel($file, $title, $head, $val, $name){
        vendor('PHPExcel.Classes.PHPExcel');
        $objPHPExcel = new \PHPExcel();
        for ($j = 0;$j<count($title);$j++){
            $objPHPExcel->getActiveSheet()->setCellValue($head[$j] . 1, ' '.$title[$j]);
        }
        for ($i = 2; $i <= count($val)+1; $i++) {
            for ($k = 0; $k<count($head); $k++){
                $input = $val[$i-2][$file[$k]];
                if(!in_array(gettype($input), array('integer', 'double'))){
                    $input = ' ' . $input;
                }
                $objPHPExcel->getActiveSheet()->setCellValue($head[$k] . $i, $input);
            }
        }
        $objWriter = new \PHPExcel_Writer_Excel5($objPHPExcel);
        header("Pragma: public");
        header("Expires: 0");
        header("Cache-Control:must-revalidate, post-check=0, pre-check=0");
        header("Content-Type:application/force-download");
        ob_end_clean();//去除中文乱码
        header("Content-Type:application/vnd.ms-execl");
        header("Content-Type:application/octet-stream");
        header("Content-Type:application/download");
        header('Content-Disposition:attachment;filename="'.$name.'-'.date('Ymd').'.xls"');
        header("Content-Transfer-Encoding:binary");
        $objWriter->save('php://output');
    }
```
## post_curl() 模拟POST请求方法
```php
    /**
     * curlPOST请求方法
     * @param unknown $xmlend  参数集合
     * @param unknown $headers  sessionID
     * @param string $url   请求路径
     * @return mixed
     */
    function post_curl($xmlend,$headers,$url){
        //初始化
        $ch = curl_init();
        //设置属性
            //CURLOPT_URL 指提交到哪里相当于表单的action
        curl_setopt($ch, CURLOPT_URL, $url);
            //https 访问的时候相应配置
        curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE);
        //curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
            //设置头文件的信息作为数据流输出
        curl_setopt($ch, CURLOPT_HEADER, 0);
            //设置获取的信息以文件流的形式返回，而不是直接输出。
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
            //设置请求数据格式
        $head = array(
                "Content-type: application/json;charset='utf-8'",　　
            );
        curl_setopt($ch, CURLOPT_HTTPHEADER, $head);

            // post数据为1表示会发送一个常规的请求
        curl_setopt($ch, CURLOPT_POST, 1);
            // 设置post数据
        curl_setopt($ch, CURLOPT_POSTFIELDS, $xmlend);
        //执行并获得结果
        $output = curl_exec($ch);
        //释放curl句柄
        curl_close($ch);
        return $output;
    //     dump($output);dump(json_decode($output,true));
    }
```
## encodePass() “密码加盐”
```php
    /**
     * 密码加盐Hash
     * @param $passSHA 要加密的密码
     * @return mixed
     */
    function encodePass($passSHA){
        $passSHA = strtolower($passSHA);
        $data['salt'] = strtolower(substr(uniqid(),7,6));
        $data['hash'] = strtoupper(MD5($passSHA.$data['salt']));
        return $data;
    }
```

## checkPass() “密码校验”

```php
    /**
     * 密码校验
     * @param $passSHA 待验证的密码
     * @param $salt  库里的盐
     * @param $passHASH 库里存储的密码
     * @return bool
     */
    function checkPass($passSHA, $salt, $passHASH){
        $passSHA = strtolower($passSHA);
        $salt = strtolower($salt);
        $res = strtoupper(MD5($passSHA.$salt));
        if($passHASH===$res){
            return true;
        }else{
            return false;
        }
    }
```

## returnHtmldecode() “html实体化decode方法”
```php
    function returnHtmldecode($data){
        $back=html_entity_decode($data);
        return $back;
    }
```

## time_to_date() “时间戳转时间”
```php
    function time_to_date($time,$date='Y-m-d H:i:s'){
            return date($date,$time);
    }
```

## pageFormatBiz() “分页格式化数据”
```php
    /**
     * 分页格式化
     * $Page = new \Think\Page($count,15);
     */
    function pageFormatBiz($page){
        $page->rollPage = 6;
        $page->setConfig('prev', '上一页');
        $page->setConfig('next', '下一页');
        $page->setConfig('last', '末页');
        $page->setConfig('first', '首页');
        $page->setConfig('theme', '%FIRST% %UP_PAGE% %LINK_PAGE% %DOWN_PAGE% %END%');
        return $page;
    }
```

## sendSms() “发送短信”

```php
    /* 发送短信
     * $phone 接收短信号码
     * $sign 发送短信前面必填
     * $action 类型字段
     * $msg 短信息
     * @return stdClass
     */
    function sendSms($phone,$sign,$action='reg',$msg) {
        Vendor('Aliyundy.api_demo.SmsDemo');
        set_time_limit(0);
//         header('Content-Type: text/plain; charset=utf-8');       
        $code = mt_rand(100000,999999);
        $msgs = '';
        switch($action){ // 短信模板编号
            case 'reg':
                $msgs .= SMS_74265123;
                $arr =  Array("code"=>$code,'product'=>'拾起卖');// 短信模板中字段的值
                break;
            case 'regok':
                $msgs .= SMS_76465035;
                $arr =  Array("appname"=>$msg,'name'=>$phone);// 短信模板中字段的值
                break;
           
        } 
        $response = SmsDemo::sendSms($sign,$msgs,$phone,$arr);

//         dump(json_decode(json_encode($response),true));exit;
        $res = json_decode(json_encode($response),true);
        if($res['Code']=='OK' && $res['Message']=='OK'){//将短信信息存到数据库
            $sms_code = array(
                'sc_mobile'=>$phone,
                'sc_code'=>$code,
                'sc_time'=>time(),
                'sc_action'=>$action,
                'sc_content'=>json_encode($arr),
            
            );
            M('sms_code')->add($sms_code);
            return 1;
        }
        return 0;
    }
```

## checkVerifyCode() “验证码校验”
```php
    /**
     * 验证码校验
     * @param unknown $phone 手机号
     * @param unknown $code  验证码
     * @param string $action  类型
     * @return number
     */
    function checkVerifyCode($phone,$code,$action='forget'){
        $res = M('sms_code')->where(array('sc_mobile'=>$phone,'sc_action'=>$action,'sc_code'=>$code,'sc_used'=>0))->find();
    
        if ($res){
            M('sms_code')->where(array('sc_id'=>$res['sc_id']))->setField('sc_used',1);
            $data['res'] = 1;
        }else{
            $data['res'] = 0;
        }
        return $data;
    }
```

## ch2arr() “中文拆数组”
```php
    /**
     * @param $str 中文字符串
     * $str = "中国话";
     * @return array {'0'=>'中','1'=>'国','2'=>'话'}
     */
    function ch2arr($str)
    {
        $length = mb_strlen($str, 'utf-8');
        $array = array();
        for ($i=0; $i<$length; $i++)
            $array[] = mb_substr($str, $i, 1, 'utf-8');
        return $array;
    }
```

## getFirstCharter() “php获取中文字符拼音首字母”
```php
    /**
     * @name php获取中文字符拼音首字母
     * @param $str
     * @return null|string
     */
    function getFirstCharter($str){
        if (empty($str)) {
            return '';
        }
        $fchar = ord($str{0});
        if ($fchar >= ord('A') && $fchar <= ord('z')) return strtoupper($str{0});
        $s1 = iconv('UTF-8', 'gb2312', $str);
        $s2 = iconv('gb2312', 'UTF-8', $s1);
        $s = $s2 == $str ? $s1 : $str;
        $asc = ord($s{0}) * 256 + ord($s{1}) - 65536;
        if ($asc >= -20319 && $asc <= -20284) return 'A';
        if ($asc >= -20283 && $asc <= -19776) return 'B';
        if ($asc >= -19775 && $asc <= -19219) return 'C';
        if ($asc >= -19218 && $asc <= -18711) return 'D';
        if ($asc >= -18710 && $asc <= -18527) return 'E';
        if ($asc >= -18526 && $asc <= -18240) return 'F';
        if ($asc >= -18239 && $asc <= -17923) return 'G';
        if ($asc >= -17922 && $asc <= -17418) return 'H';
        if ($asc >= -17417 && $asc <= -16475) return 'J';
        if ($asc >= -16474 && $asc <= -16213) return 'K';
        if ($asc >= -16212 && $asc <= -15641) return 'L';
        if ($asc >= -15640 && $asc <= -15166) return 'M';
        if ($asc >= -15165 && $asc <= -14923) return 'N';
        if ($asc >= -14922 && $asc <= -14915) return 'O';
        if ($asc >= -14914 && $asc <= -14631) return 'P';
        if ($asc >= -14630 && $asc <= -14150) return 'Q';
        if ($asc >= -14149 && $asc <= -14091) return 'R';
        if ($asc >= -14090 && $asc <= -13319) return 'S';
        if ($asc >= -13318 && $asc <= -12839) return 'T';
        if ($asc >= -12838 && $asc <= -12557) return 'W';
        if ($asc >= -12556 && $asc <= -11848) return 'X';
        if ($asc >= -11847 && $asc <= -11056) return 'Y';
        if ($asc >= -11055 && $asc <= -10247) return 'Z';
        return null;
    }
```

## getMapName() “查询当前id所属父级类别的名称”
```php
    /**
     * @param $id post id值
     * @param array $map 返回的数组
     * @param $table  查询的表名
     * @param $table_id_name  表的主键id名
     * @param $name 查询的名称的值
     * @param $pid_name 父类ID名
     * @return array 返回的数组
     */

    function getMapName($id, &$map=array(), $table, $table_id_name, $name, $pid_name){
        $model = M($table);
        $where[$table_id_name] = $id;

        $listName= $model -> where($where) -> getField($name);
        $listPid = $model -> where($where) -> getField($pid_name);
        $map[] =$listName;

        if($listPid>0){
            getMapName($listPid,$map,$table, $table_id_name, $name, $pid_name);
        }
        return $map;
    }
```

## getMapId() “得到当前id的父类id数组集合”
```php
    /**
     * @param $id post id值
     * @param array $map 返回的数组
     * @param $table  查询的表名
     * @param $table_id_name  表的主键id名
     * @param $pid_name 父类ID名
     * @return array 返回的数组
     */
    function getMapId($id,&$map=array(),$table, $table_id_name, $pid_name){
        $model = M($table);
        $where[$table_id_name] = $id;

        $listPid = $model -> where($where) -> getField($pid_name);

        if($listPid>0){
            //self::getMapId($listPid,$map);
            //$this->getMapId($listPid,$map);
            getMapId($listPid,$map,$table,$table_id_name,$pid_name);
            $map[] =$listPid;
        }
        return $map;
    }
```

## getTree() “递归获取列表”
```php
    /**
     *
     * 递归获取列表
     * @param $array
     * @param int $pid 父类id
     * @param int $level 顶级为0
     * @return array
     */
    public function getTree($array, $pid =0, $level = 0){

        //声明静态数组,避免递归调用时,多次声明导致数组覆盖
        static $list = array();
        if($level>=2){
            return $list;
        }
        foreach ($array as $key => $value){
            //第一次遍历,找到父节点为根节点的节点 也就是pid=0的节点
            if ($value['organization_chart_pid'] == $pid){
                //父节点为根节点的节点,级别为0，也就是第一级
                $value['organization_chart_level'] = $level;
                //把数组放到list中
                $list[] = $value;
                //把这个节点从数组中移除,减少后续递归消耗
                unset($array[$key]);
                //开始递归,查找父ID为该节点ID的节点,级别则为原级别+1
                $this->getTree($array, $value['organization_chart_id'], $level+1);
            }
        }
        return $list;
    }
```

# PublicController.class.php

## pageS() “封装的tp的分页”
```php
    /**
     * 封装的tp分页--用tp封装的$page
     * @param $table  表名
     * @param $num    每页显示数量
     * @param string $order  排序规则
     * @param string $field  返回参数
     * @param string $where  查询条件
     * @param string $join   链接表
     * @return mixed  返回 page 和 list
     */
    public function pageS($table, $num, $order='', $field='*', $where='', $join=''){
        $User = M($table); // 实例化User对象
        $count      = $User->join($join)->where($where)->count();// 查询满足要求的总记录数
        $Page       = new \Think\Page($count,$num);// 实例化分页类 传入总记录数和每页显示的记录数
        $show       = $Page->show();// 分页显示输出// 进行分页数据查询 注意limit方法的参数要使用Page类的属性
        $list = $User->join($join)->order($order)->field($field)->where($where)->limit($Page->firstRow.','.$Page->listRows)->select();
        $this->assign('page',$show);// 赋值分页输出
        $this->assign('list',$list);// 赋值分页输出
        return $list;
    }
```

## appPageS() “封装的tp分页，根据传参返回列表”
```php
    /**
     * @param $table  表名
     * @param int $page      传人的页码
     * @param int $limit     每页显示的条数
     * @param string $order  排序方法
     * @param string $field  查询的规则
     * @param string $where  查询的条件
     * @param string $join   链接的表名
     * @return $data['list'] 返回的列表  $data['pages'] 返回的总页数
     */
    public function appPageS($table, $page=1, $limit=30, $order='', $field='*', $where='', $join=''){
        $User = M($table); // 实例化User对象
        $count      = $User->join($join)->where($where)->count();// 查询满足要求的总记录数
        if(!empty($count)){
           // $list = $User->join($join)->order($order)->field($field)->where($where)->page($page)->limit($limit)->select();
           // $list['pages'] = ceil($count/$limit) ;
            $data['list'] = $User->join($join)->order($order)->field($field)->where($where)->page($page)->limit($limit)->select();
            $data['pages'] =  ceil($count/$limit) ;
            return $data;
        }else{
            return null;
        }

    }
```

## EditOk() “增加编辑函数”
```php
    /**
     * 编辑增加操作
     * @param $table "表名"
     * @param string $data array('name'=>'豆豆','password'=>'33')
     * @param string $idname "接收id的字段名"
     * @return bool|mixed  "返回的结果"
     */
    protected function EditOk($table, $data='', $idname='id'){
        $table=M($table);
        $table->data(array());
        $table->create();
        if (!empty($data)) {
            if (!is_array($data)) {
                return false;
            }
            foreach ($data['all'] as $k=>$v) {
                $table->$k=$v;
            }
        }
        if (empty($_REQUEST[$idname])) {
            foreach ($data['add'] as $k=>$v) {
                $table->$k=$v;
            }
            $res=$table->add($data);
        }else{
            foreach ($data['save'] as $k=>$v) {
                $table->$k=$v;
            }
            $res=$table->where(array($table->getPk()=>$_REQUEST[$idname]))->save($data);
        }
        return $res;
    }
```

## Del() “删除”
```php
    /**
     * @param $table 表名
     * @param string $is_del  为空表示真删除
     * @param string $idname  接收的id名称
     * @param string $Tidname 表中的id名称
     * @return bool|mixed
     */
    protected function Del($table, $is_del='', $idname='id', $Tidname=''){
        if (empty($_REQUEST[$idname])) {
            return false;
        }
        $table=M($table);
        $Tidname || $Tidname=$table->getPk();
        if (empty($is_del)) {
            $res=$table->where(array($Tidname=>$_REQUEST[$idname]))->delete();
        }else{
            $res=$table->where(array($Tidname=>$_REQUEST[$idname]))->setField($is_del,1);
        }
        return $res;
    }
```

## DelAll “批量删除”
```php
    /**
     * 批量删除
     * @param $table 表名
     * @param string $idname  接收的id名称
     * @param string $Tidname 表单中的id名称
     * @return bool|mixed
     */
    protected function DelAll($table, $idname='id', $Tidname=''){
        if (empty($table)) {
            return false;
        }
        $ids=(is_array($_REQUEST[$idname])?implode(',',$_REQUEST[$idname]):$_REQUEST[$idname]);
        if (empty($ids)) {
            return false;
        }
        $table=M($table);
        $Tidname || $Tidname=$table->getPk();
        $res=$table->where(array($Tidname=>array('in',$ids)))->delete();
        return $res;
    }
```

## detail() “详情”
```php
    /**
     * @param $table 表单
     * @param string $idname 接收的id名称
     * @return bool|mixed|string 返回的数据
     */
    protected function detail($table, $idname='id'){
        if (empty($table)) {
            return false;
        }
        $table=M($table);
        if (empty($_REQUEST[$idname])) {
            $res='';
        }else{
            $res=$table->where(array($table->getPk()=>$_REQUEST[$idname]))->find();
        }
        return $res;
    }
```

## addAll()批量增加
```php
    /**
     * @param $model   实例化模型$model = M('user','t_','DB_CONFIG1');
     * @param $data    批量插入的二维数组
     * @param $idname  接收的id名称
     * @param $Tidname 数据库中的id名称
     * @return mixed   返回的参数
     */
    public function addAll($model, $data, $idname, $Tidname){
        foreach ($data as $k=>$v){
            if (!empty($v[$idname])){
                $res = $model -> where(array($Tidname=>$v[$idname])) ->save($v);
            }else{
                $res = $model -> add($v);
            }
        }
        return $res;
    }
```


