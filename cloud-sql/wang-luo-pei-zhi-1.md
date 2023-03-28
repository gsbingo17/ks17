# 网络配置

### 到底为Cloud SQL分配什么样子的私有子网合适？

通常是推荐1个/16的子网的。Cloud SQL需要为可能创建的Replica预留IP地址，所以使用IP地址比预想的要多。

如果遇到了下面的错误，也不要慌张，可以在线增加子网。

{% code overflow="wrap" %}
```
Failed to create subnetwork. Couldn't find free blocks in allocated IP ranges. Please allocate new ranges for this service provider
```
{% endcode %}

你需要增加一个新的子网，大概步骤如下图：\
![](https://lh4.googleusercontent.com/Dbi\_s2cDOO9OO-8zAv87yeEBsMg7Hh17H4TeKuydNQoXzaGoAgxgQ7niWZTGfTeeNpQn7Wr8gZWD-E7EEE2W49sHKIB9n0Cyfe4y-wScW6AhjFZOYz2Ik20lopnMnn4xrhdngqWu63xveDRs8\_lNg41JpAoDRrcKaa8XXbcC4vWEB8H5fLz4WS-v1Ly7hGZTGSg\_tY\_yif7Srbr3y2-1nBLIlQxcSVeyvlEOXA)

![](https://lh6.googleusercontent.com/WyCb6k-p5f5hz7xhY0HiX9IpOEKlSkYPP7HeHubb-HH58DjJi0STS8Ieu9L4Ikt4MpomtQuAwJvYgdaeoAwhE7V-eSIaCZ5A0MXSF19I2Gb2Ij42aQiMj44pkFRHGWsTEVDXnO5oOpLjZh6mPKWbJmdsqijQd\_T\_PleRf9H6k5YFr7s5cdmZ6Knoy304UjzQP5PlQT-uja16MI\_iouOZMTnhkV6ywlh43JW\_\_g)

![](https://lh3.googleusercontent.com/a0zB8N4vHyTnBu\_y0dbJkRQPA2Jnzxg2KAIsMo6jLWL9yC6e4H9fqK3yU6vEAJUwi2wgKYb5ORE5LvJT8nlYHplI6YUqdTPVGERwNZ7oskYlbIBPbOiQdZNcNrocs518FAucSmfg0Ec2FviPT2bPATmkb44GmCsfMjIjQtEL3cefMtHVcATdVJuWlX8b55K8nBQqGE\_Yk6h0tOgoZAD3ezbRK8gDZaRDThzmUA)

![](https://lh3.googleusercontent.com/Mc9XbdSh90MIltnJPPp3M3IDDxILjd5pF7Gea27N-tmFqsNW4Kk2rQI1RMXksdvvnS8Y5\_BcjugGnt-8fjawN2-cFnbVkHUjgeQCK-q4V7AWxDExNcmQUITzKVhThpFoptsI3iw8743LxwYYOkZf7G\_YgnVWZzxRvSnFHQXp7i-K\_M9IU9Lh-skqDgfc8tMpclSf0CdNK6Xf\_lysofVnC3zAbFhZ8s7xvwsWeg)

![](https://lh4.googleusercontent.com/pAJpOLcJhECA75\_Tl3F80rNEOfEI5nv7dJQe0ngUkcCAEYjdCtDWqxiktw5q75tHZ1XWe6eSp8MHZsb2I-oBuInHEo\_72ccxn9MBJZ6BmUDHtTTTK\_lAKbH27YRRXjh9G8RlOinrhl54OdKvvV-rUiztcBZoxJbQsokRoZjRtNjPS-GLvhWVc1WZMps9VJtnXuj-t6fhpg8Xlkpb42UDFOKzq9q9IjcLiDfeCg)

如果在Console操作遇到告警或者报错，可以使用下面Command完成

```
gcloud services vpc-peerings update \
--network=newdefault \
--ranges=test-cloudsql-network1,test-cloudsql-network2 \
--service=servicenetworking.googleapis.com \
--project= \
--force
```

### 可以通过 Cloud SQL 的私有 IP 在 BigQuery 上执行 Federated Queries 吗？

可以通过编辑 Cloud SQL 数据库，勾选 Google Cloud services authorization 选项，允许 BigQuery 等其他的 Google Cloud 服务访问数据并通过私有 IP 进行查询。支持 Cloud SQL MySQL 以及 Cloud SQL PostgreSQL。

<figure><img src="../.gitbook/assets/Screenshot 2023-03-06 at 18.01.33.png" alt=""><figcaption></figcaption></figure>

### 如何将 CloudSQL 实例启动在指定的 IP range 中？

通过 API 的方式，可以在启动实例的时候，指定 allocatedIpRange 将 CloudSQL的主实例或者副本起在指定的 IP range (subnet) 中。

示例如下：

```text
{
  "masterInstanceName": "t1",
  "project": "test-poc-2023",
  "databaseVersion": "MYSQL_8_0_28",
  "name": "t1-ro",
  "region": "europe-west4",
  "deletionProtectionEnabled": true,
  "settings": {
    "settingsVersion": 0,
    "userLabels": {
      "team": "infra",
      "project": "infra",
      "app": "infra-test"
    },
    "ipConfiguration": {
      "ipv4Enabled": false,
      "privateNetwork": "projects/test-poc-2023/global/networks/default",
      "allocatedIpRange": "cloud-sql-ip-range",
      "locationPreference": {
      "zone": "europe-west4-a"
     }
     },
     "databaseFlags": [
      {
        "name": "lower_case_table_names",
        "value": "1"
      },
      {
        "name": "transaction_isolation",
        "value": "READ-COMMITTED"
      }
    ],    
    "tier": "db-custom-2-4096"
  }
}

curl -X POST \
    -H "Authorization: Bearer $(gcloud auth print-access-token)" \
    -H "Content-Type: application/json; charset=utf-8" \
    -d @create_repl_mysql.json \
    "https://sqladmin.googleapis.com/v1/projects/test-poc-2023/instances"
```

如果是通过 Console 或者 gcloud 的方式，可以在创建主实例（Primary）的时候进行配置，例如：

<figure><img src="../.gitbook/assets/Screenshot 2023-03-27 at 16.20.10.png" alt=""><figcaption></figcaption></figure>

在上图示例中，我们将数据库实例启动在 VPC ”new1“ 的 subnet ”new1-ip-range“ 中，通过 Allocated IP Range，可以选择允许的某个或者多个特定的子网。该 Primary 实例如果在后续还启动 read replica 的话, 所有的 replica 的网络配置将继承主实例的配置。也就是说，如果主实例起在”new1-ip-range“的子网中，在该 region 下的所有 read replica 也都启动在这个子网中（注意需要在子网中预留足够的 IP 地址）。
