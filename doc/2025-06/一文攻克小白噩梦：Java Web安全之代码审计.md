#  一文攻克小白噩梦：Java Web安全之代码审计  
原创 rrr0ya1  黑曜网安实验室   2025-06-13 08:16  
  
前言  
  
很多同学反馈代码审计在学习过程经常让人十分头痛，本文总结了java代码审计的思路以及常见的漏洞。  
  
一.何为代码审计  
  
通俗的说Java代码审计就是通过审计Java代码来发现Java应用程序自身中存在的安全问题，由于Java本身是编译型语言，所以即便只有class文件的情况下我们依然可以对Java代码进行审计。对于未编译的Java源代码文件我们可以直接阅读其源码，而对于已编译的class或者jar文件我们就需要进行反编译了。  
  
Java代码审计其本身并无多大难度，只要熟练掌握审计流程和常见的漏洞审计技巧就可比较轻松的完成代码审计工作了。但是Java代码审计的方式绝不仅仅是使用某款审计工具扫描一下整个Java项目代码就可以完事了，一些业务逻辑和程序架构复杂的系统代码审计就非常需要审计者掌握一定的Java基础并具有具有一定的审计经验、技巧甚至是对Java架构有较深入的理解和实践才能更加深入的发现安全问题。  
  
二.Java代码审计  
  
通常我喜欢把代码审计的方向分为业务层安全问题、代码实现和服务架构安全问题。  
  
1.业务层安全常见问题  
  
业务层的安全问题集中在业务逻辑和越权问题上，我们在代码审计的过程中尽可能的去理解系统的业务流程以便于发现隐藏在业务中的安全问题。  
  
以下是一些常见的逻辑漏洞：  
```
1.用户登陆、用户注册、找回密码等功能中密码信息未采用加密算法。
2.用户登陆、用户注册、找回密码等功能中未采用验证码或验证码未做安全刷新(未刷新Session中验证码的值)导致的撞库、密码爆破漏洞。
3.找回密码逻辑问题(如:可直接跳过验证逻辑直接发包修改)。
4.手机、邮箱验证、找回密码等涉及到动态验证码等功能未限制验证码失败次数、验证码有效期、验证码长度过短导致的验证码爆破问题。
5.充值、付款等功能调用了第三方支付系统未正确校验接口(如:1分钱买IPhone X)。
6.后端采用了ORM框架更新操作时因处理不当导致可以更新用户表任意字段(如:用户注册、用户个人资料修改时可以直接创建管理员账号或其他越权修改操作)。
7.后端采用了ORM框架查询数据时因处理不当导致可以接收任何参数导致的越权查询、敏感信息查询等安全问题。
8.用户中心转账、修改个人资料、密码、退出登陆等功能未采用验证码或Token机制导致存在CSRF漏洞。
9.后端服务过于信任前端，重要的参数和业务逻辑只做了前端验证(如:文件上传功能的文件类型只在JS中验证、后端不从Session中获取用户ID、用户名而是直接接收客户端请求的参数导致的越权问题)。
10.用户身份信息认证逻辑问题(如:后台系统自动登陆时直接读取Cookie中的用户名、用户权限不做验证)。
11.重要接口采用ID自增、ID可预测并且云端未验证参数有效性导致的越权访问、信息泄漏问题(如:任意用户订单越权访问)。
12.条件竞争问题，某些关键业务(如:用户转账)不支持并发、分布式部署时不支持锁的操作等。
13.重要接口未限制请求频率，导致短信、邮件、电话、私信等信息轰炸。
14.敏感信息未保护，如Cookie中直接存储用户密码等重要信息。
15.弱加密算法、弱密钥，如勿把Base64当成数据加密方式、重要算法密钥采用弱口令如123456。
16.后端无异常处理机制、未自定义50X错误页面,服务器异常导致敏感信息泄漏(如:数据库信息、网站绝对路径等)。
17.使用DWR框架开发时前后端不分漏洞(如:DWR直接调用数据库信息把用户登陆逻辑直接放到了前端来做)。
```  
  
  
2.  
代码实现常见问题  
  
代码审计的核心是寻找代码中程序实现的安全问题，通常我们会把代码审计的重心放在SQL注入、文件上传、命令执行、任意文件读写等直接威胁到服务器安全的漏洞上，因为这一类的漏洞杀伤力极大也是最为致命的。  
  
SQL注入  
### 成因  
  
未过滤的用户输入直接拼接至SQL语句，导致恶意SQL执行。  
### 高危场景  
- • MyBatis：使用${}  
动态拼接参数（如like '%${title}%'  
）  
  
- • Hibernate：createQuery  
拼接字符串  
  
- • 原生JDBC：字符串拼接SQL语句  
  
