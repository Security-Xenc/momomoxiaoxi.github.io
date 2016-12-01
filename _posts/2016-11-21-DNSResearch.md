---
title: 两篇CDN相关的论文阅读
subtitle: CDN路由循环攻击、CDN与HTTPS的冲突
time: 2016.11.21 00:01:00
layout: post
tags:
- Security
- Research

excerpt: 因为毕设选题与CDN相关，所以这两周主要花了一些时间研究CDN与HTTPS的技术。这里分享两篇CDN相关的论文。cdn_loop-final-camera-ready、https-in-cdn


---



# 两篇CDN相关的论文阅读

----

### 论文1:HTTPS Meets CDN

#### 研究背景

CDN和HTTPS是现今应用非常广泛的两个协议，他们单独部署的时候没有问题，但是共同部署时，出现了一些冲突问题。

#### 问题

HTTPS融入CDN后，通信者从两方变成了三方。这与HTTPS本意上的设计有冲突。尤其当网站启用HTTPS并使用带有基于DNS的请求路由的CDN时，可能会发生证书名称不匹配，因为DNS的重定向在HTTPS的身份验证中是透明的。 

1. 前端问题

   前端一般采用两种方式来进行联合部署。

   - 自定义认证
     该方法的问题：1.Web与CDN供应商共享了一个私钥，违反了公钥体制，会造成很多额外的攻击与危险2. 无法独立撤销

   - 分享证书

     该方法问题：1. 共享证书虽然解决了自定义认证的问题，但是会削弱证书安全指标的功能。2. 无法独立撤销

   前端的问题是由于CDN与HTTPS联合部署时，产生的语义冲突。

2. 后端问题

   因为一些CDN产商对该问题的不重视，在实现部署方面存在一些问题。在测试过程中，发现一些产商甚至直接使用http在后端连接，而一些产商虽然使用了HTTPS，但是没有在建立安全通道时建立适当的身份验证，易受到MITM攻击。

#### 解决方案

1. 首先尝试名称约束的技术

   在这种方法中，网站所有者扮演从属CA的角色，向CDN提供商发布证书，限制到所有者的域。 

   然而这种方法存在一些局限性，不适用实际：

   - 在一些流行的浏览器中，可以通过漏洞轻松绕过
   - 运行从属CA会对网站所有者造成沉重的负担
   - 因为沉重的审查和审计责任，商业CA不大有可能有动机允许他们对客户是从属CA



2. 基于DANE为基础提出了一个新的解决方案

    在这个解决方案中，网站所有者可以向他的代表团显示他的TLSA记录，该记录将网站和CDN提供商的证书相关联。 因此，最终用户可以验证原始网站和CDN提供商的身份以及它们之间的委派。 通过分析和实现表明，该解决方案可以有效地解决CDN中的HTTPS问题。

   该方案能满足三大需求：1. 委派令牌不可伪造2. 委托人能独立有效地签发和撤销委托令牌 3. 委托令牌能包括委托人的完整识别，能保持HTTPS证书的有效功能，显示适当的安全指示

   具体解决方案： ![1](1.png)

#### 学习与总结

##### 总结：

这篇论文主要对CDN与HTTPS共同部署时，可能产生的冲突进行了分析。研究发现由于CDN与HTTPS本身语义的冲突和相关产商部署时候的不重视，当下CDN与HTTPS联合部署时会产生很多问题。主要分为前端问题和后端问题。后端问题主要是由于产商的不重视造成的，值得警示与及时修正；前端问题，是由于基本语义的冲突。CDN产商在缓和这些基本冲突时，主要采用了自定义证书或者分享证书的技术。然而，这些技术在实际运行时，存在一些比较严重的问题。自定义认证技术需要客户与CDN产商共享一个私钥，这严重违反了公钥体制，会造成很多额外的攻击与威胁。分享证书依靠CA颁发对多个域名有效的证书，这种情况下一些证书会退化，无法显示较高级别的绿色标志。此外，这两种技术都无法独立的撤销。为了解决这个问题，作者先考量了名称约束的技术。然而，发现该技术存在一定的局限性，不适宜部署。最后，作者提出了一个基于DANE的轻量级解决方案。该方案在未来DANE广泛部署时，具有极好的优越性，且能较好地解决当下CDN与HTTPS冲突的问题。此外，该方法在开销上也较小，值得推荐。

##### 值得学习的思维模式：

	1. 研究思考方法：可能某样东西单独存在不会有问题，但是共同部署时就会有问题。因为一些协议在最初设计的时候，只考虑了其单独运行的安全性，而没有考虑其与其它协议共同工作时候的场景。这种思考模式值得借鉴。
	2. 信任边界的思考：我们有时候在研究一些东西的时候，会想当然地认为某样事物一定是可信的，一定不会出错的。比如，在一开始的求学时期，我们难免会认为书籍是永远正确的。但是，我们现在可以发现，其实书籍上出现一些谬误的情况，很正常。也就是说，在做一些安全研究的时候，我们应该多去怀疑，多去审视一些东西，不要完全信任某些技术一定是安全的。

