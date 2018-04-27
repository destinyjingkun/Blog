---
title: database/sql常见错误
date: 2018-04-27 15:22:31
tags: [Go, sql]
category: Go
---
# 查询
* 如果方法包含`Query`，那么这个方法是用于查询并返回`rows`的。其他情况应该用Exec(),例如:Prepared Statements和`Exec()`完成`INSERT`, `UPDATE`, `DELETE`
* 占位符?不能替代查询的字段名跟表明

```go
// doesn't work
db.Query("SELECT * FROM ?", "mytable")
// also doesn't work
db.Query("SELECT ?, ? FROM people", "name", "location")
```

# 数据读取
* 如果不遍历整个rows结果，切记调用rows.Close()来释放连接

```go
for rows.Next(){
	...
	break
}
rows.Close()
//其实正解应该还handle Err
if err = rows.Close(); err !=nil{
	log.Println(err)
}
//not defer rows.Close()
```
# 事务
>PS在Tx中唯一绑定一个连接，不会re-prepare。

Tx和statement不能分离，在DB中创建的statement也不能在Tx中使用，因为他们必定不是使用同一个连接使用Tx必须十分小心，例如下面的代码:
```go
tx, err := db.Begin()
if err != nil {
    log.Fatal(err)
}
defer tx.Rollback()
stmt, err := tx.Prepare("INSERT INTO foo VALUES (?)")
if err != nil {
    log.Fatal(err)
}
defer stmt.Close() // danger!
for i := 0; i < 10; i++ {
    _, err = stmt.Exec(i)
    if err != nil {
        log.Fatal(err)
    }
}
err = tx.Commit()
if err != nil {
    log.Fatal(err)
}
// stmt.Close() runs here!
```
`*sql.Tx`一旦释放，连接就回到连接池中，这里stmt在关闭时就无法找到连接。所以必须在Tx commit或rollback之前关闭statement。
# Error处理
* Rows的Error
如果循环中发生错误会自动运行rows.Close()，用`rows.Err()`接收这个错误，Close方法可以多次调用。循环之后判断error是非常必要的。

```go
for rows.Next() {
    // ...
}
if err = rows.Err(); err != nil {
    // handle the error here
}
```
* 关闭Resultsets时的error
如果你在rows遍历结束之前退出循环，必须手动关闭Resultset，并且接收error。上面数据遍历的时候已经谈过了

```go
or rows.Next() {
    // ...
    break; // whoops, rows is not closed! memory leak...
}
// do the usual "if err = rows.Err()" [omitted here]...
// it's always safe to [re?]close here:
if err = rows.Close(); err != nil {
    // but what should we do if there's an error?
    log.Println(err)
}
```
