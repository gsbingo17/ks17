# 下载binlog文件

目前，Cloud SQL暂未支持以原生的方式导出binlog。但用户有些时候，为了排障与备份，需要导出binlog文件到本地或其它环境。本教程介绍如何使用mysqlbinlog工具下载Cloud SQL的binlog文件到本地。

安装 mysqlbinlog，mysqlbinlog 包含在 mysql-server 内，需要安装新版本的 mysql-server，否则可能会报错找不到版本，安装参考以下链接：

[https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/#apt-repo-fresh-install](https://dev.mysql.com/doc/mysql-apt-repo-quick-guide/en/#apt-repo-fresh-install)

安装完成后即可使用 mysqlbinlog 来读取 Cloud SQL 上的 binlog 并保存到本地

`mysqlbinlog -u[$User] -p -h[$Host] --read-from-remote-server --raw mysql-bin.XXX > [$File_Name]`

\[$Host] 指 Cloud SQL 实例远程连接地址。

\[$File\_Name] 远程获取 Binlog 文件保存在本地的文件名。

\[$User] 指远程连接使用的用户。

<figure><img src="https://lh5.googleusercontent.com/CxtpkdYIezbCMFgMR5FALkgbMQeztBwzqJR7xJ-tzjtncH6hFzgMs2ySw2C53zGYP2ftfQfhUrn_kkB0xGdLAPbBuK3C2ls7npqeT4EbX6kWiQU2gDdo2lI-sfrJLNfspWh5SYFNyAe4dYTUZb7005E" alt=""><figcaption></figcaption></figure>

在客户端执行以下命令，通过mysqlbinlog工具查看Binlog日志文件内容

`mysqlbinlog -vv --base64-output=decode-rows mysql-bin.XXX | more`

\-vv 参数 为查看具体SQL语句及备注。

\--base64-output=decode-rows 参数 为解析Binlog日志文件。

<figure><img src="https://lh4.googleusercontent.com/GJKc4k9iaMGBGHD7j-koGORFhYD3Sx3p2ir7vfl0g1b2pdEYVX_TZ24aAbFw0A2Y65IAvHu1MmVz1P_cLTuHu4_IcGCn-PQxG3eE1mlucP9cJCaDjg8BQe64Wyw3Exiehbgk7tiYB6VjbDjFZJIcoBQ" alt=""><figcaption></figcaption></figure>