#### 一些疑惑：

由于之前确实没有接触过类似这种协议方面的研究，所以对一些东西还存在一些疑惑。

1. 论文作者曾对共享证书的使用情况，进行了监控，以观察CDN在其代理商更新共享证书的频率。我很好奇，论文作者是如何进行监控的？进行一个小脚本的编程监控吗？还是怎么？
2. 对协议编程方面不是很了解。DANE是一个比较新的协议，论文作者以DANE为基础提出了一个解决方案。那么作者对于其提出的解决方案POC测试时，是如何测试的？直接在网络上部署DANE+POC吗？


---

### 论文2:CDN LOOP FINAL CAMERA READY

#### 研究背景

CDN在当下广泛部署，有着无与伦比的优越性。

#### 问题

在CDN上存在一个转发循环攻击的 情况。转发循环攻击允许攻击者用过建立在CDN节点之间的大量循环来大量地消耗CDN资源。对于转发循环攻击，作者进行了广泛的测试，发现尽管一些CDN有内部机制来检查重复的请求，但是攻击者仍可通过某些方法来绕过检查。此外，该攻击的威胁是非常严重的，尤其结合一些特定的CDN功能，如自动服务器探测、转发重试、主动解压缩等等，该攻击的攻击放大因子会大大增强，较快速地放大攻击，造成Dos的效果。

##### 转发循环攻击

1. 自循环，在单个CDN节点内循环
2. CDN内循环，循环在一个CDN供应商中的多个节点
3. 跨越多个CDN循环的CDN间环路
4. CDN大坝洪水攻击，它将转发环路攻击与及时控制的HTTP相应相结合，以显着增加损害

这几个攻击结合一些CDN技术，例如解压操作等等，能大大提高攻击的放大因子，快速地进行DoS攻击。

##### 攻击的成本分析

发现大多数CDN产商为了吸引顾客，都会有免费试用CDN服务的功能。这种功能的提供，可以大大降低攻击的成本，也从一定程度上降低了匿名攻击的成本。攻击者可以通过免费试用账户以匿名形式发动攻击，且成本很低。

##### 影响攻击威力的因素

1. 主机头修改的情况，主机头用以转发。
2. 修改其它头域，这个是某些CDN为了防止循环攻击加的一些头部防御措施，但是研究者发现其能被绕过。
3. 如何处理超时情况。
4. DNS的解析行为。DNS的解析从一定程度上能控制攻击。
5. 流式和非流式传输



#### 解决方案

1. 统一和标准化环路检测头。为所有CDN提供一个规范，用于检测该攻击。
2. 对自定义环路检测头进行模糊处理。例如通过加解密来验证关键字的存在，从而进行检测。这样攻击者就不知道如何构造攻击头。
3. 监控和控制速率。
4. 转发黑名单约束：就是不允许转发到一些目的地，如另外的CDN

#### 学习与总结

##### 总结：

该论文发现了CDN中存在的转发循环攻击，发现该攻击具有十分严重的安全威胁，尤其该攻击的攻击成本很低，且能匿名化。对于该攻击，作者提出了多种环节机制。最根本的解决方案是，各CDN相互协商，确认一个共同的包头来检测攻击，完美解决这个问题。此外，作者还提出了一些环节措施，比如监控和控制速率、进行转发黑名单约束、对自定义环路检测头进行模糊处理。

##### 值得学习的地方：

1. 分析一个攻击的全面性：该论文对CDN循环攻击的分析十分详尽，值得学习。1. 分析了CDN技术的广泛运用（如果出现攻击，就会很威胁）2. 分析该攻击攻击面很广（能攻击所有的CDN产商），尤其可能某个产商的防御检测措施很好，但是由于某个其它CDN产商的不小心，攻击可能还会存在。3. 攻击成本很低，且可匿名化 4. 分析了影响该攻击的一些因素   从这四个方面，可以非常充分地说明该攻击的严重威胁性，从而突出该论文工作的杰出贡献。

##### 一些疑惑：

1. 该论文是否只是验证了攻击的有效性，对于其解决方案因为CDN产商的缘故没有真正编程实现？

----

#### 参考

1. [http://netsec.ccert.edu.cn/duanhx/files/2010/12/https-in-cdn.pdf](http://netsec.ccert.edu.cn/duanhx/files/2010/12/https-in-cdn.pdf)
2. [http://netsec.ccert.edu.cn/duanhx/files/2010/12/cdn_loop-final-camera-ready.pdf](http://netsec.ccert.edu.cn/duanhx/files/2010/12/cdn_loop-final-camera-ready.pdf)