---
title: 推荐一个超好用的Golang数据库迁移工具gobuffalo/pop
date: 2018-04-28 15:45:11
tags: [Go, Package]
category: Go
---
# 前言
这几天被数据库迁移折腾的半死,几乎把`gayhub`上面的一些Star数高一些的都使用了个遍.mattes/migrate虽然星星数多,但是说实在的并不怎么样,反倒是gorm的AutoMigrate最简单实用,就在准备放弃的时候,看到了一篇Povilas Versockas的一篇关于Go schema migration tools的对比文章.顺藤摸瓜,终于找到一个非常满意的Migrate Tool:`markbates/pop`,可惜作者repository已经不更新了,于是乎就找到了接盘侠:`gobuffalo/pop`,讲到这顺带推荐下作者的Web Development Tool `gobuffalo/buffalo`,非常友善的DSL,仿佛是在用go写rails,好吧,感觉有点跑偏了,下面就进入正题
>参考链接:
>[Go schema migration tools](https://povilasv.me/go-schema-migration-tools/)
>[markbates/pop](https://github.com/markbates/pop)
>[gobuffalo/pop](https://github.com/gobuffalo/pop)

# 安装CLI
这个就没啥好说的了,主要注意下,默认是不支持sqlite3的,如果需要的话请自己添加tag
正常版:
```powershell
go get github.com/gobuffalo/pop/...
go install github.com/gobuffalo/pop/soda
```
支持sqlite3
```powershell
go get -u -v -tags sqlite github.com/gobuffalo/pop/...
go install github.com/gobuffalo/pop/soda
```
# 配置database.yml
使用soda g config即可生成配置文件database.yml
```powershell
soda g config
```
*配置文件的位置需要注意下,必须是config/database.yml或者database.yml
Example Configuration File
```yaml
development:
  dialect: "postgres"
  database: "your_db_development"
  host: "localhost"
  port: "5432"
  user: "postgres"
  password: "postgres"

test:
  dialect: "mysql"
  database: "your_db_test"
  host: "localhost"
  port: "3306"
  user: "root"
  password: "root"

staging:
  dialect: "sqlite3"
  database: "./staging.sqlite"

production:
  dialect: "postgres"
  url: {{ env "DATABASE_URL" }}
```
然后只需要在main.go(或者你自己的db init)里面执行Connect即可
```go
db, err := pop.Connect("development")
if err != nil {
  log.Panic(err)
}
```
# CLI Support
```powershell
Available Commands:
  create      Creates databases for you
  drop        Drops databases for you
  generate
  help        Help about any command
  migrate     Runs migrations against your database.
  schema      Tools for working with your database schema
```
## create/drop 
```powershell
soda create -a
```
会根据database.yml的配置生成全部的数据库,含dev,production,test等
当然可以指定environment
```powershell
soda create -e development
```
相反的删除数据库用drop操作
```powershell
soda drop -a
soda drop -e development
```
## models
```powershell
soda generate model user name:text email:text
```
会生成model文件跟测试文件,同时还有数据库迁移fizz文件.
>models/user.go
>models/user_test.go
>migrations/20170115024143_create_users.up.fizz
>migrations/20170115024143_create_users.down.fizz

到这是不是感觉跟rails g model超像的.
下面是model字段支持的类型:
* text (string in Go)
* blob ([]byte in Go)
* time or timestamp (time.Time)
* nulls.Text (nulls.String) which corresponds to a nullifyable string, which can be distinguished from an empty string
* uuid (uuid.UUID)
* Other types are passed thru and are used as Fizz types.

>友情链接: [Fizz支持的类型](https://github.com/gobuffalo/pop/blob/master/fizz/README.md)

默认生成的是fizz文件,如果需要后缀为sql的文件,可以根据--migration-type 指定
```powershell
soda generate model user name:text email:text --migration-type sql -e development
```
>models/user.go
>models/user_test.go
>migrations/20170115024143_create_users.postgres.up.sql
>migrations/20170115024143_create_users.postgres.down.sql

## migrate
有别于model,如果不指名model则是生成migraton文件并不会生成model文件
```powershell
soda generate sql name_of_migration
```
>./migrations/20160815134952_name_of_migration.up.sql
>./migrations/20160815134952_name_of_migration.down.sql

通过up/down 即可对数据库进行迁移操作
```powershell
soda migrate up
soda migrate down
```
最后我们看下fizz文件:
```
create_table("users", func(t) {
  t.Column("email", "string", {})
  t.Column("twitter_handle", "string", {"size": 50})
  t.Column("age", "integer", {"default": 0})
  t.Column("admin", "bool", {"default": false})
  t.Column("company_id", "uuid", {"default_raw": "uuid_generate_v1()"})
  t.Column("bio", "text", {"null": true})
  t.Column("joined_at", "timestamp", {})
})

create_table("todos", func(t) {
  t.Column("user_id", "integer", {})
  t.Column("title", "string", {"size": 100})
  t.Column("details", "text", {"null": true})
  t.ForeignKey("user_id", {"users": ["id"]}, {"on_delete": "cascade"})
})
```
是不是觉得这个DSL非常友善,一目了然,赶紧在你的项目当中用上吧~