### 错误示例  
```
// MyBatis错误示例@Select("SELECT * FROM user WHERE username = '${username}'")User getUserByUsername(String username);// JDBC错误示例String sql = "SELECT * FROM product WHERE price > " + price;Statement stmt = conn.createStatement();ResultSet rs = stmt.executeQuery(sql);
```  
### 审计技巧  
- • 搜索输入拼接符号：全局搜索+  
、concat()  
、${}  
、append()  
  
- • 检查动态排序：MyBatis中是否存在order by ${time}  
  
- • 框架特性：Hibernate的createQuery  
是否使用参数绑定  
  
### 修复方案  
- • 强制使用预编译：MyBatis中优先使用#{}  
，JDBC中使用PreparedStatement  
  
- • 输入校验：对参数进行正则过滤（如仅允许数字）  
  
### 正确示例  
```
// MyBatis正确示例@Select("SELECT * FROM user WHERE username = #{username}")User getUserByUsername(String username);// JDBC正确示例String sql = "SELECT * FROM product WHERE price > ?";PreparedStatement stmt = conn.prepareStatement(sql);stmt.setDouble(1, price);
```  
## XXE  
### 成因  
  
允许解析外部实体，导致文件读取、SSRF等风险。  
### 高危接口示例  
```
// 使用DOM4J解析XML（未禁用DTD）SAXReader reader = new SAXReader();Document doc = reader.read(new File("user.xml"));
```  
### 审计技巧  
- • 搜索任务XML解析器：全局搜索DocumentBuilder  
、SAXReader  
  
- • 检查安全配置：确认是否禁用DTD和外部实体  
  
### 修复方案  
- • 禁用外部实体：通过设置FEATURE_SECURE_PROCESSING  
  
- • 使用安全库：如Jackson的XmlMapper  
默认禁用外部实体  
  
### 正确配置示例  
```
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
```  
##   
## SSRF  
### 成因  
  
未校验请求URL，攻击者可访问内网资源。  
### 高危代码示例  
```
URL url = new URL(request.getParameter("url"));HttpURLConnection conn = (HttpURLConnection) url.openConnection();
```  
### 审计技巧  
- • 搜索文件网络请求方法：全局搜索URL.openConnection()  
、HttpClient.execute()  
  
- • 协议限制检查：确认是否禁止file://  
、ftp://  
等协议  
  
### 修复方案  
- • 域名/IP白名单：允许仅访问指定域名  
  
### 校验域名示例  
```
if (!url.getHost().endsWith(".example.com")) {    throw new SecurityException("非法域名");}
```  
##   
## 文件操作漏洞  
### 成因  
  
未校验文件路径，导致任意文件读写。  
### 高危场景示例  
```
File file = new File("/uploads/" + userInput);InputStream is = new FileInputStream(file);
```  
### 审计技巧  
- • 搜索图片文件IO类：全局搜索FileInputStream  
、FileOutputStream  
  
- • 路径拼接检查：确认是否使用getCanonicalPath()  
规范化路径  
  
### 修复方案  
- • 限制文件目录：仅允许访问固定根目录下的文件  
  
### 安全路径拼接示例  
```
File baseDir = new File("/safe/uploads/");File targetFile = new File(baseDir, userInput);if (!targetFile.getCanonicalPath().startsWith(baseDir.getCanonicalPath())) {    throw new SecurityException("非法路径");}
```  
##   
## 命令执行  
### 成因  
  
用户输入直接拼接至系统命令。  
### 高危代码示例  
```
Process process = Runtime.getRuntime().exec("ping " + userInput);
```  
### 审计技巧  
- • 搜索和命令执行函数：全局搜索Runtime.exec()  
、ProcessBuilder.start()  
  
- • 特殊符号过滤：检查是否过滤&  
、;  
、|  
  
### 修复方案  
- • 参数白名单：仅允许特定参数格式  
  
### 正则过滤示例  
```
if (!Pattern.matches("^((25[0-5]|2[0-4]\\d|[01]?\\d\\d?)\\.){3}(25[0-5]|2[0-4]\\d|[01]?\\d\\d?)$", userInput)) {    throw new SecurityException("非法IP");}
```  
##   
## 反序列化漏洞  
### 成因  
  
反序列化不可信数据导致恶意代码执行。  
### 高危接口示例  
```
// Fastjson反序列化User user = JSON.parseObject(jsonStr, User.class);
```  
### 审计技巧  
- • 搜索功能反序列化方法：全局搜索ObjectInputStream.readObject()  
、JSON.parseObject()  
  
- • 依赖版本检查：确认Fastjson、XStream是否使用安全版本  
  
### 修复方案  
- • 升级依赖：Fastjson ≥1.2.83，XStream ≥1.4.19  
  
- • 启用安全模式：Fastjson使用Feature.SupportNonPublicField  
  
##   
## 中间件漏洞  
### 成因  
  
使用存在已知漏洞的中间件版本。  
### 常见漏洞  
- • Apache Shiro：反序列化漏洞（默认密钥）  
  
- • WebLogic：SSRF、反序列化漏洞（CVE-2020-2551）  
  
