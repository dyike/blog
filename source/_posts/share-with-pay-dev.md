---
title: 【05-20】讨论一下支付系统的功能设计或相关开发经验
tags:
  - 支付功能
date: 2015-05-20 21:08:00
---

 来自黑夜路人技术群讨论分享内容。

【今日话题】

讨论一下支付系统的功能设计或相关开发经验 - darkmi

1\. 资金的数据库变更最基础要记得使用事务

然后还有是数据锁问题

比如一个逻辑是用户购买东西

先判断用户余额够不够东西价格，如果够就购买，不够就返回错误

这个时候获取用户余额的时候就要记得对这个用户的资金进行读写锁

然后整个事务结束后解锁

这样才不会有脏数据和重复消费 - 轩脉刃

2\. &#x1f633; 性能很差吧 - diwayou@高朋

回: 不会，又不是锁所有用户账户 - 轩脉刃

回: 当账号是一个商家的账号 就会有性能问题 如果是个人用户不会有问题 - diwayou@高朋

回: 好吧，具体场景具体分析了 - 轩脉刃

3\. 安全问题要保证！日志要记录好 - 若凡

4\. 用户资金流水一定要记录，记录订单号，交易金额，账户资金快照，等信息。 - 丁靖

问: 资金快照是什么？ - lemon...

回: 就是这次交易后，用户帐户上的资金余额. 如果不是这种业务场景，就不需要记录了 - 丁靖

5\. 问: 主要是分布式事务咋实现好？xopen的两阶段性能太差 - 张勇

回: 分布式事务不好做啊 如果业务能够接受异步稍微好处理点 如果必需同步 那只能损失性能了

如果做异步 可能需要分步消耗 例如 1.冻结 准备资源 2.真正消耗

冻结资源如何释放可以发起方通知 也可以接受放自己超时释放 - diwayou@高鹏

6\. 前一段时间看过支付宝程立之前分享的分布式事务介绍，觉得他们就是自己实现一个事务管理器，事务是业务级别的，每个业务提供tcc操作和提交、回滚。每个数据库都是用的本地事务。

也就是说每个业务自己实现回滚操作。

都是2阶段的事务，还有infoq的微信红包的一个演讲也使用的自己实现的2阶段提交。 - 张勇

7\. 自己回滚需要设置好超时时间 最好是主动通知释放 - diwayou@高鹏

8\. 微信的http://www.infoq.com/cn/presentations/mobile-Internet-massive-access-system-design

微信红包的 - 张勇

9\. 大规模SOA系统中的分布事务处事_程立 http://wenku[ baidu![](http://cdncache-a.akamaihd.net/items/it/img/arrow-10x10.png)](#94081119 "Click to Continue &gt; by DealExpress").com/link?url=r6DpwDbFeUG93YX_gpLTlegBirLxu0gst50c50yJ6oYYItSqDn7P1DRtKY5JrYTISCCjalBk6DkE4K-wKVm0VBp8qUUvQVuV1ojoZsTJetC - 张勇

10\. 支付去年我做过，主要是用redis 分布锁+mysql 事务。mysql 分库分表，用户之间隔离，互不影响。

每天定时对账，报警。- 天天

11\. 能详细点儿不？比如手user1在A库，user2在B库，user1给user2转帐，这种事务怎么处理的？还是不处理靠对账来解决？ - 张勇

回: 首先，我那个系统不涉及用户间交易。如果用户间，涉及到库事务，这个mysql 暂时做不了。你可以看一下redis 实现的一个简单的分布锁，如果跨库更新失败写日志，标记两个锁定。

日志这块也非常重要，恢复，对账大部分靠它。。 - 天天

回: 恩。日志、流水是对账的依据，相当于数据库的redo log，比较重要。 - 张勇

12\. 实时到账还是延迟到账？如果是延迟到账 用一个带事务的queue来做 实时到账 两阶段来 - diwayou@高鹏

