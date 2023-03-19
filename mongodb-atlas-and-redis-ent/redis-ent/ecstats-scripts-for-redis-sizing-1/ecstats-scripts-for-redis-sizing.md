---
description: 面对ElastiCache的部署，Redis Ent提供相应的Sizing工具。
---

# 面向ElastiCache的Sizing

ECstats 是一个用于提取 ElasticCache 数据库指标的工具，用来做redis的sizing。该脚本能够处理属于特定 AWS 区域的所有 Redis 数据库，包括单实例、复制和集群数据库。可以在配置中定义多个区域。该脚本将纯粹查询 cloudwatch 的指标。它永远不会连接到 Redis 数据库，也不会向数据库发送任何命令。这个脚本绝不会影响性能和存储在它正在扫描的 Redis 数据库中的数据。该脚本至少需要 CloudwatchReadOnlyAccess 和 AmazonElastiCacheReadOnlyAccess 权限才能提取信息。

部署方式如下：

**先决条件：**该脚本将在安装了 Python 3.6 或更高版本的任何系统上运行。

下载存储库

```
# git clone https://github.com/Redislabs-Solution-Architects/ecstats2 && cd ecstats2
```

准备并激活虚拟环境

```
# python3 -m venv .env && source .env/bin/activate
```

安装必要的库和依赖项

```
# pip install -r requirements.txt
```

复制示例配置文件并更新其内容以匹配您的配置。需要 AWS 用户访问密钥 ID 和秘密访问密钥才能访问您的 AWS ElastiCache 实例。可以在此文件中定义多个 AWS 环境（例如生产、暂存）和 AWS 区域，脚本将处理在 config.ini 文件中定义为单独部分的所有 AWS ElastiCache 实例。

```
# cp config.ini.example config.ini && vim config.ini
```

执行下面的 python 命令来运行脚本。如果文件名与 config.ini 不同，则对配置文件使用 -c 选项

```
# python ecstats.py -c config.ini
```

完成后不要忘记停用虚拟环境

```
# deactivate
```
