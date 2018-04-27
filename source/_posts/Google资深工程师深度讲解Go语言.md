---
title: Google资深工程师深度讲解Go语言
date: 2018-03-28 15:15:30
tags: [Go, 基础]
category: Go
---
# 1.byte(4字节8位),rune(char类型,32位,4字节的int32)
# 2.强制类型转换,go没有隐式类型转换
```go
//勾股定理
a, b := 3, 4 //int
var c int
//Sqrt(float64)
c = math.Sqrt(a*a+b*b)
//>> 类型错误
//必须显示的进行类型转换
c = int(math.Sqrt(float64(a*a+b+b)))
//如果是const且没有定义类型的话不会报错
const a, b = 3,4
c := math.Sqrt(a*a +b*b)
//>> 5
```
# 3.iota
```go
const(
	b=1 <<(10*iota)
	kb
	mb
	tb
	pb
)
```
# 4.指针
golang只有值传递一种方式
![Alt text](./1522322191257.png)
下面这种方式`看着像引用传递,实际是复制了a的指针地址然后传递`达到了引用传递的效果
![Alt text](./1522322152670.png)
# 5.Slice
![Alt text](./1522375003785.png)
* slice可以向后扩展,但无法向前扩展
* s[i]不可以超越len(s),向后扩展不能超过cap(s)
* 添加元素时如果超越cap,系统会重新分配更大的底层数组
```go
s := []int{0,1,2,3,4,5,6,7}
s1 :=s[2:6]
//>> [2,3,4,5]
s2 := s1[4]
//>> Error 下标越界了
s3 := s1[3:5]
//>> [5,6] 向后扩展了一个6
s4 := append(s3,10)
//>>s4: [5,6,10]
//>>s: [0,1,2,3,4,5,6,10] 原来的7被append为10了
s5 := append(s4,11)
//>>s5: [5,6,10,11]
//>>s: [0,1,2,3,4,5,6,10]当cap不够用的时候,系统已经重新扩展了一个新的slice就不再是操作原来的slice了.原来的slice如果有人用才会存在,否则垃圾回收机制会把回收了.
//copy
s6 := make([]int, 8, 16)
copy(s6, s5)
//>> [5,6,10,11,0,0,0,0] len:8 cap:16
//delete因为go没有内置的delete方法,所以用append来实现
s7 := append(s5[:3], s5[4:]...)
//>> [5,6,10,0,0,0,0] len:8 cap:16
```
# 6.Map
* 创建 make(map[string]string)
* 获取 m[key]
* key不存在时,获取的是零值
* value, ok :=m[key]判断值是否存在
* delete(m, "key")删除

# 7.Rune(相当于go的char类型)
 * 使用range遍历pos,rune对
 * 使用utf8.RuneCountString()获取字符数量,而不是直接使用len()
 * 使用len()只是获取到字节长度
 * 使用[]byte获得字节

# 8.值接收者VS指针接收者
* 要改变内容必须使用指针接收者
* 结构体过大也考虑使用指针接收者
* 一致性:如有指针接收者,最好都是指针接收者
* 值接收者是go特有的
* 值/指针接收者均可以接收值/指针

# 9.接口
![Alt text](./1522636423539.png)
![Alt text](./1522636466984.png)
![Alt text](./1522636555167.png)
Type Assertion
.(type类型)获取interface肚子里的类型
```go
r.(*real.Retriver)
```
# 10.函数式编程
![Alt text](./1522638766633.png)
![Alt text](./1522638856776.png)
![Alt text](./1522639321599.png)
裴波纳契
```go
func fibonacci() func() int {
  a, b := 0, 1
  return func() int {
    a, b = b, a+b
    return a
  }
}
func main(){
	f := fabonacci()
	f() //1
	f() //1
	f() //2
	f() //3
	f() //5
}
```
# 11.资源管理
![Alt text](./1522653776704.png)
![Alt text](./1522653870889.png)
![Alt text](./1522654193952.png)
![Alt text](./1522740353096.png)
![Alt text](./1522740395657.png)
![Alt text](./1522741904099.png)
# 12.测试
![Alt text](./1522743849182.png)
![Alt text](./1522744008349.png)
```powershell
go test #测试当前目录
go test --coverprofile=c.out #生成c.out文件
go tool cover -html=c.out #将c.out文件生成html格式并用浏览器打开
```
![Alt text](./1522745826671.png)
```powershell
go test -bench . #benchmark测试
go test -bench . -cpuprofile=cpu.out
go tool pprof cpu.out
```
![Alt text](./1522759597530.png)
```powershell
godoc -http :6060 #开启webservice文档
```
![Alt text](./1522933077902.png)
# 13.Goroutine
![Alt text](./1522933919194.png)
```go
runtime.Gosched()//手动交出控制权
```
```powershell
go run -race main.go #race condition 数据访问冲突,进行错误检测
```
![Alt text](./1522934660041.png)
![Alt text](./1522934767052.png)
![Alt text](./1522935142751.png)
![Alt text](./1522935187304.png)
# 14.Channel
![Alt text](./1522935343442.png)
![Alt text](./1522938466872.png)
![Alt text](./1523012404646.png)
# 15.http
![Alt text](./1523068699367.png)
![Alt text](./1523068751236.png)
第一种
```go
import _ "net/http/pprof"
//server func ...
//然后就可以通过url/debug/pprof进行访问
```
第二种
```powershell
#会获得30秒的CPU使用率
go tool pprof http://localhost:8888/debug/pprof/profile
#会获得30秒的内存使用率
go tool pprof http://localhost:8888/debug/pprof/heap
#期间访问想要测试的http request,30秒后进入prof模式,可以使用web进行查看
```
# 16.广度优先算法
1.探索顺序:上左下右
2.结点的三种状态:
* 已经发现还未探索(最关键,这些未探索的点需要排队,不能急于探索必须轮到才能探索)
* 已经发现并且探索
* 连发现都没发现

![Alt text](./1523070115303.png)
![Alt text](./1523070126952.png)
![Alt text](./1523070135463.png)
![Alt text](./1523070180156.png)
![Alt text](./1523070188843.png)
![Alt text](./1523070357078.png)
一层一层往外递进,确保每到一个点都是用最短的路径到达
![Alt text](./1523070504150.png)
![Alt text](./1523070546138.png)
![Alt text](./1523070594114.png)
![Alt text](./1523070622309.png)
![Alt text](./1523070643640.png)
最终倒过来走就是最短路径
3.结束条件:
* 走到终端
* 队列为空(死路)
