# 连接配置

### Datastream  支持 Terraform 么？

现在可以使用Datastream和Terraform 结合，而不必费力地为每个数据源单独设置。编译数据源列表，配置相关文件即可。

```
// 例如，配置一个 PostgreSQL 为源端的 Connection Profile

resource "google_datastream_connection_profile" "source" {
    display_name          = "Postgresql Source"
    location              = "us-central1"
    connection_profile_id = "source"

    postgresql_profile {
        hostname = "HOSTNAME"
        port     = 5432
        username = "USERNAME"
        password = "PASSWORD"
        database = "postgres"
    }
}
```

更多内容请参考以下文档：

* [Getting started with Terraform and Datastream: Replicating Postgres data to BigQuery](https://cloud.google.com/blog/products/data-analytics/replicating-postgres-data-to-bigquery)
* [Terraform Doc: google\_datastream\_connection\_profile](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/datastream\_connection\_profile)
