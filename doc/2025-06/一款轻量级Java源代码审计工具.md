#  一款轻量级Java源代码审计工具  
Tr0e  夜组安全   2025-06-26 00:03  
  
免责声明  
  
由于传播、利用本公众号夜组安全所提供的信息而造成的任何直接或者间接的后果及损失，均由使用者本人负责，公众号夜组安全及作者不为此承担任何责任，一旦造成后果请自行承担！如有侵权烦请告知，我们会立即删除并致歉。谢谢！  
**所有工具安全性自测！！！VX：**  
**baobeiaini_ya**  
  
朋友们现在只对常读和星标的公众号才展示大图推送，建议大家把  
**夜组安全**  
“**设为星标**  
”，  
否则可能就看不到了啦！  
  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icZ1W9s2Jp2WrOMH4AFgkSfEFMOvvFuVKmDYdQjwJ9ekMm4jiasmWhBicHJngFY1USGOZfd3Xg4k3iamUOT5DcodvA/640?wx_fmt=png&from=appmsg "")  
  
## 工具介绍  
  
JavaSinkTracer 是基于 Python javalang 库开发的一款轻量级 Java 源代码漏洞审计工具。  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icZ1W9s2Jp2XJKoG6oBRkx2IKN1kiaQKBk442p8qiaOk6zWiaonb2EMiajq8E4OyTNcCX2b2PrklM5Dfd7FMeicofHzw/640?wx_fmt=png&from=appmsg "")  
## 工具功能  
  
此工具当前已实现的功能有：  
- 可在 Sink 配置文件中，自由拓展待检测的漏洞危险函数；  
  
- 自动构建 Java 源代码中所有函数的互相调用关系图call graph；  
  
- 从 Sink 点反向追溯到 Source 函数（可从配置文件的“depth”自定义追溯深度），提取调用链；  
  
- 程序自动识别“污点传播链路”上“不包含任何参数”的函数，排除不可能存在外部可控变量的链路；  
  
- 借助 Python javalang 三方库，自动提取每条“函数级污点链路”上所有函数的源代码，方便分析审计；  
  
- 自动生成漏洞报告（Md和html两种格式），Html报告支持漏洞栏目导航、漏洞代码高亮、代码变量跟踪等；  
  
## 工具用法  
  
基础环境：  
```
pip install -r requirements.txt
```  
  
运行命令：  
```
python JavaSinkTracer.py [-h] [-p PROJECTPATH] [-o OUTPUTPATH]
```  
## 效果  
  
以主流的开源 Java 漏洞靶场 java-sec-code 为例，扫描过程和漏洞报告效果如下：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icZ1W9s2Jp2XJKoG6oBRkx2IKN1kiaQKBkX1ViaAicpQ6VzJrGB3STXoky6yT80P5mbXzgicMkrwcR9McDlWDvOibktw/640?wx_fmt=png&from=appmsg "")  
  
漏洞的报告展示，以其中一个涉及到跨文件 SSRF 漏洞链路为例：  
  
![](https://mmbiz.qpic.cn/sz_mmbiz_png/icZ1W9s2Jp2XJKoG6oBRkx2IKN1kiaQKBk442p8qiaOk6zWiaonb2EMiajq8E4OyTNcCX2b2PrklM5Dfd7FMeicofHzw/640?wx_fmt=png&from=appmsg "")  
>   
> 【注意】工具扫描出来的漏洞数量，受Sink点配置文件的影响。上述扫描过程我仅配置了简单的RCE、反序列化、SSRF漏洞Sink点的扫描结果，故并未覆盖此靶场的全量漏洞。  
  
  
## 工具获取  
  
  
  
点击关注下方名片  
进入公众号  
  
回复关键字【  
250626  
】获取  
下载链接  
  
  
## 往期精彩  
  
  
往期推荐  
  
[360主动防御绕过抓取hash](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247494603&idx=1&sn=d47d9f357bae6aa718d2657badd732b9&chksm=c36baf33f41c26255fb7af314533d21c6d0c6396fbe88d35804de2f5c75a9d4496ced9ee3d48&scene=21#wechat_redirect)  
  
  
[Gathery 是一款信息收集与分析工具，集成了多种实用的信息收集功能和小工具](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247494602&idx=1&sn=8b431a1ba6554e4b2ec000b5cc11f110&chksm=c36baf32f41c2624611632d49b49801a6984d23be17389e5887e096d07422abe9ad23cc96b29&scene=21#wechat_redirect)  
  
  
[风险隐患报告生成器](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247494564&idx=1&sn=7a609107c661a38aa867ef5535a8c79a&chksm=c36baf5cf41c264ac96ac7e7ab5fd038edb3abe6efc140b2d5173ab367a120e7a857854e1bfe&scene=21#wechat_redirect)  
  
  
[Nuclei POC 漏洞验证可视化工具 | 更新V3.0.0](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247494556&idx=1&sn=c79aa1a66057f879b6885cd60d3cbd62&chksm=c36baf64f41c2672b05b76f754cbe2ae0c7d680f42f845067e92cb8b8868723fedd24de6da68&scene=21#wechat_redirect)  
  
  
[Android 远控工具-安卓C2](http://mp.weixin.qq.com/s?__biz=Mzk0ODM0NDIxNQ==&mid=2247494549&idx=1&sn=a72e5f2f2587c1bf7dd6a93b3b907c34&chksm=c36baf6df41c267b50d468872ba1567a3d3f8a9ed9cb803aa83ee05b9646968de8d6681deac2&scene=21#wechat_redirect)  
  
  
![](https://mmbiz.qpic.cn/mmbiz_png/OAmMqjhMehrtxRQaYnbrvafmXHe0AwWLr2mdZxcg9wia7gVTfBbpfT6kR2xkjzsZ6bTTu5YCbytuoshPcddfsNg/640?wx_fmt=other&wxfrom=5&wx_lazy=1&wx_co=1&random=0.8399406679299557&tp=webp "")  
  
