title: Sina APP Engine初试
tags:
  - SAE
id: 210
categories:
  - 技术分享
date: 2011-11-12 22:38:57
---

**一、 ****官方概述（参考：**[**Sina APP Engine**](http://sae.sina.com.cn)**）**

Sina App Engine（以下简称SAE）是新浪研发中心于2009年8月开始内部开发，并在2009年11月3日正式推出第一个Alpha版本的国内首个公有云计算平台（http://sae.sina.com.cn），SAE是新浪云计算战略的核心组成部分。<!--more-->

SAE作为国内的公有云计算，从开发伊始借鉴吸纳Google、Amazon等国外公司的公有云计算的成功技术经验，并很快推出不同于他们的具有自身特色的云计算平台。SAE选择在国内流行最广的Web开发语言PHP作为首选的支持语言，Web开发者可以在Linux/Mac/Windows上通过SVN、[SDK](http://sae.sina.com.cn/?m=sdk)或者Web版在线代码编辑器进行开发、部署、调试，团队开发时还可以进行成员协作，不同的角色将对代码、项目拥有不同的权限；SAE提供了一系列分布式计算、存储服务供开发者使用，包括分布式文件存储、分布式数据库集群、分布式缓存、分布式定时服务等，这些服务将大大降低开发者的开发成本。同时又由于SAE整体架构的高可靠性和新浪的品牌保证，大大降低了开发者的运营风险。另外，作为典型的云计算，SAE采用“所付即所用，所付仅所用”的计费理念，通过日志和统计中心精确的计算每个应用的资源消耗（包括CPU、内存、磁盘等）。

总之，SAE就是简单高效的分布式Web服务开发、运行平台。

**二、 ****我的理解**

SAE基本上是Google APP Engine（GAE）的一个翻版，其为Web App开发者提供稳定、快捷、透明、可控的服务化的平台，并且减少开发者的开发和维护成本。和GAE一样，它们都属于PaaS平台型云计算服务。

SAE通过Web Service Tool，提供以PHP+HTTP的计算中心，类似于传统的虚拟主机，只是运行环境不同。可能由于国内业界环境不够好，SAE还不足以大量吸引开发者入驻，不过相信通过新浪的努力，SAE能够越做越大。

与传统主机托管服务相比而言，传统托管面向的是硬软件设备，使用者得到的设备的使用权；而SAE面向的服务，使用者得到的是服务的使用权。开发者通过SAE可以在上面通过在线调试、日志分析、协作共享等功能进行web开发。（具体参见：附1 SAE与虚拟主机的区别）

有人可能会想，单纯的一个在线调功还不足以让开发者从虚拟主机转向SAE，那么SAE更大的吸引力是什么呢？SAE最大的吸引力就是它所提供的完整的分布式web服务的解决方案。自己的服务器可能会坏，新浪高可靠性的服务器不会！有了完整的解决方案，开发者只需专注于应用的功能开发，而不必担心故障宕机、服务扩容问题，因为所有这些SAE都已经为用户完整提供。且SAE与虚拟主机采用固定计费的方式不同，SAE采用预充值方式，“所付即所用，所付仅所用”，按需付费更加灵活和节省成本，web服务的一切损耗均提供报表查询和账单汇总，用户一目了然。

开发者不用关心硬件运维方面的事情，可以缩短整个项目开发周期，减少运营成本，这也是云概念提出的一个出发点，不过新浪的路还有很远。

**三、 ****SAE****与GAE的比较**
<table border="1" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td width="94" valign="top"></td>
<td width="184" valign="top">**Sina App Engine**</td>
<td width="200" valign="top">**Google App Engine**</td>
</tr>
<tr>
<td width="94" valign="top">**云计算模型**</td>
<td width="184" valign="top">PaaS</td>
<td width="200" valign="top">PaaS</td>
</tr>
<tr>
<td width="94" valign="top">**支持语言**</td>
<td width="184" valign="top">PHP</td>
<td width="200" valign="top">Java、Python、Go</td>
</tr>
<tr>
<td width="94" valign="top">**数据库支持**</td>
<td width="184" valign="top">MySQL 最大5GB</td>
<td width="200" valign="top">暂不支持</td>
</tr>
<tr>
<td width="94" valign="top">**每个帐号可拥有app****数量**</td>
<td width="184" valign="top">10个</td>
<td width="200" valign="top">10个</td>
</tr>
<tr>
<td width="94" valign="top">**单app****存储限额**</td>
<td width="184" valign="top">最多10GB，单文件不大于4M</td>
<td width="200" valign="top">1GB免费，无最大上限</td>
</tr>
<tr>
<td width="94" valign="top">**代码大小**</td>
<td width="184" valign="top">每帐户不超过100M，单app总代码不超过50M</td>
<td width="200" valign="top">单app不超过150MB</td>
</tr>
<tr>
<td width="94" valign="top">**绑定域名**</td>
<td width="184" valign="top">需另行申请，备案</td>
<td width="200" valign="top">支持</td>
</tr>
<tr>
<td width="94" valign="top">**免费额度**</td>
<td width="184" valign="top">各项服务通过扣除虚拟货币“云豆”实现限额。

成功注册：增500云豆

实名认证：增2000云豆</td>
<td width="200" valign="top">每日6.5 CPU-hours，流入流出带宽各1GB，存储1GB。</td>
</tr>
<tr>
<td width="94" valign="top">**超过免费限额的收费标准**</td>
<td width="184" valign="top">1元=100云豆

赠送机制对创业者帮助很大</td>
<td width="200" valign="top">流入带宽：0.1美元/GB
流出带宽：0.12美元/GB
CPU 时间：0.1美元/CPU小时
存储：0.15美元/GB 每月</td>
</tr>
</tbody>
</table>
**四、 ****SAE****小用感受**

从开始注册，到部署一个WordPress，花了不到半个小时，比在这个空间部署方便快捷很多，而且不要钱，可以考虑将个人博客搬到Sina APP Engine之上。不过美中不足的是，SAE还不支持个人的域名绑定功能，新浪方给出的理由是国家还未出台相应的域备案政策。

还有一个不足的地方，就是平台的开发语言限制。不知新浪出于何种战略考虑，抑或是技术攻关不利，只支持了Php一种语言Web App，这让很多Java、Python程序员神伤啊！

**【附****】**

**SAE****与虚拟主机的区别（摘自官方介绍）**

传统服务托管面向的是硬件软件设备，使用者得到的也是设备的使用权；而SAE面向的服务，使用者得到的是服务的使用权。

传统服务托管不面向开发者，开发者无法在其上享受到开发的乐趣；而SAE的一个重要用户就是web developer，开发者可以在其上通过在线调试、日志分析、协作共享等功能进行web开发。

传统服务托管不提供分布式系统解决方案；而SAE提供的完整的分布式web服务的解决方案，其中不仅仅包括分布式数据库、分布式文件系统，更包括分布式定时器系统、网页抓取服务、图像处理服务等。

传统服务托管不解决域名问题，用户往往烦恼于域名申请；而SAE的用户将自动得到在sinaapp下的二级域名，同时SAE还支持域名cname。

传统服务托管无法保证SLA（Service Level Agreement），硬件故障的成本基本由使用者承担；而SAE保证用户的SLA，用户的web服务自动享有高冗余的前端服务器、享有自动负载均衡系统、服务自动扩展、服务自动收缩等功能。

传统的服务托管采用预付费的方式，费用固定且和实际使用情况无直接关系；而SAE采用预充值方式，“所付即所用，所付仅所用”，web服务的一切损耗均提供报表查询和账单汇总，让用户一目了然。

参考资料：[http://sae.sina.com.cn](http://sae.sina.com.cn/)