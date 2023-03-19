# 如何迁移到MongoDB Atlas

MongoDB Atlas提供了Live Migrate的功能实现向MongoDB Atlas的迁移。同时，也可以使用一些工具软件完成。详细信息请参考这个[文档](https://www.mongodb.com/docs/atlas/import/)。

本教程介绍了如何使用 MongoDB 的 [Atlass Live Migration Service (Atlass Live Migration)](https://docs.atlas.mongodb.com/import/live-import/)实现从包含数据库的自行管理的 [MongoDB 副本集](https://docs.mongodb.com/manual/replication/)到 [MongoDB Atlas](https://www.mongodb.com/cloud/atlas) 中全托管式数据库的迁移。

本教程适用于对完全托管的 MongoDB 服务感兴趣或负责将 MongoDB 副本集中的 MongoDB 数据库迁移到 MongoDB Atlas 集群的数据库架构师、数据库管理员和数据库工程师。

### 目标 <a href="#objectives" id="objectives"></a>

* 通过创建文档并将其加载到示例 MongoDB 副本集中，设置自行管理的来源。
* 在 MongoDB Atlas 中设置迁移目标集群。
* 使用 Atlas Live Migration 将数据库从自行管理的 MongoDB 副本集迁移到全托管式 MongoDB Atlas 集群。
* 了解并选择测试、切换和后备策略。

本教程使用一个 MongoDB 副本集作为来源。本教程未介绍如何将[分片 MongoDB 集群](https://docs.atlas.mongodb.com/import/live-import-sharded/)迁移到 MongoDB Atlas。请参阅 [Stack Exchange：数据库管理员](https://dba.stackexchange.com/questions/52632/difference-between-sharding-and-replication-on-mongodb)博文，了解 MongoDB 副本集和分片 MongoDB 集群之间的架构差异。

要完成本教程的迁移，请使用 [Atlas Live Migration](https://docs.atlas.mongodb.com/import/live-import/)，而不是 [Atlass `mongomirror`](https://docs.atlas.mongodb.com/reference/mongomirror/) 实用程序。`mongomirror` 实用程序需要在来源 MongoDB 环境中安装代理，并在较低抽象级别上操作。

本教程中的迁移设置是采用复制语义的[同构迁移](https://cloud.google.com/solutions/database-migration-concepts-principles-part-1?hl=zh-cn#heterogeneous\_migration\_versus\_homogeneous\_migration)。在迁移期间数据不会进行转换，也不会发生数据库合并或数据重新分片。您可以使用 [Striim](https://www.striim.com/) 等集成技术实现除复制语义之外的功能。

本教程中的迁移是从源 MongoDB 副本集到目标 MongoDB Atlas 集群的单向迁移。从源 MongoDB 副本集到目标集群的转换完成后，源数据库不会与目标集群的更改保持同步。因此，如果在生产环境中实现此解决方案，则无法将应用切换到后备中的最新源数据库。如需详细了解后备流程，请参阅[数据库迁移：概念和原则（第 2 部分）](https://cloud.google.com/solutions/database-migration-concepts-principles-part-2?hl=zh-cn)。

### 费用 <a href="#costs" id="costs"></a>

本教程使用 Google Cloud 的以下收费组件：

* [Compute Engine](https://cloud.google.com/compute/all-pricing?hl=zh-cn)

要完成本教程，您不能使用 MongoDB Atlas 免费层级。免费层级中的可用机器类型不支持 Atlas 实时迁移。在 MongoDB Atlas 中，所需的最低机器类型（在撰写本文时为 M10）具有每小时服务成本。如要生成价格估算，请转到 [MongoDB Atlas 价格](https://www.mongodb.com/cloud/atlas/pricing)，点击 **Google Cloud Platform**，然后按照说明操作。如要在生产环境中实现此迁移，我们建议使用[常规托管的 MongoDB Atlas 版本](https://console.cloud.google.com/marketplace/details/mongodb/atlas-pro?hl=zh-cn)。

完成本教程后，您可以删除所创建的资源以避免继续计费。如需了解详情，请参阅[清理](https://cloud.google.com/architecture/mongodb-atlas-live-migration?hl=zh-cn#clean-up)。

### 准备工作 <a href="#before-you-begin" id="before-you-begin"></a>

1.  在 Google Cloud Console 中的项目选择器页面上，选择或[创建一个 Google Cloud 项目](https://cloud.google.com/resource-manager/docs/creating-managing-projects?hl=zh-cn)。

    注意：如果您不打算保留在此过程中创建的资源，请创建新的项目，而不要选择现有的项目。完成本教程介绍的步骤后，您可以删除所创建的项目，并移除与该项目关联的所有资源。

    [转到“项目选择器”](https://console.cloud.google.com/projectselector2/home/dashboard?hl=zh-cn)
2. 确保您的 Cloud 项目已启用结算功能。了解如何[检查项目是否已启用结算功能](https://cloud.google.com/billing/docs/how-to/verify-billing-enabled?hl=zh-cn)。

### 迁移架构 <a href="#migration_architecture" id="migration_architecture"></a>

下图显示了您在本教程中创建的部署架构。

![Compute Engine 上的 MongoDB 服务器，具有从主数据库到 MongoDB Atlas 的迁移路径。](https://cloud.google.com/static/architecture/images/mongodb-atlas-live-migration-architecture.png?hl=zh-cn)

箭头表示从 Compute Engine 上运行的源 MongoDB 副本集到 Google Cloud 上 MongoDB Atlas 上运行的目标群集的数据迁移路径。

部署架构包含以下组件：

* **源数据库**：在三个 Compute Engine 实例上运行的自行管理的 MongoDB 副本集
* **目标数据库**：全托管式 MongoDB Atlas 集群
* **迁移服务**：用于将数据从来源迁移到目标的 Atlas Live Migration 配置

虽然本教程使用 Compute Engine 实例上的自行管理的 MongoDB 副本集，但您也可以在本地数据中心或其他云环境中部署源 MongoDB 副本集。

Atlas Live Migration 支持[零停机时间数据库迁移方法](https://cloud.google.com/solutions/database-migration-concepts-principles-part-1?hl=zh-cn#migration\_downtime\_zero\_versus\_minimal\_versus\_significant)。在从源 MongoDB 副本集进行迁移期间，您的应用仍可以访问源数据库，不会受到影响。在[初始加载](https://cloud.google.com/solutions/database-migration-concepts-principles-part-2?hl=zh-cn#phase\_1\_initial\_load)完成后，Atlass Live Migration 会迁移在迁移开始后发生的更改。

如需在初始数据集迁移后执行从源数据库到目标集群的切换，请按以下步骤操作：

1. 暂停对源数据库的写入访问。
2. 等待 Atlas Live Migration 捕获其余更改并将其应用于目标数据库。
3. 在 Atlas Live Migration 中执行切换。
4. 停止源数据库。

迁移完所有数据后，Atlas Live Migration 会在用户界面的进度状态行上通知您。至此，数据迁移完成，应用系统可以开始访问目标集群作为新的记录系统。

### 创建自行管理的 MongoDB 副本集 <a href="#creating_a_self-managed_mongodb_replica_set" id="creating_a_self-managed_mongodb_replica_set"></a>

首先，将在 Google Cloud 上安装 MongoDB 副本集。此数据库充当源数据库。接下来，请检查源数据库是否符合所有必需的前提条件。

**注意**：本教程包括检查前提条件的步骤的原因是为您在生产环境中做好准备。即使生产环境中已经存在 MongoDB 副本，您仍然需要检查前提条件。

完成前提条件检查后，您必须启用身份验证并重启源 MongoDB 实例。最后，要测试迁移，您需要将示例数据集添加到迁移到目标数据库的源 MongoDB 实例中。

#### 安装 MongoDB 副本集 <a href="#install_the_mongodb_replica_set" id="install_the_mongodb_replica_set"></a>

1.  在 Google Cloud Marketplace 中，转到 Compute Engine 上的 MongoDB 副本集安装。在撰写本文时，当前版本为 MongoDB 4.0。

    [转到 Cloud Marketplace 中的 MongoDB](https://console.cloud.google.com/marketplace/details/click-to-deploy-images/mongodb?hl=zh-cn)
2.  点击**启动**。 由于启用了多个 Google Cloud API，启动过程可能需要一段时间。

    如果您拥有多个项目的权限，则显示项目列表。选择要安装 MongoDB 的项目。

    根据 Deployment Manager 模板在一组 Compute Engine 实例上部署 MongoDB 副本集。
3. 接受所有默认配置设置。
4. 点击**部署**。
5.  在 Google Cloud 控制台中，激活 Cloud Shell。

    [激活 Cloud Shell](https://console.cloud.google.com/?cloudshell=true\&hl=zh-cn)

    [Cloud Shell](https://cloud.google.com/shell/docs/how-cloud-shell-works?hl=zh-cn) 会话随即会在 Google Cloud 控制台的底部启动，并显示命令行提示符。Cloud Shell 是一个已安装 Google Cloud CLI 且已为当前项目设置值的 Shell 环境。该会话可能需要几秒钟时间来完成初始化。
6.  使用 `ssh` 登录运行 MongoDB 主实例的 Compute Engine 实例：

    ```
    gcloud compute ssh MONGODB_VM_NAME --project PROJECT_ID --zone ZONE_OF_VM
    ```

    替换以下内容：

    * `MONGODB_VM_NAME`：MongoDB 副本集的主副本的名称
    * `PROJECT_ID`：Cloud 项目 ID 的名称
    *   `ZONE_OF_VM`：虚拟机 (VM) 实例所在的区域

        如需了解详情，请参阅[地理位置和地区](https://cloud.google.com/docs/geography-and-regions?hl=zh-cn)。

    如果生成了 SSH 密钥，则系统会要求提供密码。如果不想提供密码，请按 `Enter`。如果确实提供了密码，请注明以备将来参考。

    如果无法使用 Cloud Shell 进行连接，请在 Deployment Manager 中点击**服务器层级虚拟机的 SSH**。
7.  启动 [`mongo` shell](https://www.mongodb.com/docs/v6.0/)：

    ```
    mongo
    ```
8.  列出现有数据库：

    ```
    show dbs
    ```

    输出内容类似如下：

    ```
    admin   0.000GB
    config  0.000GB
    local   0.000GB
    ```

    对于即将启动的命令，请保持 `mongo` shell 的打开状态。

您已创建并访问 MongoDB 副本集并确认其可正常运行。

#### 检查源数据库的前提条件 <a href="#check_preconditions_for_source_database" id="check_preconditions_for_source_database"></a>

Atlas Live Migration 要求源 MongoDB 副本集符合特定的配置标准或前提条件。[Atlas 迁移概览](https://www.mongodb.com/cloud/atlas/migrate/migrate-from-aws)和[Atlas 文档](https://docs.atlas.mongodb.com/import/live-import/#prerequisites)概述了相关检查。以下命令源自这些资源。验证这些前提条件并进行所有必要的更改后，源数据库需要[进一步配置和重启](https://cloud.google.com/architecture/mongodb-atlas-live-migration?hl=zh-cn#enable-auth)。

**注意**：虽然在本教程中安装的源 MongoDB 副本符合所需的版本，但仍需在生产环境中检查前提条件。因此，我们这里包含一个演示该检查的过程。

如要检查是否满足所有前提条件，请执行以下操作：

1.  在 `mongo` shell 中，检查 MongoDB 的版本是否为 2.6 或更高版本。（在生产数据库实例中，打开 `mongo` shell，使用 SSH 连接连接到 MongoDB 服务器，然后运行以下命令来确定版本。）

    ```
    db.version()
    ```

    输出会显示版本。如果您的版本低于 2.6，您需要按照[升级说明](https://docs.mongodb.com/v2.6/release-notes/2.6-upgrade/)进行操作。
2.  检查当前部署是否为 MongoDB 副本集：

    ```
    rs.status()
    ```

    输出是 MongoDB 副本集的状态。以下输出显示在未启用 MongoDB 副本集的情况下启动的 MongoDB 实例。

    ```
    {
        "ok" : 0,
        "errmsg" : "not running with --replSet",
        "code" : 76,
        "codeName" : "NoReplicationEnabled"
    }
    ```

    在这种情况下，请停止并重启启用了 MongoDB 副本集的 MongoDB 实例。如果您有独立的 MongoDB 实例，请[将 MongoDB](https://docs.mongodb.com/manual/tutorial/convert-standalone-to-replica-set/) 实例升级为 MongoDB 副本集。
3.  通过登录检查源集群上是否启用了身份验证：

    ```
    mongo -u YOUR_ADMIN_USERNAME -p --authenticationDatabase admin
    ```

    替换以下内容：

    * `YOUR_ADMIN_USERNAME`：部署的管理员用户名

    之前创建的 MongoDB 副本集未启用身份验证。

    如果未启用身份验证，您需要按照[启用身份验证的说明](https://docs.mongodb.com/v4.0/tutorial/enable-authentication/)进行操作。以下是用于启用身份验证的示例命令，包含示例用户名和密码：

    ```
    use admin
    db.createUser(
      {
        user: "myUserAdmin",
        pwd: "myUserAdminPassword",
        roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase", "clusterMonitor" ]
      }
    )
    ```

    启用身份验证后，必须具备 MongoDB 角色 `clusterMonitor` 才能执行 `rs.status()`。上述命令指定了此角色。
4.  检查管理员用户是否已为 MongoDB 副本集的版本分配正确的角色。如需查看与特定版本对应的角色列表，请参阅 Atlas Live Migration 文档中有关[源集群安全性](https://docs.atlas.mongodb.com/import/live-import/#source-cluster-security)的讨论。

    ```
    use admin
    db.getUser("YOUR_ADMIN_USERNAME")
    ```

    用户名必须放在引号之间。
5. （可选）如果 MongoDB 部署基于早于 4.2 的版本，则其包含密钥超过 [1024 字节索引密钥限制](https://docs.mongodb.com/v4.0/reference/limits/#Index-Key-Limit)的索引。在这种情况下，请在启动 Atlas Live Migration 过程之前将 MongoDB 服务器参数 [`failIndexKeyTooLong`](https://docs.mongodb.com/v4.0/reference/parameters/#param.failIndexKeyTooLong) 设置为 false。

#### 启用身份验证并重启 MongoDB 副本集 <a href="#enable-auth" id="enable-auth"></a>

如需开启身份验证，除了创建管理员之外，还需要密钥文件。以下步骤显示了如何手动创建密钥文件。在生产环境中，您可以考虑使用脚本来自动化该过程。

1.  在 Cloud Shell 中，创建一个密钥文件：

    ```
    openssl rand -base64 756 > PATH_TO_KEY_FILE
    ```

    替换以下内容：

    * `PATH_TO_KEY_FILE`：SSH 密钥的存储位置，例如 `/etc/mongo-key`
2. 为三个虚拟机分别启用授权：
   1.  将密钥文件复制到虚拟机：

       ```
       gcloud compute copy-files PATH_TO_KEY_FILE NAME_OF_THE_VM:PATH_TO_KEY_FILE --zone=ZONE_OF_VM
       ```

       替换以下内容：

       * `NAME_OF_THE_VM`：运行副本集副本的某个虚拟机的名称
       * `ZONE_OF_VM`：`NAME_OF_THE_VM` 中引用的虚拟机所在的 Google Cloud 区域
   2.  使用 `ssh` 登录虚拟机并更改所有者以及密钥文件的访问权限：

       ```
       sudo chown mongodb:mongodb PATH_TO_KEY_FILE

       sudo chmod 400 PATH_TO_KEY_FILE
       ```
   3. 在首选文本编辑器中，在修改模式下打开 `mongod.conf` 文件。如果要回写任意更改，您可能需要使用 `sudo` 命令来启动文本编辑器。
   4.  修改 `security` 部分，如下所示：

       ```
       security:
         authorization: enabled
         keyFile: PATH_TO_KEY_FILE
       ```
   5.  重启副本：

       ```
       sudo service mongod restart
       ```
3.  验证是否可以登录到 MongoDB 副本集的主数据库：

    ```
    mongo -u YOUR_ADMIN_USERNAME -p --authenticationDatabase admin
    ```

#### 插入示例数据 <a href="#insert_sample_data" id="insert_sample_data"></a>

在以下步骤中，将示例数据插入源数据库，然后验证文档是否已成功插入：

1.  在 Cloud Shell 中，使用 `ssh` 连接到 MongoDB 主 Compute Engine 实例：

    ```
    gcloud compute ssh MONGODB_VM_NAME --project PROJECT_ID --zone ZONE_OF_VM
    ```

    您可能需要提供 SSH 密钥的密码。
2.  启动 `mongo` shell：

    ```
    mongo -u YOUR_ADMIN_USERNAME -p --authenticationDatabase admin
    ```

    提供在创建管理员用户名时指定的密码。
3.  创建一个数据库：

    ```
    use migration
    ```
4.  创建集合

    ```
    db.createCollection("source")
    ```
5.  验证集合是否为空：

    ```
    db.source.count()
    ```
6.  添加以下五个文档作为初始数据集：

    ```
    db.source.insert({"document_number": 1})
    db.source.insert({"document_number": 2})
    db.source.insert({"document_number": 3})
    db.source.insert({"document_number": 4})
    db.source.insert({"document_number": 5})
    ```

    这些命令的输出与以下内容类似：

    ```
    WriteResult({ "nInserted" : 1 })
    ```
7.  验证是否已将五个文档成功添加到集合迁移中。结果必须是 5。

    ```
    db.source.count()
    ```

设置并启动数据库迁移后，这些文档将迁移到 MongoDB Atlas 中的目标集群。

### 在 MongoDB Atlas 中创建集群 <a href="#creating_a_cluster_in_mongodb_atlas" id="creating_a_cluster_in_mongodb_atlas"></a>

MongoDB 副本集在 MongoDB Atlas 中称为集群。如果您尚未将集群设置为目标数据库，请按照本部分中的步骤操作。这些步骤基于 [MongoDB 文档](https://docs.atlas.mongodb.com/tutorial/create-new-cluster/)。如果您已将集群设置为目标数据库，则可以跳过本部分。

**注意**：本教程使用每小时收费的 MongoDB Atlas 版本。如要在生产环境中实现此迁移，我们建议使用[常规托管的 MongoDB Atlas 版本](https://console.cloud.google.com/marketplace/details/mongodb/atlas-pro?hl=zh-cn)。如需了解详情，请参阅[费用](https://cloud.google.com/architecture/mongodb-atlas-live-migration?hl=zh-cn#costs)。

1.  在 Cloud Marketplace 中，转到 **MongoDB Atlas - 免费层级安装**页面。

    [转到 Marketplace 上的 MongoDB Atlas](https://console.cloud.google.com/marketplace/details/gc-launcher-for-mongodb-atlas/mongodb-atlas?hl=zh-cn)
2. 点击**访问 MongoDB 网站进行注册**。
3. 点击**启动第一个集群**。
4. 填写所需信息，然后点击**免费开始使用**。备注提供的信息。
5. 点击**高级配置选项**。
6. 对于 **Cloud Provider & Region**，选择 **Google Cloud Platform** 和 **Iowa (us-central1)**。
7. 点击**集群层级**标签页，然后选择 **M10**。
8. 点击**其他设置**标签页，选择 **MongoDB 4.0 或 MongoDB 4.2**，然后关闭备份。
9.  点击**创建集群**。

    等待集群创建完成。请注意，项目名称为 `Project 0`（含空格），集群名称为 `Cluster0`（不含空格）。

    **注意**：这些流程的最新版本屏幕截图包含在 [MongoDB Atlas 文档](https://docs.atlas.mongodb.com/tutorial/create-new-cluster/)中。

目标集群在 MongoDB Atlas 中进行设置和运行。

### 测试 MongoDB Atlas 集群的故障切换 <a href="#testing_the_failover_of_the_mongodb_atlas_cluster" id="testing_the_failover_of_the_mongodb_atlas_cluster"></a>

迁移完成后，MongoDB Atlas 中的集群将执行滚动重启。每个集群成员依次重启。为了确保此流程有效，请[按照 MongoDB 文档中的步骤](https://docs.atlas.mongodb.com/tutorial/test-failover/)测试故障切换。

### 启动实时迁移 <a href="#starting_the_live_migration" id="starting_the_live_migration"></a>

如要将数据从源数据库迁移到目标数据库，请执行以下操作：

1. 登录到 [MongoDB Atlas](https://account.mongodb.com/account/login)。
2. 转到**集群**页面，然后选择要迁移到的集群。
3. 在目标集群 (`Cluster 0`) 窗格中，点击省略号按钮 more\_horiz。
4. 选择**将数据迁移到此集群**。
5.  在打开的窗口中，查看信息。准备好迁移后，点击**我已准备好迁移**。

    系统会显示包含数据迁移说明的窗口。列出的 IP 地址必须能够访问 MongoDB 副本集。如果您尚未为这些地址创建防火墙规则，请使用 Cloud Shell 根据以下示例命令添加防火墙规则：

    ```
    gcloud compute firewall-rules create "allow-mongodb-atlas" --allow=tcp:27027 --source-ranges="35.170.231.208/32,3.92.230.111/32,3.94.238.78/32,54.84.208.96/32" --direction=INGRESS
    ```
6. 在**主机名：副本集的主服务器端口**字段中，输入 MongoDB 副本集的主服务器的 IP 地址和端口，例如，`IP_ADDRESS`：`PORT_FOR_PRIMARY`。
   1.  如要确定主实例，请在 Cloud 项目中运行的任一实例的 `mongo` shell 中运行以下命令：

       `rs.isMaster().primary`
   2. 如要查找相应的外部 IP 地址，请转到 Compute Engine [**虚拟机实例**页面](https://console.cloud.google.com/compute/instances?hl=zh-cn)。标准 MongoDB 端口为 `27017`。
7.  输入 MongoDB 副本集的管理员用户名和密码。

    其他所有设置保留默认值。
8.  点击**验证**，然后执行以下某项操作：

    * 如果验证成功，请点击**开始迁移**。
    * 如果验证失败，请按照提供的说明进行问题排查。例如，如果 MongoDB Atlas 无法连接到 MongoDB 副本集，则会提供 MongoDB Atlas 尝试连接的 IP 地址。为这些地址添加一条防火墙规则，该规则允许 MongoDB 副本集服务器的端口 27017 上的 TCP 流量。

    MongoDB Atlas 屏幕显示取得的进展。等待进度条中的消息**初始同步完成！**。

MongoDB 副本集中的初始加载现已完成。下一步是验证初始加载是否成功。

初始迁移完成后，MongoDB Atlas 会估算必须切转到目标集群之前剩余的小时数。您可能还会收到 MongoDB 发送的电子邮件，让您了解剩余小时数、扩展时间的能力以及如未在给定时间内完成最终切换，迁移将被取消的警告。

### 验证数据库迁移 <a href="#verifying_the_database_migration" id="verifying_the_database_migration"></a>

设计和实施数据库迁移验证策略以确认数据库迁移成功非常重要。虽然特定的验证策略取决于您的特定用例，但我们建议您执行以下检查：

* **完整性检查**。验证初始文档集是否从源数据库成功迁移（初始加载）。
* **动态检查**。验证源数据库中的更改是否正被转移到目标数据库（正在进行迁移）。

首先，验证初始加载是否成功：

1. 在 MongoDB Atlas 中，点击**集群**。
2. 点击**集合**。
3. 验证名为 `migrations` 的数据库是否存在，以及名为 `source` 的集合是否有 5 个文档。

接下来，验证对源数据库正在进行的更改是否反映在目标数据库中：

1. 在 Cloud Shell 中，使用 `ssh` 登录源 MongoDB 副本集的主虚拟机。
2. 启动 `mongo` shell。 `mongo`
3.  插入其他文档：

    ```
    use migration
    db.source.insert({"document_number": 6})
    ```
4. 在迁移集合的 **MongoDB Atlas 集合**页面中，点击**刷新**观察将一个文档添加到集合 `source`。

您已经验证了所有原始数据的来源以及对源的任何正在进行的更改都将通过 Atlas Live Migration 自动迁移到目标。

### 测试 Atlas 目标集群 <a href="#testing_your_atlas_target_cluster" id="testing_your_atlas_target_cluster"></a>

在生产环境中，测试至关重要。对访问目标数据库的应用进行测试，以确保其正常运行是必要的。本部分讨论多种测试策略。

#### 在迁移过程中，使用目标数据库测试应用 <a href="#test_applications_with_a_target_database_during_a_migration" id="test_applications_with_a_target_database_during_a_migration"></a>

如上一部分所示，您可以在正在进行的数据库迁移期间执行应用测试。如果应用程序不以与源数据库中正在迁移的数据相冲突的方式更改目标，则此方法可能会起作用。此方法是否适合您，取决于您的环境和依赖项。如果测试应用将数据写入目标数据库，则可能会与正在进行的迁移发生冲突。

#### 使用临时目标数据库测试应用 <a href="#test_applications_with_a_temporary_target_database" id="test_applications_with_a_temporary_target_database"></a>

如果在生产数据库迁移期间无法测试应用，则可以将数据迁移到仅用于测试的临时目标数据库，然后在测试迁移后删除测试目标。

对于此方法，您可以在某个时刻停止测试迁移（就如同数据库迁移完成），并针对这些测试数据库测试应用。测试完成后，删除目标数据库并启动生产数据库迁移，以便将数据迁移到永久性目标数据库。此策略的优点是可以读取和写入目标数据库，因为它们仅用于测试。

#### 迁移完成后，使用目标数据库测试应用 <a href="#test_applications_with_a_target_database_after_migration_is_complete" id="test_applications_with_a_target_database_after_migration_is_complete"></a>

如果前两个策略都不可行，则其余策略是在迁移完成后在数据库上测试应用。将所有数据存储在目标数据库之后，对应用进行测试，然后再将其提供给用户使用。如果测试包括写入数据，则必须写入测试数据而不是生产数据，以避免生产数据不一致，这一点很重要。完成测试后，必须移除测试数据，以避免目标数据库中出现数据不一致或多余的数据。

我们建议先备份目标数据库，然后再将其打开以供应用系统进行生产访问。此步骤有助于确保在需要时可以重新创建一致的起点。

### 从源 MongoDB 副本集切换到目标集群 <a href="#cutting_over_from_the_source_mongodb_replica_set_to_the_target_cluster" id="cutting_over_from_the_source_mongodb_replica_set_to_the_target_cluster"></a>

完成所有测试并验证正在进行的更改已反映在目标数据库中后，您可以计划转换。

首先，您需要停止对源数据库的所有更改，以便 Atlas Live Migration 能够将未迁移的更改输出到目标数据库。在目标数据库中捕获所有更改后，您可以启动 Atlas Live Migration 转换过程。完成该过程后，您可以将客户端从源数据库转换到目标数据库。

1. 在 MongoDB Atlas 中，点击**集群**。
2. 在 **`Cluster0`** 窗格、点击**准备切换**。系统会显示了切换过程的步骤说明以及到目标集群的连接字符串。
3.  点击**切换**。

    迁移完成后，系统会显示消息**成功！集群迁移已完成。**

您已成功将 MongoDB 副本集迁移到 MongoDB Atlas 集群。

### 准备后备策略 <a href="#preparing_a_fallback_strategy" id="preparing_a_fallback_strategy"></a>

切换完成后，目标集群即为记录系统；源数据库已过期，最终将被移除。但是，当新目标数据库出现严重故障时，您可能希望回退到源数据库。例如，如果在测试期间未执行应用中的业务逻辑，之后又无法正常运行，则会发生失败。性能或延迟行为与源数据库不匹配并导致错误会导致另一个失败。

为了避免此类失败，您可能需要使原始源数据库与目标数据库更改保持同步。Atlas Live Migration 不提供后备机制。如需详细了解后备策略，请参阅[数据库迁移：概念和原则（第 2 部分）](https://cloud.google.com/solutions/database-migration-concepts-principles-part-2?hl=zh-cn)。

### 清除数据 <a href="#clean-up" id="clean-up"></a>

#### 删除 Cloud 项目 <a href="#delete-the-google-cloud-project" id="delete-the-google-cloud-project"></a>

为避免因本教程中使用的资源导致您的 Google Cloud 帐号产生费用，您可以删除为本教程创建的 Cloud 项目。

1. **警告**：删除项目会产生以下影响
2.
   * **项目中的所有内容都会被删除。**如果您将现有项目用于本教程，则在删除该项目后，还将删除您已在该项目中完成的任何其他工作。
   * **自定义项目 ID 丢失。**创建此项目时，您可能创建了要在将来使用的自定义项目 ID。要保留使用该项目 ID 的网址（例如 `appspot.com` 网址），请删除项目内的所选资源，而不是删除整个项目。
3. 如果您打算浏览多个教程和快速入门，重复使用项目可以帮助您避免超出项目配额上限。
4.  在 Google Cloud 控制台中，转到管理资源页面：

    [转到“管理资源”](https://console.cloud.google.com/iam-admin/projects?hl=zh-cn)
5. 在项目列表中，选择要删除的项目，然后点击删除。
6. 在对话框中输入项目 ID，然后点击关闭以删除项目。

#### 暂停或终止 MongoDB Atlas 集群 <a href="#pause_or_terminate_the_mongodb_atlas_cluster" id="pause_or_terminate_the_mongodb_atlas_cluster"></a>

为了避免对 MongoDB Atlas 集群收取更多费用，您需要暂停或终止集群。如需详细了解结算方面的影响，请参阅[暂停或终止集群](https://docs.atlas.mongodb.com/pause-terminate-cluster/)。

