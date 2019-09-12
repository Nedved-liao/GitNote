---
title: IBM WebSphere 远程代码执行漏洞安全预警通告
date: 2018-09-13 18:17:42
tags: CSDN迁移
---
   近日，IBM发布安全通告称修复了一个WebSphere Application Server中一个潜在的远程代码执行漏洞（CVE-2018-1567）。攻击者可以构造一个恶意的序列化对象，随后通过SOAP连接器来执行任意JAVA代码.目前没有更多漏洞细节披露

 该漏洞 CVSS 得分 9.8， 攻击者可在不经过身份认证的情况下远程对 We bSphere 服务器发起攻击，造成远程代码执行，最终导致服务器被控制。

 

 

 
## 受影响的版本

 IBM WebSphere Application Server:

 
  * Version 9.0 
  * Version 8.5 
  * Version 8.0 
  * Version 7.0 
## 不受影响的版本

 
  * Version >= 9.0.0.10 
  * Version >= 8.5.5.15 注：官方已不再完整支持v7,v8，用户需根据官方说明进行更新。

 
> https://www-01.ibm.com/support/docview.wss?uid=ibm10730503
> 
>  
 
## 解决方案

 IBM官方已经发布了新版本修复了上述漏洞，请受影响的用户尽快更新升级进行防护。

 用户可以使用interim fix, Fix Pack 或者包含APARs PI95973的PTF进行升级，具体操作步骤请参考官方说明中Remediation/Fixes部分：

 
> https://www-01.ibm.com/support/docview.wss?uid=swg22016254
> 
>  
 

   
 