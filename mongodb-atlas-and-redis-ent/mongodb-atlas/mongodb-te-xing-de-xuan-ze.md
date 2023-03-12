# MongoDB常见售后问题

#### <mark style="color:red;">问题：我的备份显示为灰色，我无法下载它们</mark> <a href="#issue-my-backups-are-greyed-out-and-i-am-unable-to-download-them" id="issue-my-backups-are-greyed-out-and-i-am-unable-to-download-them"></a>

1. 询问客户是否为其部署启用了 KMS 加密。如果是这样，他们将无法下载备份。
2. 如果未启用 KMS，则很可能仍在拍摄他们的快照。他们将不得不等到它完成。
3. 如果以上两种情况均未发生，则可能存在错误。与客户确认没有表面错误。如果有，让他们提供带有上述信息的错误。

**解决**

1. KMS 无法下载备份。让他们转储`mongodump`数据，在本地恢复，然后获取他们需要的数据（或执行他们需要数据的任何操作）。
2. 他们将不得不等待快照完成。如果快照准备时间很长，则可能存在问题。与客户确认快照的开始时间，并让他们提供给 MongoDB。我们可以检查我们的内部日志记录，看看是什么导致它挂起。
3. MongoDB 将处理持续的调查和潜在的错误。

#### <mark style="color:red;">问题：我的磁盘空间已满且无法自动缩放</mark> <a href="#issue-my-disk-space-has-filled-up-and-is-not-auto-scaling" id="issue-my-disk-space-has-filled-up-and-is-not-auto-scaling"></a>

他们的磁盘可能填满得太快了。我们可以通过让客户提供“已用磁盘空间”和“可用磁盘空间”指标来验证这一点。如果磁盘空间降至 KB 或更少，自动缩放通常不会启动。

**解决**

1. MongoDB 将需要删除缓冲文件并推进计划。升级到 MongoDB。（二级支持）
2. 指示客户限制写入以避免立即填满磁盘，从而重新进入集群无法扩展的状态。

#### <mark style="color:red;">问题：集群运行缓慢</mark> <a href="#issue-cluster-is-operating-sluggishly" id="issue-cluster-is-operating-sluggishly"></a>

让客户提供过去八小时的以下指标；这些将有助于显示工作量的峰值：

1. Opcounters - 这些显示命令、查询、更新、删除、getmores 和插入。如果其中任何一项显示出大幅增加，则可以帮助解释迟缓，例如工作量增加。
2. 文档指标 - 这些显示返回、插入和更新了多少文档。通常如果我们看到大量的返回，或者插入，这也说明工作量增加了很多。
3. 内存 - 如果内存不足，则可能表示存在资源争用，也与工作负载高峰相符。
4. 页面错误 - 如果发生页面错误，它也是内存/资源争用的强烈指示。
5. 磁盘 IOPS - IOPS 尖峰也表示工作负载发生变化。
6. 让他们验证他们的工作量是否发生了变化，或者他们是否异常增加。

**解决**

他们要么需要限制他们的工作量，要么将他们的集群层级提高到一个新的水平。

#### <mark style="color:red;">问题：如何终止 Atlas 中的操作</mark> <a href="#issue-how-to-kill-operations-in-atlas" id="issue-how-to-kill-operations-in-atlas"></a>

**在 M0-M5 集群上**

