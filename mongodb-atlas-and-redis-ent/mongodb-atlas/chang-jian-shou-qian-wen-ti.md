---
description: 内容仅供参考，请以官方文档为准
---

# 常见售前问题

### 为什么选择 MongoDB Atlas 而不是 MongoDB Cloud Manager？

Atlas 提供托管和简化的体验。 Atlas 用户可以访问精选的配置和基础设施选项。可用的 Atlas 配置和基础设施选项可能无法提供某些用户所需的灵活性。例如，Atlas 需要 TLS 来实现集群连接，并且没有提供禁用 TLS 的选项。 Atlas 适合希望管理更少移动部件的用户，使开发人员和数据库管理员能够提高工作效率。



### MongoDB Altas 与社区版的区别？



<figure><img src="../../.gitbook/assets/image (2) (2).png" alt=""><figcaption></figcaption></figure>

### MongoDB与Firestore的对比？



<figure><img src="../../.gitbook/assets/image (37).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

### 有哪几种不同的订阅模式？

1. **承诺计费订阅**

如果您的订阅包括一个期限的承诺，您可以在atlas界面上查看您的承诺用量和相关支出及剩余量。进度条可以直观显示出完成承诺的进展程度。

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

2. **弹性计费订阅**

按照实际使用量来收费

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>



### 我可以将现有的 MongoDB 部署引入 MongoDB Atlas 进行管理吗？

否。但是您可以将现有 MongoDB 部署中的数据上传到 MongoDB Atlas。

* 您可以使用 Atlas 中的实时迁移功能从源副本集迁移到 Atlas 副本集集群。
* 您可以使用 Atlas 中的实时迁移功能从源分片集群迁移到 Atlas 分片集群。
* 您可以使用 mongomirror 将数据从现有的 MongoDB 副本集迁移到 MongoDB Atlas。
* 您可以使用 mongodump 和 mongorestore 从现有的独立、副本集和分片集群中为 MongoDB Atlas 集群播种。

### 我可以在区域之间迁移吗？

是的。您可以在该集群的原始云服务提供商内更改给定集群的一个或多个区域。 MongoDB Atlas 使用滚动迁移策略将节点从原始区域移动到新区域，从而保持集群可用性。

如果您需要在不同云服务提供商的区域之间迁移数据，您可以：

* 执行实时迁移，或
* 使用 mongomirror 迁移，或
* 使用 mongorestore 为目标集群播种，或
* 将备份从源集群还原到目标集群。

### Atlas支持跨地域部署吗？

是的。在创建或扩展部署时，您可以指定其他区域以实现高可用性或本地读取。

Atlas 不支持跨云服务提供商部署。

### 我可以暂停或停止我的 Atlas 集群吗？

您一次最多可以暂停 M10+ 付费集群 30 天。 Atlas 会在 30 天后自动恢复集群。

### 我可以在 Atlas 分片集群中预拆分块吗？

Atlas 管理员数据库用户角色具有在空分片集合中预拆分块的必要权限。

### MongoDB Atlas 能否部署超过 50 个分片的集群？

虽然开箱即用的 MongoDB Atlas 允许选择多达 50 个分片，但对超过 50 个分片感兴趣的客户应该咨询 MongoDB。

### MongoDB Atlas 如何实现高可用性？

Atlas 集群使用 MongoDB 的复制功能来提供高可用性。所有 Atlas 集群要么是副本集，要么是分片集群，其中每个分片都是一个副本集。

Atlas 使用滚动升级策略来执行维护或基础架构操作，例如应用安全补丁或扩展 Atlas 集群。滚动升级策略确保集群可以处理大部分维护或基础设施操作的读写。在滚动升级过程中：

* Atlas 将更改应用于集群中的每个辅助节点。
* Atlas 指示主节点降级到辅助状态并触发新主节点的选举。
* 一旦集群有了新的主节点，Atlas 就会将更改应用于之前的主节点。

当集群选择一个新的主节点时，应用程序必须保持写操作。在此期间集群可以继续处理二次读操作。 Atlas 集群上的选举通常会在几秒钟内完成。网络延迟等因素可能会延长完成副本集选举所需的时间，这反过来会影响您的集群在没有主节点的情况下可以运行的时间。这些因素取决于您的特定集群架构。

### 什么是分析节点？

适用于 M10+ 集群。

分析节点是专门的只读节点，用于隔离您不想影响操作工作负载的查询。它们对于处理分析数据很有用，例如由 BI 工具执行的报告查询。

分析节点和只读节点配置有不同的副本集标签，允许您将查询定向到所需的节点类型和区域。

多区域集群上最多可以有 50 个节点。在该限制内，没有分析节点的最大数量。

分析节点无法为集群的可用性做出贡献，因为它们无法参与选举，也无法成为其集群的主要节点。

### 单个 Atlas 集群可以有多少个集合？

虽然单个集群中的集合数量没有硬性限制，但如果集群服务于大量集合和索引，则集群的性能可能会降低。更大的集合对性能有更大的影响。

Atlas 集群层建议的最大集合和索引组合数如下：

<figure><img src="https://lh3.googleusercontent.com/XL683HUpXwXe4NrBTi7b3U8oZdU7oSDmTfrOGWm0XoXlVLbThqVq_n8Dfx8YsJiOvF6QRym1cuTfedi4fSJxqCdZC7_DRrCXxpKV6_-qtpJY4tnCRKQzuNOvawFpUJrQ1RBMP9iaqs0bskZa8zqMTcksQQA6wnUjxOvrcsKFHzA5XHX75G4UgcHnKFWPPgk" alt=""><figcaption></figcaption></figure>

### Atlas 集群使用哪些版本的 MongoDB？

Atlas 支持创建具有以下层级和 MongoDB 版本的集群

<figure><img src="https://lh3.googleusercontent.com/GoPHWOUmh-tlJ3qfFoAJPEVW4-TK1akOcVjwA6jJW2Fms1e_PZWkvjZ3516HoAjVohJReQia1Uazklekrh5dbdE-n_lRmCrtrWyK19NbJQdMRSKcMhTw26MgQzL99TER__xH9vGR9mcyYQkHhqLaw8sDtQ8ZU7p7Yc0wwqVNXpojrZYpuKeLqldy6526Dt0" alt=""><figcaption></figcaption></figure>

随着新维护版本可用，Atlas 通过滚动过程升级到这些版本以保持集群可用性。

### MongoDB什么时候升级免费和共享层集群的数据库版本？

在多个补丁版本可用于该版本后，Atlas 将免费和共享层集群升级到最新的 MongoDB 版本。





### 使用接近生命周期结束的 MongoDB 版本的 Atlas 集群会发生什么情况？ 

MongoDB 会在 MongoDB 版本到期前至少六个月向您发送电子邮件通知。在您收到此通知几个月后，Atlas：

* 不再允许您使用生命周期结束的版本部署新集群。
* 通知您版本截止日期。在截止日期之后，除非您请求并获得扩展批准，否则 Atlas 会将您的集群升级到下一个 MongoDB 版本。

例子

当 MongoDB 4.0 生命周期结束时，Atlas 会将运行 MongoDB 4.0 的每个集群升级到 MongoDB 4.2。

如果您在项目设置中配置了一个，则此升级会在您的维护窗口内发生。

在大多数情况下，此升级不会导致停机或对您的应用程序产生负面影响。您应该在截止日期之前升级您的集群，以确保您的服务和应用程序不会因与新 MongoDB 版本不兼容而出现停机或其他问题。\
