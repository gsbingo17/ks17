# 通过复制支持高可用和故障转移

在 Redis 复制的基础上（不包括由 Redis Cluster 或 Redis Sentinel 作为附加层提供的高可用性功能）有一个易于使用和配置的_领导者跟随者（主副本）复制。_它允许副本 Redis 实例成为主实例的精确副本。_每次链接断开时，副本都会自动重新连接到主服务器，并且无论_主服务器发生什么情况，都会尝试成为它的精确副本。

该系统使用三种主要机制工作：

1. 当主实例和副本实例连接良好时，主实例通过向副本发送命令流来复制副本的更新，以复制由于以下原因而在主端发生的对数据集的影响：客户端写入、密钥过期或被逐出，更改主数据集的任何其他操作。
2. 当主从之间的链接断开时，由于网络问题或因为在主或从中检测到超时，从重新连接并尝试进行部分重新同步：这意味着它将尝试仅获取部分它在断开连接期间错过的命令流。
3. 当无法进行部分重新同步时，副本将要求进行完全重新同步。这将涉及一个更复杂的过程，其中主节点需要创建其所有数据的快照，将其发送到副本，然后随着数据集的变化继续发送命令流。

Redis 默认使用低延迟和高性能的异步复制，这是绝大多数 Redis 用例的自然复制模式。但是，Redis 副本异步确认它们定期从主服务器接收到的数据量。因此主服务器不会每次都等待副本处理命令，但是如果需要，它知道哪个副本已经处理了什么命令。这允许有可选的同步复制。