对于 M0（免费层）和 M2、M5（共享层）集群，与帐户关联的所有 MongoDB 用户都可以通过 [`db.killOp()`](https://docs.mongodb.com/manual/reference/method/db.killOp/index.html) shell 执行命令[`mongo`](https://docs.mongodb.com/manual/mongo/index.html) 以终止与同一集群关联的所有其他 MongoDB 用户的操作。有关详细信息，请参阅 MongoDB 文档的[具有特殊行为的命令部分。](https://docs.atlas.mongodb.com/reference/unsupported-commands/index.html#commands-with-special-behavior)

**在 M10+ 集群上，副本集**

对于 M10+ 集群，具有 [Organization Owner](https://docs.atlas.mongodb.com/reference/user-roles/#Organization-Owner)、 [Project Owner](https://docs.atlas.mongodb.com/reference/user-roles/#Project-Owner) 或 [Project Data Access Admin角色的 Atlas 用户可以通过](https://docs.atlas.mongodb.com/reference/user-roles/#Project-Data-Access-Admin)[Atlas 中的实时性能面板 (RTPP) 选项卡](https://docs.atlas.mongodb.com/real-time-performance-panel/index.html) 终止长时间运行的操作 ，或者通过[shell](https://docs.mongodb.com/manual/mongo/index.html) 执行 [`db.killOp()`](https://docs.mongodb.com/manual/reference/method/db.killOp/index.html) 命令 以终止运行_的操作由该用户启动_。[`mongo`](https://docs.mongodb.com/manual/mongo/index.html)

**在 M10+ 集群上，分片集群**

请查看 [命令](https://docs.mongodb.com/manual/reference/method/db.killOp/index.html#sharded-cluster)[的 Sharded Cluster 部分，`db.killOp()`](https://docs.mongodb.com/manual/reference/method/db.killOp/index.html#sharded-cluster) 以获取有关终止分片集群上正在进行的操作的具体步骤的详细信息。

#### <mark style="color:red;">问题：我的实时迁移尝试在验证过程中返回错误“无法到达指定的源”</mark> <a href="#issue-my-live-migrate-attempt-returned-an-error-could-not-reach-specified-source-during-the-validati" id="issue-my-live-migrate-attempt-returned-an-error-could-not-reach-specified-source-during-the-validati"></a>

**解决**

根据提供的错误，请注意这与源集群相关。如果您在源上启用了 SSL，请提供适当的证书并在 Live Migrate UI 中切换“SSL”选项。对于 Atlas 到 Atlas 的迁移，您只需将 SSL 切换设置为启用即可继续。

可以在以下文档中找到其他信息：

* [副本集迁移验证](https://docs.atlas.mongodb.com/import/live-import/#pre-migration-validation)
* [分片迁移验证](https://docs.atlas.mongodb.com/import/live-import-sharded/#pre-migration-validation)
* [Atlas 的实时迁移文档](https://docs.atlas.mongodb.com/import/live-import/index.html)
* [实时迁移验证文档故障排除](https://docs.atlas.mongodb.com/import/live-import-troubleshooting/#common-live-migration-validation-errors)

#### <mark style="color:red;">问题：我的实时迁移尝试在验证过程中返回错误“提供的用户名或密码不正确”</mark> <a href="#issue-my-live-migrate-attempt-returned-an-error-username-or-password-provided-is-not-correct-during" id="issue-my-live-migrate-attempt-returned-an-error-username-or-password-provided-is-not-correct-during"></a>

**解决**

根据提供的错误，请务必注意 Live Migrate 利用`admin`数据库进行身份验证和授权。`admin`因此，请确保在数据库中为源定义了具有适当权限和已知密码的用户。

可以在以下文档中找到其他信息：

* [副本集迁移验证](https://docs.atlas.mongodb.com/import/live-import/#pre-migration-validation)
* [分片迁移验证](https://docs.atlas.mongodb.com/import/live-import-sharded/#pre-migration-validation)
* [Atlas 的实时迁移文档](https://docs.atlas.mongodb.com/import/live-import/index.html)
* [实时迁移验证文档故障排除](https://docs.atlas.mongodb.com/import/live-import-troubleshooting/#common-live-migration-validation-errors)

#### <mark style="color:red;">问题：我的实时迁移尝试在验证过程中返回错误“其他”</mark> <a href="#issue-my-live-migrate-attempt-returned-an-error-other-during-the-validation-process" id="issue-my-live-migrate-attempt-returned-an-error-other-during-the-validation-process"></a>

**解决**

请向客户提供以下文档，并指示他们在迁移过程中遇到更多困难时联系 MongoDB 支持人员。

* [副本集迁移验证](https://docs.atlas.mongodb.com/import/live-import/#pre-migration-validation)
* [分片迁移验证](https://docs.atlas.mongodb.com/import/live-import-sharded/#pre-migration-validation)
* [Atlas 的实时迁移文档](https://docs.atlas.mongodb.com/import/live-import/index.html)
* [实时迁移验证文档故障排除](https://docs.atlas.mongodb.com/import/live-import-troubleshooting/#common-live-migration-validation-errors)

#### <mark style="color:red;">问题：客户无法创建 Atlas 搜索索引或仅收到部分结果</mark> <a href="#issue-customer-is-unable-to-create-an-atlas-search-index-or-receiving-only-partial-results-back" id="issue-customer-is-unable-to-create-an-atlas-search-index-or-receiving-only-partial-results-back"></a>

1. 确认他们尝试使用的集群
   1. Atlas Search 在 M10 和 M20 集群上不可用。
2. 确认他们使用的集群版本
   1. 仅支持 MongoDB 4.2 或更高版本
3. 确认是否创建了Atlas Search索引
   1. 索引创建是一个资源密集型过程，集群的性能会在索引构建时受到影响。
      1. 如果集群的资源紧张或接近可接受的性能限制，请考虑在实施 Atlas 搜索功能之前升级到更大的集群层。
   2. 尝试对 Atlas 搜索索引运行查询将生成不完整的结果
4. 对于集群层 M0、M2、M5，请注意 [以下限制](https://docs.atlas.mongodb.com/reference/atlas-search/limitations/)
   1. [索引定义 JSON 对象的](https://docs.atlas.mongodb.com/reference/atlas-search/index-definitions/) 大小不能超过 3kb
   2. 您不能将 [分析器定义](https://docs.atlas.mongodb.com/reference/atlas-search/analyzers/)添加 到集群上的索引中，该按钮将在禁用时可见。
   3. 您不能创建超过
      1. M0 集群上的 3 个索引
      2. M2 集群上的 5 个索引
      3. M5 集群上的 10 个索引
5. 如果已考虑这些缓解因素并且客户仍然遇到 Atlas Search 问题升级到 MongoDB 支持，但建议客户准备以下信息：
   1. 在集合上创建的[全文搜索索引](https://docs.atlas.mongodb.com/reference/full-text-search/edit-index/)的名称和完整结构
   2. 创建索引时显示的错误消息的完整文本
   3. 正在运行的查询使用全文搜索索引，以及输出
   4. 如果使用，任何 明确定义的[分析器](https://docs.atlas.mongodb.com/reference/full-text-search/analyzers/)
   5. 相关集合中的一个或多个示例对象（已编辑敏感字段值）
   6. 最后一次出现问题的日期和时间（包括时区）
   7. 任何可能有助于 MongoDB 支持人员更好地了解环境的屏幕截图或其他信息。

#### <mark style="color:red;">问题：无法通过对等连接与 Google Kubernetes Engine 连接</mark> <a href="#issue-cannot-connect-with-google-kubernetes-engine-over-peering" id="issue-cannot-connect-with-google-kubernetes-engine-over-peering"></a>

1. 验证常规对等互连是否适用于他们的云部署。
   1. 如果不是，请参阅下面的[对等故障排除](https://g3doc.corp.google.com/company/gfw/support/cloud/products/mongo-db-atlas/index.md?cl=head#peering-troubleshooting) 部分。
2. 如果是，继续解决。

**解决**

让客户将 GKE Pod 地址范围列入白名单，如此屏幕截图所示：

![GKE Pod 白名单](https://g3doc.corp.google.com/company/gfw/support/cloud/products/mongo-db-atlas/gke-pod-whitelist.png)

#### <mark style="color:red;">问题：无法通过对等网络连接</mark> <a href="#peering-troubleshooting" id="peering-troubleshooting"></a>

**解决**

1. 确认客户可以在 VPC 网络外部进行连接。
   1. 如果他们不能，则可能会发生本地连接问题。
   2. 如果可以，请继续进行故障排除。
2. 让客户运行以下命令（从 VPC 内的主机）：
   1. 远程登录主机名：27017
      1. 如果 telnet 成功，并返回私有 IP，则它们能够通过对等成功连接。
      2. 如果 telnet 超时，但确实返回私有 IP，请在步骤 3 中索取对等屏幕截图。
      3. 如果 telnet 返回公共 IP，无论成功与否，请检查步骤 3 中的对等配置。
3. 让客户分享以下内容的屏幕截图：
   1. 专有网络
      1. 确保客户已正确链接网络（GCP -> Atlas）。
      2. 如果它们没有链接，请让客户重新创建配置。
      3. 如果它们已链接，请继续进行故障排除。
   2. 安全部分 -> 网络访问
      1. 入站/出站流量的防火墙规则（仅当它们未使用默认设置时）
         1. 验证它们没有阻止 mongod 27017 入站。
   3. VPC 网络对等
      1. 有关连接的详细信息，包括 CIDR。
      2. 确保对等互连已连接到正确的 VPC 网络。
      3. 确保 Atlas UI 中的对等 VPC 网络和对等项目 ID 是正确的。
   4. 项目编号
      1. 谷歌项目 ID (my-gcp-12345)
   5. Atlas UI -> 网络 -> 安全
      1. 用于集群访问的 Atlas 白名单。
         1. 验证他们是否已将正确的 GCP CIDR 列入白名单。
         2. 如果列入白名单，请检查对等配置。
         3. 如果不是，请将它们列入白名单并通过步骤 2 进行测试。

#### <mark style="color:red;">问题：关于数据传输成本的问题</mark> <a href="#issue-questions-regarding-data-transfer-costs" id="issue-questions-regarding-data-transfer-costs"></a>

请客户参阅 [计费文档的数据传输部分](https://docs.atlas.mongodb.com/billing/#data-transfer)。

[有关其他信息，请参阅有关计费的](https://docs.atlas.mongodb.com/billing/#billing)MongoDB Atlas 文档 。他们还可以使用[Atlas 定价工具](https://www.mongodb.com/cloud/atlas/pricing)来估算他们希望用于其用例的特定 Atlas 部署的成本。

#### <mark style="color:red;">问题：集群反复触发初始同步，尤其是在大量写入活动或客户收到复制 Oplog 警报之后</mark> <a href="#issue-cluster-repeatedly-triggers-an-initial-sync-particularly-after-periods-of-heavy-write-activity" id="issue-cluster-repeatedly-triggers-an-initial-sync-particularly-after-periods-of-heavy-write-activity"></a>

`PRIMARY`如果在副本集的节点上进行大量写入操作，并且`SECONDARY`节点没有足够的时间重放包含在 中的所有操作 [`oplog`](https://docs.mongodb.com/manual/core/replica-set-oplog/index.html)，这通常会导致“掉落`oplog`”，这需要 [初始同步](https://docs.mongodb.com/manual/core/replica-set-sync/index.html#initial-sync) 为了恢复并确保数据在所有节点上是一致的。

[Alert Resolutions > Replication Oplog Alerts 文档](https://docs.atlas.mongodb.com/reference/alert-resolutions/replication-oplog/)中概述了更多信息和可能的缓解步骤 。

#### <mark style="color:red;">问题：客户想要查看其集群的 MongoDB 日志</mark> <a href="#issue-customer-would-like-to-review-their-clusters-mongodb-logs" id="issue-customer-would-like-to-review-their-clusters-mongodb-logs"></a>

如果客户希望查看集群的 MongoDB 日志，他们可以使用以下步骤进行：

1. 导航到集群页面
2. 在相关集群部分中，单击省略号 ( `...`) 按钮，然后选择下载日志选项。
3. 在下载日志模式中，根据需要更新选择服务器、开始时间和结束时间字段。
4. 单击下载日志按钮。

#### <mark style="color:red;">问题：客户寻求有关查询定位警报的指导或帮助审查和调整他们的工作负载</mark> <a href="#issue-customer-seeking-guidance-on-a-query-targeting-alert-or-help-with-reviewing-and-tuning-their-w" id="issue-customer-seeking-guidance-on-a-query-targeting-alert-or-help-with-reviewing-and-tuning-their-w"></a>

要了解如何生成查询定位警报，我们建议查看 [警报解决方案 > 查询定位文档](https://docs.atlas.mongodb.com/reference/alert-resolutions/query-targeting/index.html)。

为了进一步研究如何优化查询，这里有一些额外的资源：

* 警报 [解决方案 - 查询定位](https://docs.atlas.mongodb.com/reference/alert-resolutions/query-targeting/index.html) 文档描述了一些缓解此问题的短期和长期解决方案。
* Atlas [Query Profiler](https://docs.atlas.mongodb.com/tutorial/profile-database/index.html) 提供长达 24 小时的历史查询示例，而不会为 M10+ 集群带来额外的成本或性能开销。
* [实时性能面板](https://docs.atlas.mongodb.com/real-time-performance-panel/index.html#real-time-performance-panel)可以 在执行时实时显示性能不佳的查询。
* Database [Profiler](https://docs.mongodb.com/manual/tutorial/manage-the-database-profiler/#database-profiler) 可用于收集有关在部署中运行的操作的细粒度数据。
* MongoDB [大学 M201 课程](https://university.mongodb.com/courses/M201/about) 深入介绍了 MongoDB 性能。

如果客户要求对所确定的查询和索引的调优和性能提供更具体的指导，请让他们直接联系 MongoDB 支持。

#### <mark style="color:red;">问题：客户请求帮助修改他们的备份配置</mark> <a href="#issue-customer-requests-help-modifying-their-backup-configuration" id="issue-customer-requests-help-modifying-their-backup-configuration"></a>

重要提示：请向客户指出，禁用备份会导致 Atlas 立即删除集群的所有备份快照。 有关详细信息，请参阅 [完全托管备份服务文档。](https://docs.atlas.mongodb.com/backup-cluster/)

请客户参阅“ [为集群启用或禁用备份”部分](https://docs.atlas.mongodb.com/scale-cluster/#enable-or-disable-backup-for-the-cluster)，该部分介绍了调整备份设置，是“ [修改集群”文档](https://docs.atlas.mongodb.com/scale-cluster/#walkthrough)的一部分 ，作为更新集群配置以满足客户要求的参考。此外，客户可以使用 [Atlas API](https://docs.atlas.mongodb.com/reference/api/clusters-modify-one/)以编程方式修改您的集群。

\
