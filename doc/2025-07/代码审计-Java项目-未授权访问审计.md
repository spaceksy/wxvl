#  代码审计-Java项目-未授权访问审计  
原创 小黑子安全666  小黑子安全   2025-07-18 04:23  
  
代码审计必备知识点：  
  
1  
、  
代码审计开始前准备：  
  
环境搭建使用，工具插件安装使用，掌握各种漏洞原理及利用  
,  
代码开发类知识点。  
  
2、代码审计前信息收集：  
  
审计目标的程序名，版本，当前环境(系统,中间件,脚本语言等信息),各种插件等。  
  
3、代码审计挖掘漏洞根本：  
  
可控变量及特定函数，不存在过滤或过滤不严谨可以绕过导致的安全漏洞。  
  
4、代码审计展开计划：  
  
审计项目漏洞原理->审计思路->完整源码->应用框架->验证并利用漏洞。  
  
代码审计两种方法  
：  
  
功能点或关键字分析可能存在  
的  
漏洞  
  
-  
抓包或搜索关键字找到代码出处及对应文件  
。  
  
-  
追踪过滤或接收的数据函数，寻找触发此函数或代码的地方进行触发测试。  
  
  
-常规或部分M  
VC  
模型源码可以采用关键字的搜索挖掘思路。  
  
  
-框架  
  
MV  
C   
墨香源码一般会采用功能点分析抓包追踪挖掘思路。  
  
1.  
搜索关键字找敏感函数  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCCYEwz6P0TOt7ep6Ys8tgPr8cLJ3Udc4mHiclRROb5ricE1YiaFRS2cIicYQ/640?wx_fmt=png&from=appmsg "")  
  
2.  
根据目标功能点判断可能存在的漏洞  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCCrruViafRH38poSzo4pDY9lfe5Dp516ok91AvRDd7LBB88sN3C8ws0gA/640?wx_fmt=png&from=appmsg "")  
  
Javaweb身份验证访问控制  
：  
  
审计此类漏洞的第一步就是要搞清楚代码的验证方式  
  
开发做  
    
访问控制身份验证  
    
有几种技术方案实现：  
  
1、自写传统验证代码——登录性判断文件查看代码  
  
2、Shiro框架引用——看配置看引用看外部库  
  
3、Filter过滤器——看配置看过滤器目录分析代码  
  
4、JWT技术——看引用看外部库搜关键函数代码  
  
案例：  
CNVD-Tumo-  
未授权访问  
-Shiro  
框架引用  
  
1.  
根据  
cnvd  
公开的漏洞信息得知  
Tumo Blog  
博客存在未授权访问漏洞。可以利用漏洞删除别人评论。  
  
项目下载地址：  
https  
://  
github  
.  
com  
/  
TyCoding  
/  
tumo  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCCUE5ichayLQ6DbtvJlUaHwRqnibsS44poVJz52SiaszVn1dBlMNOsBNgBA/640?wx_fmt=png&from=appmsg "")  
  
2.  
下载好源码，将源码导入到  
IntelliJ IDEA   
，首先查看  
pom.xml  
文件，发现里面引用了  
Shiro  
安全框架。而  
shiro是apache的一个开源框架  
,  
是一个权限管理的框架  
,  
实现 用户认证和用户授权的  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCCRzTFRorYTVpPtCXNJzVZZctBUvqQhcZyYCdER6Q87504XKjwBz936Q/640?wx_fmt=png&from=appmsg "")  
  
3.  
知道项目使用了  
S  
hiro  
框架，就可以  
查看  
shiro  
配置文件信息。  
  
其中：  
  
tumo  
.  
shiro  
.  
anon_url  
    
—  
>  
代表不需要鉴权的配置  
  
**  
  
—  
>  
表示该接口下的所有接口  
  
也就是说tumo  
.  
shiro  
.  
anon_url中配置的目录都可以被普通用户任意操作。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCC4Is6c5IsG6oHDc5TVDlQyHZqo1m7d1glbUHhamWOHMCWamibALibw9Xg/640?wx_fmt=png&from=appmsg "")  
  
4.  
查找tumo  
.  
shiro  
.  
anon_url配置的目录下有没有删除评论的操作。  
  
最终在  
/  
comment   
目录下找到了删除评论的操作。利用  
id  
接收要删除的评论，使用  
delete  
请求方式触发。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCCtK0NLib8pErtZicxzlBjGuy7HAWpicn0yrOib3nze3Uia1NfvQPRsxKmBGA/640?wx_fmt=png&from=appmsg "")  
  
5.  
来到网站任意文章下写一条评论  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCCicF6V7Qt0wyRfmOQcr7s9ZOnZr8y2wHYiatsC5hGK7lgibfMlwxAU4Wgw/640?wx_fmt=png&from=appmsg "")  
  