客户端可以使用该命令请求某些数据的同步复制[`WAIT`](https://redis.io/commands/wait)。但是[`WAIT`](https://redis.io/commands/wait)只能保证其他Redis实例中有指定数量的已确认副本，并不能将一组Redis实例变成强一致性的CP系统：已确认的写入在故障转移期间仍然可能丢失，具体取决于Redis 持久性的确切配置。然而，[`WAIT`](https://redis.io/commands/wait)随着故障事件后丢失写入的可能性大大降低到某些难以触发的故障模式。

您可以查看 Redis Sentinel 或 Redis Cluster 文档，了解有关高可用性和故障转移的更多信息。本文档余下部分主要介绍Redis基本复制的基本特性。

#### 关于 Redis 复制的重要事实 <a href="#important-facts-about-redis-replication" id="important-facts-about-redis-replication"></a>

* Redis 使用异步复制，异步副本到主机确认处理的数据量。
* 一个 master 可以有多个副本。
* 副本能够接受来自其他副本的连接。除了将多个副本连接到同一个主服务器之外，副本还可以以类似级联的结构连接到其他副本。从 Redis 4.0 开始，所有的子副本将从主副本接收完全相同的复制流。
* Redis 复制在 master 端是非阻塞的。这意味着当一个或多个副本执行初始同步或部分重新同步时，master 将继续处理查询。
* 在副本端，复制在很大程度上也是非阻塞的。当副本执行初始同步时，它可以使用旧版本的数据集处理查询，假设您在 redis.conf 中将 Redis 配置为这样做。否则，您可以将 Redis 副本配置为在复制流关闭时向客户端返回错误。但是，在初始同步后，必须删除旧数据集并加载新数据集。副本将在此短暂窗口期间阻止传入连接（对于非常大的数据集可能长达几秒）。从 Redis 4.0 开始，您可以配置 Redis，以便在不同的线程中删除旧数据集，但是加载新的初始数据集仍将在主线程中进行并阻塞副本。
* 复制既可以用于可伸缩性，为只读查询提供多个副本（例如，慢速 O(N) 操作可以卸载到副本），也可以仅仅用于提高数据安全性和高可用性。
* 您可以使用复制来避免让 master 将完整数据集写入磁盘的成本：一种典型的技术涉及配置您的 master`redis.conf`以完全避免持久化到磁盘，然后连接一个配置为不时保存或启用 AOF 的副本. 但是，必须小心处理此设置，因为重新启动的主服务器将从空数据集开始：如果副本尝试与其同步，则副本也将被清空。

### master 关闭持久性时复制的安全性 <a href="#safety-of-replication-when-master-has-persistence-turned-off" id="safety-of-replication-when-master-has-persistence-turned-off"></a>

在使用 Redis 复制的设置中，强烈建议在主服务器和副本服务器中打开持久性。如果这是不可能的，例如由于非常慢的磁盘引起的延迟问题，实例应该被配置为避免在重启后**自动重启。**

为了更好地理解为什么将持久性关闭配置为自动重启的主服务器是危险的，请检查以下故障模式，其中数据从主服务器及其所有副本中擦除：

1. 我们有一个设置，节点 A 充当主节点，持久性被关闭，节点 B 和 C 从节点 A 复制。
2. 节点 A 崩溃了，但是它有一些自动重启系统，可以重启进程。但是，由于持久性已关闭，因此节点会以空数据集重新启动。
3. 节点 B 和 C 将从空的节点 A 进行复制，因此它们将有效地销毁其数据副本。

当 Redis Sentinel 用于高可用性时，同时关闭 master 上的持久性以及进程的自动重启是危险的。例如，master 可以足够快地重新启动，以使 Sentinel 无法检测到故障，从而发生上述故障模式。

每次数据安全很重要，并且复制与主配置一起使用而没有持久性时，应该禁用实例的自动重启。

### Redis 复制的工作原理 <a href="#how-redis-replication-works" id="how-redis-replication-works"></a>

每个 Redis 主节点都有一个复制 ID：它是一个很大的伪随机字符串，用于标记数据集的给定故事。每个 master 还采用一个偏移量，该偏移量随着它生成的要发送到副本的复制流的每个字节而递增，以使用修改数据集的新更改来更新副本的状态。即使没有实际连接的副本，复制偏移量也会增加，所以基本上每对给定的：

```
Replication ID, offset
```

标识主数据集的确切版本。

当副本连接到主服务器时，它们使用[`PSYNC`](https://redis.io/commands/psync)命令发送它们的旧主复制 ID 和它们到目前为止处理的偏移量。这样 master 就可以只发送所需的增量部分。但是，如果主缓冲区中没有足够的_积压_，或者如果副本引用不再已知的历史记录（复制 ID），则会发生完全重新同步：在这种情况下，副本将获得数据集的完整副本， 从头开始​​。

这是完整同步的更多详细信息：

master 启动后台保存过程以生成 RDB 文件。同时它开始缓冲所有从客户端接收到的新写命令。当后台保存完成后，master将数据库文件传输给replica，replica将其保存在磁盘上，然后加载到内存中。master 然后将所有缓冲的命令发送到 replica。这是作为命令流完成的，并且采用与 Redis 协议本身相同的格式。

您可以通过 telnet 自己尝试。在服务器执行某些工作时连接到 Redis 端口并发出命令[`SYNC`](https://redis.io/commands/sync)。您会看到批量传输，然后主服务器收到的每个命令都将在 telnet 会话中重新发出。Actually[`SYNC`](https://redis.io/commands/sync)是较新的 Redis 实例不再使用的旧协议，但仍然存在以实现向后兼容性：它不允许部分重新同步，因此[`PSYNC`](https://redis.io/commands/psync)改为使用 now。

如前所述，当主副本链路由于某种原因断开时，副本能够自动重新连接。如果 master 收到多个并发的副本同步请求，它会执​​行单个后台保存来为所有这些请求提供服务。

### 复制 ID 说明 <a href="#replication-id-explained" id="replication-id-explained"></a>

上一节我们说过，如果两个实例有相同的replication ID和replication offset，那么它们的数据是完全一样的。然而，了解复制 ID 到底是什么，以及为什么实例实际上有两个复制 ID：主 ID 和辅助 ID 是很有用的。

复制 ID 基本上标记了数据集的给定_历史。_每当一个实例从头开始作为主实例重新启动，或者一个副本被提升为主实例时，都会为这个实例生成一个新的复制 ID。连接到主服务器的副本将在握手后继承其复制 ID。因此，具有相同 ID 的两个实例是相关的，因为它们持有相同的数据，但可能在不同的时间。对于给定的历史记录（复制 ID），它是作为逻辑时间理解的偏移量，谁拥有最新的数据集。

例如，如果两个实例 A 和 B 具有相同的复制 ID，但一个偏移量为 1000，一个偏移量为 1023，这意味着第一个缺少应用于数据集的某些命令。这也意味着 A 只需应用几个命令，就可以达到与 B 完全相同的状态。

Redis 实例有两个复制 ID 的原因是因为副本被提升为主副本。故障转移后，提升的副本需要仍然记住它过去的复制 ID 是什么，因为这样的复制 ID 是以前的主复制 ID。这样，当其他副本将与新主控同步时，它们将尝试使用旧主控复制 ID 执行部分重新同步。这将按预期工作，因为当副本被提升为主时，它将其辅助 ID 设置为主 ID，并记住此 ID 切换发生时的偏移量。稍后它将选择一个新的随机复制 ID，因为新的历史开始了。在处理连接的新副本时，master 会将它们的 ID 和偏移量与当前 ID 和辅助 ID 匹配（为了安全起见，直到给定的偏移量）。

如果你想知道为什么提升为 master 的副本需要在故障转移后更改其复制 ID：可能由于某些网络分区，旧的 master 仍在作为 master 工作：保留相同的复制 ID 将违反以下事实：任意两个随机实例的相同 ID 和相同偏移量意味着它们具有相同的数据集。

### 无盘复制 <a href="#diskless-replication" id="diskless-replication"></a>

通常，完全重新同步需要在磁盘上创建一个 RDB 文件，然后从磁盘重新加载相同的 RDB 以向副本提供数据。

对于慢速磁盘，这对主服务器来说可能是一个压力很大的操作。Redis 2.8.18 版本是第一个支持无盘复制的版本。在此设置中，子进程直接通过线路将 RDB 发送到副本，而不使用磁盘作为中间存储。

### 配置 <a href="#configuration" id="configuration"></a>

配置基本的 Redis 复制很简单：只需将以下行添加到副本配置文件中：

```
replicaof 192.168.1.1 6379
```

当然，您需要将 192.168.1.1 6379 替换为您的主 IP 地址（或主机名）和端口。或者，您可以调用该[`REPLICAOF`](https://redis.io/commands/replicaof)命令，主控主机将开始与副本同步。

还有一些参数用于调整主服务器在内存中获取的复制积压以执行部分​​重新同步。`redis.conf`有关详细信息，请参阅 Redis 发行版附带的示例 。

可以使用`repl-diskless-sync`配置参数启用无盘复制。在第一个副本之后开始传输以等待更多副本到达的延迟由参数控制`repl-diskless-sync-delay` 。`redis.conf`有关详细信息，请参阅 Redis 发行版中的示例文件。

### 只读副本 <a href="#read-only-replica" id="read-only-replica"></a>

从 Redis 2.6 开始，副本支持默认启用的只读模式。此行为由 redis.conf 文件中的选项控制`replica-read-only`，并且可以在运行时使用启用和禁用[`CONFIG SET`](https://redis.io/commands/config-set)。

只读副本会拒绝所有写命令，这样就不可能因为错误而写入副本。这并不意味着该功能旨在将副本实例暴露给 Internet，或者更普遍地暴露给存在不受信任的客户端的网络，因为诸如 或 之类的管理命令仍处于[`DEBUG`](https://redis.io/commands/debug)启用[`CONFIG`](https://redis.io/commands/config)状态。安全页面描述了如何保护 Redis 实例[。](https://redis.io/topics/security)

您可能想知道为什么可以恢复只读设置并拥有可作为写操作目标的副本实例。答案是可写副本的存在只是出于历史原因。使用可写副本会导致主从不一致，所以不推荐使用可写副本。要了解在哪些情况下这会成为问题，我们需要了解复制的工作原理。通过将常规 Redis 命令传播到副本来复制主服务器上的更改。当 master 上的密钥过期时，这将作为 DEL 命令传播。如果一个密钥存在于主服务器上但被删除、过期或在副本上与主服务器具有不同的类型，那么对从主服务器传播的 DEL、INCR 或 RPOP 等命令的反应将不同于预期。传播的命令可能在副本上失败或导致不同的结果。为了最大限度地降低风险（如果您坚持使用可写副本），我们建议您遵循以下建议：

* 不要写入也在主服务器上使用的可写副本中的密钥。（如果您无法控制所有写入 master 的客户端，则很难保证这一点。）
* 在升级正在运行的系统中的一组实例时，不要将实例配置为可写副本作为中间步骤。一般来说，如果要保证数据一致性，如果实例可以升级为主实例，则不要将实例配置为可写副本。

从历史上看，有一些被认为是可写副本合法的用例。从 7.0 版开始，这些用例现在都已过时，可以通过其他方式实现。例如：

* [使用SUNIONSTORE](https://redis.io/commands/sunionstore)和[ZINTERSTORE](https://redis.io/commands/zinterstore)等命令计算缓慢的 Set 或 Sorted set 操作并将结果存储在临时本地键中。相反，使用返回结果而不存储结果的命令，例如[SUNION](https://redis.io/commands/sunion)和[ZINTER](https://redis.io/commands/zinter)。
* 使用[SORT](https://redis.io/commands/sort)命令（由于可选的 STORE 选项，它不被视为只读命令，因此不能用于只读副本）。相反，使用[SORT\_RO](https://redis.io/commands/sort\_ro)，这是一个只读命令。
* 使用[EVAL](https://redis.io/commands/eval)和[EVALSHA](https://redis.io/commands/evalsha)也不被认为是只读命令，因为 Lua 脚本可能会调用写命令。相反，请使用[EVAL\_RO](https://redis.io/commands/eval\_ro)和[EVALSHA\_RO](https://redis.io/commands/evalsha\_ro)，其中 Lua 脚本只能调用只读命令。

如果副本和主服务器重新同步，或者如果副本重新启动，对副本的写入将被丢弃，但不能保证它们会自动同步。

在 4.0 版之前，可写副本无法使设置了生存时间的密钥过期。这意味着，如果您使用[`EXPIRE`](https://redis.io/commands/expire)或其他为密钥设置最大 TTL 的命令，密钥将会泄漏，虽然您在使用读取命令访问它时可能不再看到它，但您会在密钥计数中看到它并且它将还是用内存。Redis 4.0 RC3 及更高版本能够像 masters 一样使用 TTL 逐出密钥，但以大于 63 的数据库编号写入的密钥除外（但默认情况下 Redis 实例只有 16 个数据库）。请注意，即使在 4.0 以上的版本中，使用[`EXPIRE`](https://redis.io/commands/expire)可能存在于主服务器上的密钥也会导致副本和主服务器之间的不一致。

另请注意，由于 Redis 4.0 副本写入只是本地的，不会传播到附加到实例的子副本。相反，子副本将始终接收与顶级主服务器发送给中间副本的复制流相同的复制流。因此，例如在以下设置中：

```
A ---> B ---> C
```

即使是`B`可写的，C 也看不到`B`写入，而是拥有与主实例相同的数据集`A`。

### 设置副本以向主服务器进行身份验证 <a href="#setting-a-replica-to-authenticate-to-a-master" id="setting-a-replica-to-authenticate-to-a-master"></a>

如果您的主服务器通过 获得密码`requirepass`，则可以轻松地将副本配置为在所有同步操作中使用该密码。

要在正在运行的实例上执行此操作，请使用`redis-cli`并键入：

```
config set masterauth <password>
```

要永久设置它，请将其添加到您的配置文件中：

```
masterauth <password>
```

### 仅允许写入 N 个附加副本 <a href="#allow-writes-only-with-n-attached-replicas" id="allow-writes-only-with-n-attached-replicas"></a>

从 Redis 2.8 开始，您可以将 Redis 主服务器配置为仅当至少 N 个副本当前连接到主服务器时才接受写查询。

但是，由于 Redis 使用异步复制，因此无法确保副本确实收到给定的写入，因此始终存在数据丢失的窗口。

这是该功能的工作原理：

* Redis 副本每秒 ping 主服务器，确认已处理的复制流的数量。
* Redis masters 会记住上次从每个副本收到 ping 的时间。
* 用户可以配置延迟不超过最大秒数的最小副本数。

如果至少有 N 个副本，延迟小于 M 秒，则写入将被接受。

您可能认为它是一种尽力而为的数据安全机制，其中不能确保给定写入的一致性，但至少数据丢失的时间窗口被限制在给定的秒数内。一般来说，绑定数据丢失比未绑定数据丢失要好。

如果不满足条件，master 将回复一个错误并且不接受写入。

此功能有两个配置参数：

* 最小副本写入`<number of replicas>`
* 最小副本最大滞后`<number of seconds>`

`redis.conf`有关详细信息，请查看Redis 源代码分发版附带的示例文件。

### Redis 复制如何处理键过期 <a href="#how-redis-replication-deals-with-expires-on-keys" id="how-redis-replication-deals-with-expires-on-keys"></a>

Redis 过期允许键具有有限的生存时间 (TTL)。这样的功能取决于实例计算时间的能力，但是 Redis 副本可以正确复制过期的密钥，即使使用 Lua 脚本更改此类密钥也是如此。

要实现这样的功能，Redis 不能依赖主从同步时钟的能力，因为这是一个无法解决的问题，会导致竞争条件和数据集发散，因此 Redis 使用三种主要技术来使过期密钥的复制能够工作：

1. 副本不会使密钥过期，而是等待主节点使密钥过期。当 master 使密钥过期（或由于 LRU 而将其逐出）时，它会合成一个[`DEL`](https://redis.io/commands/del)命令，该命令将传输到所有副本。
2. 然而，由于 master-driven 过期，有时副本可能仍然具有逻辑上已经过期的内存密钥，因为 master 无法[`DEL`](https://redis.io/commands/del)及时提供命令。为了解决这个问题，副本使用其逻辑时钟报告密钥不存在，仅用于不违反数据集一致性的**读取操作（因为来自主服务器的新命令将到达）。**以这种方式，副本避免报告仍然存在的逻辑过期密钥。实际上，使用副本进行缩放的 HTML 片段缓存将避免返回已经超过所需生存时间的项目。
3. 在 Lua 脚本执行期间，不会执行密钥过期。当 Lua 脚本运行时，从概念上讲，master 中的时间被冻结，因此在脚本运行的所有时间里，给定的键要么存在，要么不存在。这可以防止密钥在脚本中间过期，并且需要以保证在数据集中具有相同效果的方式将相同的脚本发送到副本。

一旦副本被提升为主人，它将开始独立地使密钥过期，并且不需要其旧主人的任何帮助。

### 在 Docker 和 NAT 中配置复制 <a href="#configuring-replication-in-docker-and-nat" id="configuring-replication-in-docker-and-nat"></a>

当使用 Docker 或其他类型的使用端口转发或网络地址转换的容器时，Redis 复制需要格外小心，尤其是在使用 Redis Sentinel 或其他系统时，在这些系统中扫描主机或命令输出以发现副本的[`INFO`](https://redis.io/commands/info)地址[`ROLE`](https://redis.io/commands/role)。

问题是[`ROLE`](https://redis.io/commands/role)命令和输出的复制部分[`INFO`](https://redis.io/commands/info)，当发布到主实例时，将显示副本具有用于连接到主实例的 IP 地址，在使用 NAT 的环境中，与副本实例的逻辑地址（客户端应该用来连接到副本的地址）。

类似地，副本将与配置为中的侦听端口一起列出`redis.conf`，如果端口被重新映射，则可能与转发端口不同。

为了解决这两个问题，从 Redis 3.2.2 开始，可以强制副本向主服务器宣布任意一对 IP 和端口。要使用的两个配置指令是：

```
replica-announce-ip 5.5.5.5
replica-announce-port 1234
```

并记录在`redis.conf`最近的 Redis 发行版示例中。

### INFO 和 ROLE 命令 <a href="#the-info-and-role-command" id="the-info-and-role-command"></a>

有两个 Redis 命令提供了有关主实例和副本实例的当前复制参数的大量信息。一个是[`INFO`](https://redis.io/commands/info)。`replication`如果使用参数as 调用命令，则`INFO replication`仅显示与复制相关的信息。另一个对计算机更友好的命令是[`ROLE`](https://redis.io/commands/role)，它提供主服务器和副本的复制状态及其复制偏移量、连接的副本列表等。

### 重启和故障转移后的部分同步 <a href="#partial-sync-after-restarts-and-failovers" id="partial-sync-after-restarts-and-failovers"></a>

从 Redis 4.0 开始，当一个实例在故障转移后被提升为 master 时，它仍然能够与旧 master 的副本执行部分重新同步。为此，副本会记住其前主控的旧复制 ID 和偏移量，因此可以向连接的副本提供部分积压，即使它们要求旧复制 ID。

然而，提升副本的新复制 ID 将不同，因为它构成了数据集的不同历史记录。例如，master 可以返回可用并且可以继续接受写入一段时间，因此在提升的副本中使用相同的复制 ID 将违反复制 ID 和偏移量对仅标识单个数据集的规则。

此外，副本 - 当轻轻关闭并重新启动时 - 能够在`RDB`文件中存储与主副本重新同步所需的信息。这在升级时很有用。当需要时，最好使用[`SHUTDOWN`](https://redis.io/commands/shutdown)命令对`save & quit`副本执行操作。

无法部分同步通过 AOF 文件重新启动的副本。然而，实例可能在关闭之前转向 RDB 持久化，然后可以重新启动，最后可以再次启用 AOF。

### `Maxmemory`在副本上 <a href="#maxmemory-on-replicas" id="maxmemory-on-replicas"></a>

默认情况下，副本将忽略`maxmemory`（除非它在故障转移后或手动提升为主）。这意味着密钥的逐出将由主服务器处理，将 DEL 命令发送到副本，作为主服务器端的密钥逐出。

此行为可确保主服务器和副本服务器保持一致，这通常是您想要的。但是，如果您的副本是可写的，或者您希望副本具有不同的内存设置，并且您确定对副本执行的所有写入都是幂等的，那么您可以更改此默认值（但一定要了解您在做什么).

请注意，由于默认情况下副本不会逐出，它最终可能会使用比通过设置的内存更多的内存`maxmemory`（因为副本上的某些缓冲区可能更大，或者数据结构有时可能占用更多内存等等）。确保监控副本，并确保它们有足够的内存，不会在主服务器达到配置设置之前遇到真正的内存不足情况`maxmemory`。

要更改此行为，您可以允许副本不忽略`maxmemory`. 要使用的配置指令是：

```
replica-ignore-maxmemory no
```

\
