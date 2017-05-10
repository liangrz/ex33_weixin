access_token
--
新增公众号按钮这些操作都需要有access_token才能操纵，所以要进一步开发公众号需要知道access_token的一些基本性质和如何获取

#### 获取access_token <br>
http请求方式: GET<br>
https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=APPID&secret=APPSECRET <br>
APPID和APPSECRET这两个参数在准备工作中已经获取。
```
define(APPID,'appidStr');//appidStr为你appid的字符串
define(APPSECRET,'appSecretStr');//appSecretStr为你appSecretStr的字符串
$url = 'https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid='.APPID.'&secret='.APPSECRET;
$result = file_get_contents($url);
echo $result;
```
如果echo出来的值如下<br>
{"access_token":"ACCESS_TOKEN","expires_in":7200};<br>
那就说明没问题了，ACCESS_TOKEN里面一串杂乱的字符就是access_token<br>
<br>

#### 存取access_token
access_token有两个小时的时限，且公众号每天请求的access_token有限的，如果每次操作都用一次access_token的话，可能会导致access_token不够用，保存它然后直至这个令牌过期都使用这个令牌是个合理的用法。<br>
<br>
步骤如下，不想看步骤的想直接要现成组合的车辆，可以去<a href='#'>get_access_token.class.php<a><br>
先新建一个类，设置需要的属性APPID、APPSECRET、access_token
```
class get_access_token{
	const APPID = "yourappid";
	const APPSECRET = "yourAppsecret";
	public $access_token;
}
```
新建access_token和保存
```
function get_new_access_token(){
  $token_access_url = "https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=".self::APPID."&secret=".self::APPSECRET;
  $res = file_get_contents($token_access_url);
  $result = json_decode($res, true);//由获取中echo的值可以得出这是一个json格式的产物
  $put = $result['access_token'].','.$result['expires_in'].','.time();//添加当前时间才能判断这个令牌什么时候过期
  file_put_contents('access_token.cookie',$put);
  return $this->access_token = $result['access_token'];
}
```
调用cookie
```
function get_cookie_access_token(){
  $res = file_get_contents('access_token.cookie');
  $result = explode(',',$res);
  if(empty($result[2])){
    return $this->get_new_access_token();
  }
  $checktime = time()-$result[2];
  if($checktime<7200){//未过期
    return $this->access_token = $result[0];
  }else{//过期调用新的
    $this->get_new_access_token();
  }
}
```

构建方法的思路，如果有cookie就调用，如果没有就新建
```
function __construct(){
  //获取cookie
  if(!file_exists('access_token.cookie')){
    $this->access_token = $this->get_new_access_token();
  }else{
    $this->access_token = $this->get_cookie_access_token();
  }
}
```
最后整合成车辆<a href='#'>get_access_token.class.php<a><br>