6.  
审查元素，获取评论的  
id  
值为：  
8  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCCKeBYWVq2Y9DXwZaUTpyuyUPJnYHNJbUBKSLr4Dm30hibkvnd27icm5RA/640?wx_fmt=png&from=appmsg "")  
  
7.  
访问删除评论的路由地址：  
/comment/8  
      
使用的是  
get  
请求方式，所以并没有触发删除，返回的是评论信息  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCCRA0jdEch2Ocskmpp0YiaThPQNZ42hChc0ohbCvDT8xkOqIqet9lU9Vg/640?wx_fmt=png&from=appmsg "")  
  
8.  
再次访问，使用  
burp  
抓包，修改为  
delete  
请求方式  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCClCm13EUk0DiciawKSQ2Q8B5nG3qIZEbEuY8F6Q0puou8a3hGicAZKhvdg/640?wx_fmt=png&from=appmsg "")  
  
9..  
放包，返回——成功  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCCA1lvZibe2ZvictZH7NkRRW1YmHauYvcrg3AuZIZl6k0vu5oibUHoicgw5w/640?wx_fmt=png&from=appmsg "")  
  
10.  
回到刚刚的文章评论处，发现评论已经不见了。删除评论成功。这个漏洞是因为管理员错误的配置  
S  
hiro  
配置文件导致的。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCIlnNxeT7FCVA8Hp7B3HBpCC0jwSxOFe6icVTEjM6CO8JRlZVb36Ga7LicZdhpiaW39rfqQiaWXWpWWNBw/640?wx_fmt=png&from=appmsg "")  
  
推荐一下作者最新研发的yakit被动漏洞检测插件，可挖掘企业src漏洞。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/b9ibb0kbHCImmI8E9Zf6JqGRia9JWljiaavtzDPaxIm6PXM8IuzuAB6ViceOVwpEMvHH403GNQp59nju2Q49TvqOpg/640?wx_fmt=png&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "")  
  
以上漏洞都是不需要任何技巧的，作者只是开启插件在目标网站用鼠标“点点点点”就挖掘出这么多漏洞，完全  
  
“零基础”“  
  
零成本”挖洞  
  
目前一共开发了四个插件：  
  
被动  
sql  
注入检测  
  
被动  
xss  
扫描优化版  
  
被动目录扫描好人版  
  
被动  
ssrf  
及  
log4j  
检测  
  
插件使用效果：[记一次企业src漏洞挖掘连爆七个漏洞！](https://mp.weixin.qq.com/s?__biz=Mzg5NDg4MzYzNQ==&mid=2247486552&idx=1&sn=41c4e4a40fd104f53e22f5a7143c88ad&scene=21#wechat_redirect)  
  
  
插件使用教程：  
[新一代SQL注入检测技术，小白也能轻松挖到漏洞！](https://mp.weixin.qq.com/s?__biz=Mzg5NDg4MzYzNQ==&mid=2247486516&idx=1&sn=8ce41df1c1c32eddb5762dc6c362a85b&poc_token=HEcld2ij2mPrmNUdYokG5LVG3B9lMQFCmGocR1XA&scene=21#wechat_redirect)  
  
  
知识星球——小黑子安全圈  
[  
精华版  
]  
    
开业大吉！  
  
每一个插件都是非常实用的，有没有用作者也已经通过  
  
企业  
src   
漏洞的挖掘来证明了，并且只需要开启插件  
  
点击鼠标就可以全自动挖掘漏洞。  
  
需要获取插件的小伙伴可以扫描下方二维码加入我的知识星球，星球  
 99  
元  
/  
年  
  
，前  
50  
个加入的  
 77  
元  
/  
年。  
  
加入知识星球的同学会提供  
  
yakit  
  
安装  
  
和  
  
插件  
  
使用教程。  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/b9ibb0kbHCImmI8E9Zf6JqGRia9JWljiaavyafDg4ZciajrrTwTH4VWgO5MnmzRk1FMbiaZT2mQxbjb1JJicMDQLXDlw/640?wx_fmt=jpeg&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp "")  
  
本星球只提供精华内容，没有烂大街的东西。会持续更新  
yakit  
插件和各种漏洞漏洞探针和利用工具，哪怕你是什么都不懂的小白用了插件点点鼠标就能挖到漏洞。  
  
注！！！红包返现！！！拉新活动！！！  
  
星球接受使用插件挖到漏洞的投稿，我审核通过会返  
30  
红包，直接微信转，文章我会发公众号  
(  
每人仅限一次  
)  
。  
  
拉新人加入星球待满三天也会返  
20  
红包  
(  
微信直接转  
)  
。插件会一直优化和上新，欢迎大家加入星球。  
  
  
  
