---
title: Golang-Reflect反射
date: 2018-03-27 17:39:52
tags: [Go, Reflect]
category: Go
---
# 1.获取基础信息
```go
package main

import (
	"fmt"
	"reflect"
)

type User struct {
	Id   int
	Name string
	Age  int
}

func (u User) Hello() {
	fmt.Println("Hello")
}

func Info(i interface{}) {
	t := reflect.TypeOf(i)
	fmt.Println("Typeof:", t.Name())

	//获取字段信息
	v := reflect.ValueOf(i)
	for i := 0; i < t.NumField(); i++ {
		f := t.Field(i)
		val := v.Field(i).Interface()
		fmt.Printf("%6s : %v = %v \n", f.Name, f.Type, val)
	}
	//获取方法信息
	for i := 0; i < t.NumMethod(); i++ {
		m := t.Method(i)
		fmt.Printf("%6s :%v\n", m.Name, m.Type)
	}
}
func main() {
	u := User{Id: 1, Name: "Evan", Age: 18}
	Info(u)
}

//Typeof: User
//Id : int = 1  Name : string = Evan   Age : int = 18
```

# 2.获取匿名信息
```go
package main

import (
	"fmt"
	"reflect"
)

type User struct {
	Id   int
	Name string
	Age  int
}

type Manager struct {
	User
	title string
}

func main() {
	m := Manager{User: User{1, "OK", 12}, title: "private_title"}
	t := reflect.TypeOf(m)

	fmt.Printf("%#v\n", t.Field(0))
	//获取到的是User,相对于Manager来说是匿名的，Anonymous:true
	//reflect.StructField{Name:"User", PkgPath:"", Type:(*reflect.rtype)(0x10b4580), Tag:"", Offset:0x0, Index:[]int{0}, Anonymous:true}

	fmt.Printf("%#v\n", t.FieldByIndex([]int{0, 0}))
	//获取到的是User里面的Id字段，相对于User来说是非匿名的，Anonymous:false
	//reflect.StructField{Name:"Id", PkgPath:"", Type:(*reflect.rtype)(0x10a5000), Tag:"", Offset:0x0, Index:[]int{0}, Anonymous:false}
}
```
# 3.修改struct的值(通过反射修改对象的值)
```go
package main

import (
	"fmt"
	"reflect"
)

type User struct {
	Id   int
	Name string
	Age  int
}

func Set(i interface{}) {
	v := reflect.ValueOf(i)
	//ptr 指针类型
	if v.Kind() == reflect.Ptr && !v.Elem().CanSet() {
		fmt.Println("xxx")
		return
	} else {
		// 如果是指针，则获取其所指向的元素
		v = v.Elem()
	}

	//测试设置下Name
	f := v.FieldByName("Name")
	if !f.IsValid() {
		fmt.Println("It's Not Valid")
		return
	}
	if f.Kind() == reflect.String {
		f.SetString("Evan")
	}
}

func main() {
	u := User{1, "OK", 12}
	Set(&u)
	fmt.Println(u)
}
```
# 4.*通过反射动态调用方法
```go
package main

import (
	"fmt"
	"reflect"
)

type User struct {
	Id   int
	Name string
	Age  int
}

func (u User) Hello(name string) {
	fmt.Println("Hello", name, ", My Name is ", u.Name)
}
func main() {
	u := User{1, "OK", 12}
	v := reflect.ValueOf(u) //通过valueOf获取u对象
	mv := v.MethodByName("Hello")

	//创建slice参数
	args := []reflect.Value{reflect.ValueOf("Evan")}
	mv.Call(args)
}
```