13\. 之前简单问过支付宝的朋友，他们用java 写的分布式事物，很强大。java 也有开源组件。 - 天天

14\. 开源的好像都是x/open JTA规范实现的，性能不能保证。- 张勇

15\. 问: 分布式事务的原理是什么 - 陈一回

回: 这本书，有试读。http://item.jd.com/11622772.html 里面讲了一些分布式事务的问题

原理的话可以看看两阶段提交和三阶段提交的资料。 - 张勇

16\. 实际这个就是业务能接受异步的都争取用异步 要不然性能很难保证 - diwayou@高鹏

17\. 这里还有一篇文章：http://coolshell.cn/articles/10910.html (注: 分布式系统的事务处理) - 张勇

18\. 人手少做个很痛苦，幸好我们有收盘锁定期，还能补补，不然，一步错，步步错，会被骂死。

做支付老老实实做好测试，别什么敏捷，天天版本的，这不是闹着玩的，我们上线钱前产品临时加了个规则，我们没经验，最后就死在这个规则上。。。 - 天天

19\. 问个细节点的问题，金额是怎么存储的？一般是按分存储还是按元来存储？ - darkmi

回: 分 - diwayou@高鹏

回: @diwayou@高鹏  用分是考虑跨项目,跨语言吧. 如果只是php+mysql,mysql用decimal存储,php用bcmath计算小数,结果是正常的 - twin

回: 嗯 用分容易些 如果用小数 各个语言计算都需要特殊处理 - @diwayou@高鹏

20\. 支付宝TCC示例 http://www.docin.com/p-866220638.html - diwayou@高鹏

21\. 支付系统的物理架构大概什么样？我看有很多和银行对接的部分，有很多要求，比如有的是win,有的是linux,需要配ftp,有的还要拉专线

木有支付行业的？ 很想了解啊 - platoli

22\. 银行或第三方支付需要与人行对接，人行在两个银行之间，负责资金转发和清算等

行内支付系统，负责准备人行接口需要数据，并且处理往账来账，并给核心记账系统提供记账数据

与人行对接，需要专门支付前置，专门加密等 - 召阳

23\. 移动端支付信息一般用oauth加密传输 - 张静小朋友

回: oauth加密是啥？ - 唐毅

回: OAuth 是授权，协议里面包含秘钥验证， - JoJo

【分享链接】

1\. MySQL索引原理及慢查询优化 http://mp.weixin.qq.com/s?__biz=MjM5MDYwMjM3Nw==&amp;mid=203600712&amp;idx=1&amp;sn=055c10be105ee20e564efa067b0f195e - 黑夜路人

2\. PHP中实现异步调用多线程程序代码 http://mp.weixin.qq.com/s?__biz=MzAwNjMxMTA5Mw==&amp;mid=210157381&amp;idx=1&amp;sn=adb9928e1b9b3ec9c10cb8b064e6ffef - 黑夜路人

注: 他说的都是基于io复用 和多线程没有关系 - 惠新宸

3\. Google基础设施副总裁：容器是计算的未来 http://mp.weixin.qq.com/s?__biz=MjM5ODM3MzkyNQ==&amp;mid=214930786&amp;idx=1&amp;sn=8806346868f72b590719e456776e029a - 黑夜路人

4\. Unique Key Generate Server https:/[ github![](http://cdncache-a.akamaihd.net/items/it/img/arrow-10x10.png)](#32285254 "Click to Continue &gt; by DealExpress").com/liexusong/ukg - 凹凸曼

5\. PHP unique ID generator https:/[ github![](http://cdncache-a.akamaihd.net/items/it/img/arrow-10x10.png)](#11868467 "Click to Continue &gt; by DealExpress").com/liexusong/ukey - 凹凸曼

6\. PHP 远程 DoS 漏洞深入分析及防护方案 http://www.nsfocus.com.cn/report/php_multipart-form-data_remote_dos_vulnerability_analysis_protection.pdf - 袁峰