### 审计技巧  
- • 检查依赖版本：通过pom.xml  
确认中间件版本  
  
- • CVE匹配：比对已知漏洞库（如NVD）  
  
### 修复方案  
- • 更新中间件：Shiro ≥1.8.0，WebLogic ≥12.2.1.4  
  
- • 删除无用组件：移除未使用的Struts2、Log4j 1.x  
  
## 其他常见漏洞  
  
Java 文件名空字节截断漏洞(%00 Null Bytes)  
  
空字节截断漏洞漏洞在诸多编程语言中都存在，究其根本是Java在调用文件系统(C实现)读写文件时导致的漏洞，并不是Java本身的安全问题。不过好在高版本的JDK在处理文件时已经把空字节文件名进行了安全检测处理。  
  
2013年9月10日发布的Java SE 7 Update 40修复了空字节截断这个历史遗留问题。此次更新在java.io.File类中添加了一个isInvalid方法，专门检测文件名中是否包含了空字节。  
  
![Java Web安全-代码审计](https://mmbiz.qpic.cn/sz_mmbiz_jpg/XvnzjMsl2uohDFCaHjkp0olIl2OOExdOMHic7YFJ45ictFnicckXBCKawVzNgm2ZLajBPz9YaddBIOfIY8IuPjoxQ/640?wx_fmt=jpeg&from=appmsg "")  
  
修复的JDK版本所有跟文件名相关的操作都调用了isInvalid方法检测，防止空字节截断。  
  
![Java Web安全-代码审计](https://mmbiz.qpic.cn/sz_mmbiz_jpg/XvnzjMsl2uohDFCaHjkp0olIl2OOExdOliaWsPslntL5eUBZYoIpQ5FdrtW2pAVAMxDKBcDoVgC0cmbsz4zZhRg/640?wx_fmt=jpeg&from=appmsg "")  
  
修复前(Java SE 7 Update 25)和修复后(Java SE 7 Update 40)的对比会发现Java SE 7 Update 25中的java.io.File类中并未添加\u0000的检测。  
  
![Java Web安全-代码审计](https://mmbiz.qpic.cn/sz_mmbiz_jpg/XvnzjMsl2uohDFCaHjkp0olIl2OOExdOT2CuAX0UlyKGPQplzQXb7zxwhTp5K352BXglXNnnllgbNMzzTHX7AQ/640?wx_fmt=jpeg&from=appmsg "")  
  
受空字节截断影响的JDK版本范围:JDK<1.7.40,单是JDK7于2011年07月28日发布至2013年09月10日发表Java SE 7 Update 40这两年多期间受影响的就有16个版本，值得注意的是JDK1.6虽然JDK7修复之后发布了数十个版本，但是并没有任何一个版本修复过这个问题，而JDK8发布时间在JDK7修复以后所以并不受此漏洞影响。  
  
  
任意文件读取漏洞  
  
任意文件读取漏洞即因为没有验证请求的资源文件是否合法导致的，此类漏洞在Java中有着较高的几率出现，任意文件读取漏洞看似很简单，但是在这个问题上翻车的有不乏一些知名的中间件:Weblogic、Tomcat、Resin又或者是主流MVC框架:Spring MVC、Struts2。所以在审计文件读取功能的时候要非常仔细，或许很容易就会有意想不到的收获！  
  
任意文件读取示例代码file-read.jsp:  
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="java.io.ByteArrayOutputStream" %>
<%@ page import="java.io.File" %>
<%@ page import="java.io.FileInputStream" %>
<%
    File file = new File(request.getParameter("path"));
    FileInputStream fis = new FileInputStream(file);
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    byte[] b = new byte[1024];
    int a = -1;
    while ((a = fis.read(b)) != -1) {
        baos.write(b, 0, a);
    }
    out.write("<pre>" + new String(baos.toByteArray()) + "</pre>");
    fis.close();
%>
```  
  
访问file-read.jsp文件即可读取任意文件:  
http://localhost:8080/file/file-read.jsp?path=/etc/passwd  
  
  
  
文末福利  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_jpg/XvnzjMsl2upA9icOic6sZYibUXsN3LAyBczliagZXibtbgrEfp04ibrQpPXs8jEV3beaE1fhupWNPy5Xod51cCmU3sCg/640?wx_fmt=jpeg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp "")  
  
大家对网络安全或者渗透测试感兴趣的话，可以添加vx：  
  
Free_rrr0ya1  
  
无偿领取全套25年最新网络安全面经、网络安全学习指南资料。  
  
![在这里插入图片描述](https://mmbiz.qpic.cn/sz_mmbiz_jpg/XvnzjMsl2upA9icOic6sZYibUXsN3LAyBczAYEA206fib09fAGbtYu4fkyfx9A5MrlstBK423wFe65NBPJUBoRwHoQ/640?wx_fmt=jpeg&watermark=1&wxfrom=5&wx_lazy=1&tp=webp "")  
  
  
  
