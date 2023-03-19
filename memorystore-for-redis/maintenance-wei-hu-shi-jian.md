# Maintenance

### 维护事件通知

1. 选择接收维护通知：[开启项目维护](https://console.cloud.google.com/user-preferences/communication?hl=zh-cn)
2. 点击**产品通知**标签页。
3. 从下拉菜单中选择您的项目。
4. 在 Memorystore 对应的行中，将电子邮件按钮切换为**开启**。

维护通知电子邮件会使用标题 `"Upcoming maintenance for your Cloud Memorystore instance [your-instance-name]"`。需要接收通知的每个电子邮件地址都必须单独选择启用。

维护通知是在**项目级**（而不是在实例上）设置的。 电子邮件通知会发送到与您的 Google 帐号关联的电子邮件地址。您不能配置自定义电子邮件别名（例如团队电子邮件别名）。

### Maintenance维护窗口

订阅了维护通知并设置了[维护窗口](https://cloud.google.com/memorystore/docs/redis/find-and-set-maintenance-windows?hl=zh-cn)，则至少会在维护事件发生前七天收到电子邮件提醒。为实例安排维护之后，你可以立即开始更新实例，也可以将更新从最初安排的维护时间开始推迟最多七天。

### 查找维护事件

1. 转到 Google Cloud 控制台中的 **Memorystore for Redis** 页面。\
   [Memorystore for Redis](https://console.cloud.google.com/memorystore/redis/instances?hl=zh-cn)
2. 点击要查看其计划维护的实例的实例 ID。
3. 在**维护**部分下，您可以查看任何计划维护更新的日期和时间。

### 维护事件对实例的影响：

**标准层级的影响**

在维护期间，标准层级实例会进行故障切换。故障切换通常**持续几秒钟**。故障切换后，客户端应用需要重新连接。收到通知即将进行维护的电子邮件后，您可以对非生产实例运行[手动故障切换](https://cloud.google.com/memorystore/docs/redis/initiating-manual-failover?hl=zh-cn)，以测试维护对您的实例的影响。

**基本层级的影响**

基本层级实例在维护期间不可用，通常约为 5 分钟。
