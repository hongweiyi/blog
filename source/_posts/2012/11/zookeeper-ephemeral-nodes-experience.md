title: Zookeeper Ephemeral结点使用心得
tags:
  - ZooKeeper
id: 619
categories:
  - 技术分享
date: 2012-11-05 20:33:28
---

公司里面在拿Zookeeper做命名服务，通过使用ZK，前端只需要根据指定的ZK地址获得相应的资源或服务的后端服务器地址即可，而后端服务器需要做的仅仅是将自己的地址注册到ZK上作为一个Ephemeral结点即可。（虽然是挺方便后端扩容，但是我个人不太建议直接上ZK，否则开发成本会增加）
<!--more-->  > **P.S.:**Ephemeral结点在Apache Zookeeper中是一个临时结点，这些结点只要创建它的结点session不挂，它就一直存在，当session中止了，结点也就删除了。  

**问题：**

在开发的时候遇到了一个奇怪的问题，当某个后端快速重启之后，该后端的结点信息过一段时间后会被删除，这样就导致了后端服务永远无法被前端访问到。

**原因：**

查了资料后得知，如果在你的session中，ephemeral结点不是由你创建的，你的session就不会拥有该结点，所以当拥有该结点的session终止（expire）了，该结点也就销毁了。那么，如果不是你显式的删除该结点的话，就只能由ZK帮你终止它，在会话超时之后ZK就自动删除结点。如果在会话还未超时的过程中（一般是30s），你重启后端服务器的话，就会导致我所说的情况。

**解决方案：**

Apache提供了几个patch，也有人提供了一些解决方案，均是显式的终止session。但是后端服务器挂了，显式终止一般是没用的。找到的这个方法是比较靠谱的，那就是在创建结点前，先删除之前的结点：   <table border="1" cellspacing="0" cellpadding="0" width="637"><tbody>       <tr>         <td valign="top" width="635">           <p> 1\. try {

 2.&#160;&#160; zk.delete(path)

 3\. } catch {

 4.&#160;&#160; e: NoNodeException =&gt; // do nothing

 5\. }

 6\. zk.create(path, data, CreateMode.EPHEMERAL)
         </td>       </tr>     </tbody></table> </p>  > **参考资料：**
> 
> [A Gotcha When Using ZooKeeper Ephemeral Nodes](http://developers.blog.box.com/2012/04/10/a-gotcha-when-using-zookeeper-ephemeral-nodes/)
> 
> [ephemerals handling after restart](http://zookeeper-user.578899.n2.nabble.com/ephemerals-handling-after-restart-td1084715.html)
