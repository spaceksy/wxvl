#  【代码审计】某系统存在文件覆盖导致getshell  
 不秃头的安全   2025-06-19 10:14  
  
## 某系统存在文件覆盖导致getshell  
```
前言：本文中涉及到的相关技术或工具仅限技术研究与讨论，严禁用于非法用途，否则产生的一切后果自行承担，如有侵权请私聊删除。还在学怎么挖通用漏洞和src吗？知识星球有什么，文章下限时优惠卷~~想要入交流群在最下方，考安全证书请联系vx咨询。
```  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/DicRqXXQJ6fWaOQhXOf0cibja9IiaN9XvbmE5jLs5PByGh6NEsygeaAwonoQf8yKn2DtF6ZC0FshCkm3icyxic2lWqQ/640?wx_fmt=other&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp "")  
  
  
  
**一、漏洞审计**  
  
****  
问题出现在admin/filemanager接口的index方法  
  
**漏洞条件1**  
、  
  
获取了请求参数的path  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yY0dJe75e29iadM1gXic0kPvPE4hibOsziazIEu7A5iabbeYZ0cESdlsHesDg/640?wx_fmt=png&from=appmsg "")  
  
  
**漏洞条件2**  
、  
  
  
在该方法里，可以看到当请求方法为post，而且mode=savefile，needpath与content参数不为空时，就会进入fm.savefile方法里。  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYubmL32tjiasW2yDA276nVcx7lKYJ8aNuJ0eKTuicl3mp1aeQRjzPqAIg/640?wx_fmt=png&from=appmsg "")  
  
  
**漏洞执行、**  
  
  
FileManagerUtils.writeString像是将content里的内容写入到getRealFilePath()里面。  
  
先查看backupfile方法是干什么的。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yY0OsO9HicgXv6VNRiaCeCIwkcEg6Wwickth6pjrGRLJg3cgDZgTpUMuicDA/640?wx_fmt=png&from=appmsg "")  
  
  
targetPath路径是由默认系统配置路径和src参数路径组成的  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYMdm30TblsKxhYud7iaW4Qur3e6GxMs2HxVGsugFvF3vqsLjEm69oP2Q/640?wx_fmt=png&from=appmsg "")  
  
  
跟进copyfile方法，该方法是将src路径的文件内容复制到target路径的文件内容，如果复制失败，即源文件（src）不存在，会导致FileNotFoundException或者在transferFrom操作时抛出IOException，使方法执行停止。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYQzRExI6ac8nzCIpD8yoSDcumrFj1OaicgA1icSib8ryW805TlMCZg0tLA/640?wx_fmt=png&from=appmsg "")  
  
  
回到savefile，只要backupfile不报错，就可以执行FileManagerUtils.writeString。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYDKnbzzRrQh1vbgF8ZadMjJuOia70pZBEZibpF5wYklXa4PHl3nqHKzpA/640?wx_fmt=png&from=appmsg "")  
  
  
其中getRealFilePath方法获取的路径是系统配置路径+请求参数的path路径。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYjXk8jwWDVicRb1IhILD7vScI6MvFytEqxgZ9MROIUPbhriaibNFa3Mcaw/640?wx_fmt=png&from=appmsg "")  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYkRr9ChPv4ViakjjI6tQPN4QzhyibqsYZnCNhicR2m8VdOvFN0jLeBmI4w/640?wx_fmt=png&from=appmsg "")  
  
通过断点可知系统配置路径如图所示。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYeNzWQiayBb5lNyAsazfq4vuHUL6sldBz290Ofn9feicgKXRaA9qBliacw/640?wx_fmt=png&from=appmsg "")  
  
继续跟进  
writeString  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYuiavfric9rnrdI0198rPRwjtr5uBwFziciaQLOH6GuBlTGVfw00UWpPohQ/640?wx_fmt=png&from=appmsg "")  
  
  
跟进write，执行了文件写入的操作，即将conten的参数内容写入到路径文件（系统配置路径+path路径）。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYjE8gnicRIAOQuYibxSYp4YyIpQcBTx8XU1J8bEzWtnOib0O1YFF0icibKDQ/640?wx_fmt=png&from=appmsg "")  
  
  
**漏洞思路整理：**  
  
post请求接口的index方法，带上参数mode=savefile，path=系统存在的文件，content=要写入的文件内容，就可以执行文件覆盖漏洞。  
  
  
**二、漏洞复现**  
  
查看系统路径有什么默认存在的文件,刚好根目录存在login.jsp文件  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yY1czqUcuRzEeXr0msATHkjicucC4icCG4icE6IKxZxcfeckLxLibfQ41YBA/640?wx_fmt=png&from=appmsg "")  
  
  
构造请求数据包，其中content是url编码后的webshell代码  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYhBtibtGlEqicJib20buH9TuQaibwFicaG368WYJPxoXtLHAhRqMSgMUPyeA/640?wx_fmt=png&from=appmsg "")  
  
  
查看login.jsp,覆盖成功。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYETJbS3OfrHmNliaWJQXbozhefXQsMKFeiaPibb438RkrhPX9p7VVtf4Tg/640?wx_fmt=png&from=appmsg "")  
  
  
先访问试试能否解析,访问文件网站显示404,文件是肯定存在的，但是网站显示404，去代码中查看原因。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYknicLJladTTPo5xSu8KqfurVsgjJGyibteNh7xy9C7QVbt4Tu6tOEHdQ/640?wx_fmt=png&from=appmsg "")  
  
  
找到了filter过滤器文件，这段代码看起来是判断请求url路径是否为jsp，为jsp就报错。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYI7OQ2PlLAZyGBX9vpGIas7wUZIAEmVq9YpZfoEFsWZgFuialtVPVbjA/640?wx_fmt=png&from=appmsg "")  
  
  
查看isJsp方法，该方法是先判断请求url最后一位是否为x,然后再判断前一位是否为p，再判断前一位是否为s，然后是j，如果满足就return false，网站返回404。  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYZ9KhZea3PgnUWjYjMRasys2ccp3lMrGjPMFfkTQuiatCCxEIXPFKavQ/640?wx_fmt=png&from=appmsg "")  
  
  
通过;号的方法绕过黑名单限制。看起来是成功解析了![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYn2xWDm1wNZ2eLxUSGKVsLeXzmmmEeRl9I6IhDKLBrAiaWLnpRoYxJbw/640?wx_fmt=png&from=appmsg "")  
  
  
  
