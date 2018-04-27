---
title: HandleFunc与FileServer
date: 2018-03-17 14:47:22
tags: [Go, FileServer]
category: Go
---
假如我们需要在某个请求当中打开一个html
例如localhost:8080/home.html
```yaml
|-public
	|-home.html
	|-home.css
|-main.go

```
home.html
```htmlbars
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>this is home page</title>
    <link rel="stylesheet" href="home.css">
</head>
<body>
<h1>this is home</h1>
</body>
</html>
```
home.css
```css
h1{
    color: red;
}
```
简单的使用HandleFunc
```go
func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		f, err := os.Open("public" + r.URL.Path)
		if err != nil{
			fmt.Println("err:", err)
		}
		defer f.Close()
		io.Copy(w, f)
	})
	http.ListenAndServe(":8080", nil)
}
```
当我们打开页面的时候可以看到h1的颜色并没有变红，检查之后发现h1的格式是text/plain这是默认的Header头，所以我们需要在Content-Type当中声明我们的文件类型
```go
func main() {
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		f, err := os.Open("public" + r.URL.Path)
		if err != nil{
			fmt.Println("err:", err)
		}
		defer f.Close()

		var contentType string
		switch {
		case strings.HasSuffix(r.URL.Path, "css"):
			contentType = "text/css"
		case strings.HasSuffix(r.URL.Path, "html"):
			contentType = "text/html"
		default:
			contentType = "text/plain"
		}
		w.Header().Add("Content-Type", contentType)
		io.Copy(w, f)
	})
	http.ListenAndServe(":8080", nil)
}
```
加入了Content-Type之后可以看到css文件已经可以加载正常了，但是代码实在太长了,如果以后有其它的类型需要加载我们又得往switch里面添加类型。
其实使用go内置的FileSever一行代码就可以解决了
```go
func main() {
	http.ListenAndServe(":8080", http.FileServer(http.Dir("public")))
}
```
