#  X-SAST 轻量、快速、多语言、代码审计工具  
原创 酒零  NOVASEC   2025-07-07 00:00  
  
![](https://mmbiz.qpic.cn/mmbiz_png/toroKEibicmZD0ibJzndB4iaGqAz3XQcz5jpm0eibx8VFSMMic3VoEIib9hQuCIiaYtia01KPrkibztzNgA9ymhdnp73Vt0g/640?wx_fmt=png&from=appmsg "")  
  
  
**△△△点击上方“蓝字”关注我****们了解更多精彩**  
  
  
  
  
  
**0x00 前言**  
  
****  
**免责声明：继续阅读文章视为您已同意[****NOVASEC免责声明****].**  
  
  
在咨询并试用了大量代码审计工具后，我们开发了一个  
多语言代码安全审计工具，满足渗透测试过程中快速的代码审计。  
  
  
这是当前已知的 公开的 第一款 基于规则的多语言GUI代码审计工具。  
  
  
1  
  
# X-SAST 多语言代码安全审计工具  
  
## 项目背景  
  
  
在渗透测试和安全评估过程中，高效快速的代码审计是一项关键能力。  
  
现有工具如 Seay 和 Fortify SCA 各有局限性：  
- Seay：仅适用于 PHP 环境，正则匹配结果冗余，缺乏有效的审计进度管理  
  
- Fortify SCA：虽支持多语言，但扫描耗时长，且常因语法环境缺失导致分析失败  
  
渗透测试人员通常采用关键函数追溯方法进行快速代码审计。  
  
经过市场调研，发现缺乏一款真正实用的、面向安全从业者的多语言快速代码审计工具。  
  
市场上主要是 semgrep、IRFIY、CodeQL 等基于 DSL 的分析工具，但这些工具分析速度相对较慢，不适合快速审计场景。  
  
基于这一需求缺口，我们开发了 X-SAST，一款结合 Seay 和 Fortify 优势的多语言快速代码审计工具套件。  
## 核心功能  
  
1. **基于正则的通用引擎**  
：支持多种编程语言，实现快速代码扫描  
  
1. **AI 辅助验证**  
：利用人工智能技术自动筛选和验证扫描结果，快速筛选误报结果  
  
1. **专业审计 UI**  
：提供直观的界面，支持人工进一步分析和筛选 AI 验证后的结果  
  
1. **IDE 联动**  
：与主流 IDE 集成，提供代码流追踪功能  
  
1. **代码流分析**  
：深入分析代码结构，PHP 部分已完成实现，其他语言持续开发中  
  
## 工具套件组成  
  
  
X-SAST 工具套件包含以下核心组件：  
- **X1_checker**  
：自动化规则检查引擎，支持命令行操作  
  
![](https://mmbiz.qpic.cn/mmbiz_png/toroKEibicmZD0ibJzndB4iaGqAz3XQcz5jp3KMq43So41qCWPXqkc1Qiaow7qS8O2ZYBlHBVr7DcuQyO5Af3x61EWg/640?wx_fmt=png&from=appmsg "")  
  
- **X2_verifier**  
：AI 增强的验证引擎，集成了 X1_checker 的所有功能，并增加 AI 辅助分析能力  
  
![](https://mmbiz.qpic.cn/mmbiz_png/toroKEibicmZD0ibJzndB4iaGqAz3XQcz5jp9icCCzVL3rElGsicThia6NxRLJrlhUDdJBf38iakAEX6U1yPiapeoev9RYg/640?wx_fmt=png&from=appmsg "")  
  
- **X3_auditor**  
：人工审计图形界面，用于深入分析和确认扫描结果  
  
- X3载入X1结果示例   
填充规则审计部分  
  
![](https://mmbiz.qpic.cn/mmbiz_png/toroKEibicmZABNHic6I2k8vayq06GiaN0PjKtXH1Nqqibe1PEavRR6VrlpJH8EuibHn54XAhEZ2Oj5OgAvVD22OAk9g/640?wx_fmt=png&from=appmsg "")  
  
  
- X3载入X2结果示例 填充AI过滤部分  
  
![](https://mmbiz.qpic.cn/mmbiz_png/toroKEibicmZABNHic6I2k8vayq06GiaN0PjfEcSPrkWbYxY5l1QxKHiaOd2ibOCgngOmEhWxlMPicX620Cic9iaAMeLk0w/640?wx_fmt=png&from=appmsg "")  
- **X9_editor**  
：规则编辑器，支持自定义安全规则的创建和管理  
  
![](https://mmbiz.qpic.cn/mmbiz_png/toroKEibicmZD0ibJzndB4iaGqAz3XQcz5jp5hkydPsCvXRdT37qgVYsIgxO0cejmohchqAOOmAbZ1ibJMJw4LFvI4g/640?wx_fmt=png&from=appmsg "")  
  
## 系统架构  
  
  
X-SAST 采用模块化设计，各组件可独立使用，也可协同工作：  
1. 基础扫描层（X1_checker）：基于正则规则引擎，高效识别潜在安全问题  
  
1. AI 增强层（X2_verifier）：利用多种 AI 模型验证扫描结果，降低误报率  
  
1. 人工审计层（X3_auditor）：提供专业的审计界面，支持深入分析和确认  
  
1. 规则管理层（X9_editor）：支持规则的创建、编辑和管理  
  
## 发布说明  
  
  
目前 X-SAST 整体以二进制形式发布，部分核心程序开源，其他部分提供 Windows 平台的预编译可执行程序：  
- **X1_checker.exe**  
：规则检查引擎（命令行工具）  
  
- **X2_verifier.exe**  
：AI 验证引擎（命令行工具，需授权）  
  
- **X3_auditor.exe**  
：人工审计工具（GUI 程序）  
  
- **X9_editor.exe**  
：规则编辑器（GUI 程序）  
  
对于其他平台需求，可联系开发团队获取支持。  
  
因开发投入较大，其中 X2_verifier 组件将考虑采用授权模式提供。  
  
X3_auditor 支持直接导入 X1_checker 的审计结果, X2_verifier 存在与否不影响用户使用.  
## 目标用户  
  
  
X-SAST 专为以下用户设计：  
- 安全研究人员  
  
- 渗透测试工程师  
  
- 需要快速评估代码安全性的开发团队  
  
- 安全教育和培训机构  
  
## 产品优势  
  
- **多语言**  
：适用于各种主流编程语言  
  
- **高效性**  
：比传统 SAST 工具速度更快，适合快速安全评估  
  
- **准确性**  
：AI 辅助验证和人工审计能够大幅降低误报率和漏报  
  
- **易用性**  
：简洁的命令行工具和直观的图形界面，降低使用门槛  
  
- **可扩展性**  
：支持自定义规则，满足特定安全需求  
  
## 技术规格  
  
- 开发语言：Python  
  
- 分发方式：Pyinstaller 打包的可执行程序  
  
- 支持平台：Windows（其他平台可定制）  
  
- AI 模型支持：集成多种本地和云端 AI 模型  
  
## 联系方式  
  
  
如需获取更多信息、技术支持或定制服务，请通过以下方式联系我们：  
  
NOVASEC微信公众号或通过社交信息联系【酒零】  
  
X-SAST - 为安全专业人员打造的新一代快速代码审计利器  
  
  
  
![](https://mmbiz.qpic.cn/mmbiz_gif/icZfUh6Tsbv0xAFjs5qQlsFCCmymOS3Vq8v6OSKDP0pw3aoCD4OTqojr5NMysBOcoMehddw6JUqYXVuurThNLsQ/640?wx_fmt=gif&wxfrom=5&wx_lazy=1 "")  
  
  
  
END  
  
  
  
如您有任何  
投稿、  
问题  
、需求、建议  
  
请  
NOVASEC  
公众号  
后台  
留言  
！  
  
![](https://mmbiz.qpic.cn/mmbiz_png/toroKEibicmZCP3AeicSCQAYIOvxVDSRUxpiadmBKZ8gtggx02BmG1WwCqoM23l72qV8AiabXSRKjGmk8S1HS1nTjXw/640?wx_fmt=png "")  
  
或添  
加 NOVASEC   
联系人  
  
![](https://mmbiz.qpic.cn/mmbiz_jpg/toroKEibicmZD7m4f7uBkNfCG8BjypNEukN0Ht6Ha0XsryrmS5PAmaVeyzb3JzsH5ibx6DmpHq9e8agwMkccrwNSQ/640?wx_fmt=jpeg "微信图片_20201214143605.jpg")  
  
  
感谢您对我们的支持、点赞和关注  
  
加入我们与萌新一起成长吧！  
  
  
**本团队任何技术及文件仅用于学习分享，请勿用于任何违法活动，感谢大家的支持！！**  
  
  
  
  
  