webshell连接成功  
  
![](https://mmbiz.qpic.cn/mmbiz_png/fp1zrrz1Uibibk2D7Jic74KIU7bicCVMm0yYicH0j8euDiaOxHUpDGw0EKhciaWk8H2e8zq0SYna23c1gIMHEahzWE34A/640?wx_fmt=png&from=appmsg "")  
  
往期推荐：  
  
[红队 AI红队平台 viper更新至3.1.7](https://mp.weixin.qq.com/s?__biz=Mzg3NzkwMTYyOQ==&mid=2247489503&idx=1&sn=ee2f5cc861160e16d8c5611634f34f87&scene=21#wechat_redirect)  
  
  
[渗透测试 ｜ 从jeecg接口泄露到任意管理员用户接管+SQL注入漏洞](https://mp.weixin.qq.com/s?__biz=Mzg3NzkwMTYyOQ==&mid=2247489466&idx=1&sn=3630915a5db00418f1e7b18541bbb73a&scene=21#wechat_redirect)  
  
  
[【SRC】分享某单位众测中的一次高危](https://mp.weixin.qq.com/s?__biz=Mzg3NzkwMTYyOQ==&mid=2247489452&idx=1&sn=1e4111d7a22cfe751f8ae3082fc35837&scene=21#wechat_redirect)  
  
  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/DicRqXXQJ6fUM2XnKDfzXgrWSFPlzVLmlrLSL63hXjKficic2XU4N9r222YGcYiaVZGZhW42XDLfFkTiaVMsbRvrQPw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1&watermark=1&tp=webp "")  
  
**关于我们:**  
  
感谢各位大佬们关注-不秃头的安全，后续会坚持更新渗透漏洞思路分享、安全测试、好用工具分享以及挖掘SRC思路等文章，同时会组织不定期抽奖，希望能得到各位的关注与支持，考证请加联系vx咨询。  
  
  
## 1. 需要考以下各类安全证书的可以联系  
  
学生pte超低价，绝对低价绝对优惠，CISP、PTE/PTS、DSG、IRE/IRS、NISP、PMP、CCSK、CISSP、ISO27001、IT服务项目经理等等巨优惠，想加群下方链接：  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/DicRqXXQJ6fUM2XnKDfzXgrWSFPlzVLmlCt5Te9O6jAVj3qFqR1q5MKPNZTP5Z2nGlaDezVaKd4baGialetAOULg/640?wx_fmt=other&from=appmsg&watermark=1&tp=webp&wxfrom=5&wx_lazy=1 "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/DicRqXXQJ6fUM2XnKDfzXgrWSFPlzVLml111eQH8roRkP0oWcDaPSMJ8Gj7woZnVwFiaz4uwSlELEyLViadQXNFXA/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&watermark=1&tp=webp "")  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/DicRqXXQJ6fUM2XnKDfzXgrWSFPlzVLml9STP6xZkiaPok60H0IZaCn8IYcqLibIr3gy70YU0K8nZELmwpwbqAXnQ/640?wx_fmt=jpeg&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&watermark=1&tp=webp "")  
## 2. 需要入星球的可以私聊优惠  
  
投稿文章可免费进星球~~星球里有什么？  
```
1、维护更新src、cnxd、cnnxd专项漏洞知识库，包含原理、挖掘技巧、实战案例2、fafo/零零信安/QUAKE 高级会员key3、POC及CXXD及CNNXD通用报告详情分享思路4、知识星球专属微信“内部圈子交流群”5、分享src挖掘技巧tips6、最新新鲜工具分享7、不定期有工作招聘内推（工作/护网内推）8、攻防演练资源分享(免杀，溯源，钓鱼等)9、19个专栏会持续更新~提前续费有优惠，好用不贵很实惠
```  
  
  
![图片](https://mmbiz.qpic.cn/sz_mmbiz_jpg/DicRqXXQJ6fUM2XnKDfzXgrWSFPlzVLmlj8JPum8LApLyciatkseusdSyc5QdDfYn5GcyjOibjySAz0oZZHicTSAEQ/640?wx_fmt=jpeg&from=appmsg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp "")  
## 3、其他合作（合法合规）  
  
1、承接各种安全项目，需要攻防团队或岗位招聘都可代发、代招（灰黑勿扰）；  
  
2、各位安全老板需要文章推广的请私聊，承接合法合规推广文章发布，可直发、可按产品编辑推广；  
合作、推广代发、安全项目、岗位代招均可发布![图片](https://mmbiz.qpic.cn/sz_mmbiz_png/DicRqXXQJ6fUM2XnKDfzXgrWSFPlzVLml8D4wUUicic6F7D1bia6rtxziaEV5vVtib08CsfJL7aPFnzyRPZbYicahOflw/640?wx_fmt=png&from=appmsg&wxfrom=5&wx_lazy=1&wx_co=1&watermark=1&tp=webp "")  
  
  
  
