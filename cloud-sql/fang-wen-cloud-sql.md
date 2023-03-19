---
description: 通过混合网络访问Cloud SQL, MemoryStore或FilestoreSQL
---

# 访问Cloud SQL

经常有用户需要在混合云或跨云环境，在本地数据中心或第三方云上访问GCP的托管服务，包含Cloud SQL, MemoryStore, Filestore等。又因安全合规考虑，数据服务不应开放公网IP访问。本教程介绍如何通过Cloud VPN 或Cloud Interconnect从远程客户端、本地或第其他云服务访问具有私有 IP 的 Cloud SQL、Filestore 或 MemoryStore 实例。本教程适用于数据工程师或开发人员，他们希望利用 GCP 上的云数据库或 nas 存储作为其在本地运行的应用程序的后端数据系统，或者希望使用混合网络将数据从本地迁移到云端，例如，在安全隧道内使用 DMS 进行 Cloud SQL 迁移。

## 参考架构 <a href="#_srujtql4d3uv" id="_srujtql4d3uv"></a>

当您在 GCP 上创建数据库或 nas 实例（Cloud SQL、Filestore 或 MemoryStore）时，通过 VPC Network Peering 在您的 VPC 网络和您的数据库或 Filestore 实例所在的底层 Google Cloud VPC 网络之间实现私有连接。

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

远程（本地或其他云服务）网络与 GCP 客户 VPC 网络之间的 VPN 或互连隧道与数据库或 Filestore 底层 VPC 网络对等，可以提供与远程客户端的连接以访问云数据库或 Filestore 实例。

<figure><img src="../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

## 先决条件 <a href="#_pzesjp67k7pv" id="_pzesjp67k7pv"></a>

1.  必须为 Cloud 数据库或 Filestore 启用私有 IP。只要不重叠，自动分配的 IP 范围和自定义 IP 范围都可以使用。以下是 Cloud SQL 设置私有服务连接以启用私有 IP 的示例：

    a. 选择私有 IP 并选择客户 VPC 作为对等网络，然后选择 Enalbe Service Networking API



    <figure><img src="../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

    b. 您可以选择现有的 IP 范围或创建一个不重叠的新 IP 范围，或选择自动分配的 IP 范围

<figure><img src="../.gitbook/assets/image (50).png" alt=""><figcaption></figcaption></figure>

&#x20;       c. 创建连接   &#x20;

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

&#x20;       d. 将设置私有服务连接，对等连接客户 VPC 和 VPC 底层的 Cloud SQL

<figure><img src="../.gitbook/assets/image (55).png" alt=""><figcaption></figcaption></figure>

2. 源 IP 必须在 RFC 1918 IP 范围内，或按照本[指南](https://g3doc.corp.google.com/company/gfw/support/cloud/playbooks/network/non-rfc-1918.md?cl=head)添加例外；
3. 对于Filestore，远程客户端IP不能在172.17.0.0/16范围内；
4. 如果 VPN 网关和 Cloud Router 与与云数据库 VPC 对等的 Customer VPC 不在同一区域，则需要在 VPC 配置中的“动态路由模式”下选择“全局”。

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

## 示例环境信息 <a href="#_n99xbqjxchsd" id="_n99xbqjxchsd"></a>

示例环境的网络信息如下图所示，您可以新建一个GCP VPC作为远程网络进行测试：

<figure><img src="../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

云服务：文件存储

远程（本地或其他云服务）网络子网：172.19.1.0/24

客户 VPC 子网：172.18.1.0/24

Cloud SQL底层VPC子网：10.169.244.200/29

本地路由器/VPN 网关 IP：34.134.215.15

Cloud VPN网关IP：34.134.21.159

## 配置步骤 <a href="#_66s1kgiiqaue" id="_66s1kgiiqaue"></a>

1. 在远程网络和 GCP VPC 网络之间建立 VPN 或 Interconnect 连接，请参阅[Cloud VPN](https://cloud.google.com/network-connectivity/docs/vpn/how-to)和[Cloud Interconnect ](https://cloud.google.com/network-connectivity/docs/interconnect/how-to#managing-dedicated-interconnect)。支持 HA VPN 和 Classic VPN；
2. 在客户 VPC 与 Cloud 数据库或 Filestore 底层 VPC 之间的相应网络对等中启用“导出自定义路由”。例如，filestore-peer-xxxxxxx 是 Customer VPC 和 VPC 下的 Filestore 实例之间的默认网络对等；

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

3.  对于静态路由隧道（基于路由或基于策略的 VPN），执行以下操作：

    a. 添加静态路由（10.169.244.200/29 → vpn\_gw）到云数据库或文件存储实例IP范围和目标VPN网关到远程网络的路由表。例如，AWS VPC 路由：



    ：

    <figure><img src="../.gitbook/assets/image (51).png" alt=""><figcaption></figcaption></figure>

&#x20;      b. 将 Cloud 数据库或 Filestore 实例 IP 范围 (10.169.244.200/29) 添加到远程 VPN 设备的路由表中。例如，AWS VPN 连接静态路由

<figure><img src="../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

4.  对于动态路由隧道（BGP VPN 或互连），执行以下操作：

    a. 将 Cloud 数据库或 Filestore IP 范围添加到 Cloud Router 配置中的自定义 IP 范围

<figure><img src="../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

&#x20;       b. 将 Cloud 数据库或 Filestore IP 范围添加到 BGP 会话配置中的自定义范围

<figure><img src="../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

&#x20;         c. 在步骤 4.a 和 4.b 之后，应将 Cloud 数据库或 Filestore IP 范围的动态路由通告到远程网络路由表。确认路由存在于远程网络路由表中

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

5. 确认网络对等配置下的导出路由中存在远程网络 IP 范围

<figure><img src="../.gitbook/assets/image (43).png" alt=""><figcaption></figcaption></figure>

6. 确认客户 VPC 路由表中存在远程网络 IP 范围

<figure><img src="../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

## 防火墙和访问控制： <a href="#_7h7l478bjkzb" id="_7h7l478bjkzb"></a>

### Filestore <a href="#_y67zumst25bx" id="_y67zumst25bx"></a>

1. 除非使用 NFS 文件锁定，否则不需要防火墙规则。为 NFS 文件锁定设置防火墙规则请参阅[此](https://cloud.google.com/filestore/docs/configuring-firewall)。
2. 在“Grant access to all clients on the VPC network”或者在“Restrict access by IP address or range”区域填入客户端源IP，满足以上环境的访问要求

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption></figcaption></figure>

### Cloud SQL <a href="#_6i3s9dffhoys" id="_6i3s9dffhoys"></a>

1. 在私有 IP 部分下选择客户 VPC 作为网络后，不需要防火墙规则和授权网络

<figure><img src="../.gitbook/assets/image (58).png" alt=""><figcaption></figcaption></figure>

### MemoryStore <a href="#_cm2wgt87hcbb" id="_cm2wgt87hcbb"></a>

1. 在授权 VPC 网络部分下选择客户 VPC 后，不需要防火墙规则和授权网络

<figure><img src="../.gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

## 参考 <a href="#_t0ieiloucrxc" id="_t0ieiloucrxc"></a>

[Cloud SQL - 私有服务访问](https://cloud.google.com/vpc/docs/private-services-access?hl=en)

[Filestore - 在远程客户端上挂载文件共享](https://cloud.google.com/filestore/docs/remote-mounting)
