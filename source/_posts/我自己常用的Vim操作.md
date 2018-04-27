---
title: 我自己常用的Vim操作,持续更新!
date: 2018-02-02 17:32:04
tags: [Vim]
category: Vim
---
vim常用
---
@(Vim笔记)
![Alt text](./1524808182243.png)
# 前言
鉴于我的Mac实在太古老了,开大型的ide整个机子散热基本可以蒸肠粉了,于是还是切回到Vim底下进行开发,以下是我自己个人开发当中比较经常用到的一些操作,这个帖子我会不间断的进行更新,有什么比较好的操作都会补上.给那些懒得看Vim tutorial的人上手用.
`小Tip:`
>Vim的Ctrl可以简写成c/C,Shift可以简写成s/S,Enter可以简写成CR/cr
# Normal模式
## 移动
```vim
w	跳到下一个单词的开头
e	跳到下一个单词的末尾
b 	跳到上一个单词的开头
^	移动到当前行开头
%	移动到当前行末尾
ctrl-f/ctrl-b	翻一页
ctrl-d/ctrl-u	翻半页
ctrl-w-方向键	进行窗口跳转
```
## 插入
```vim
i	当前位置插入
I	当前行的最前面位置插入
a	当前位置的后面插入
A	当前行最后位置插入
o	下一行插入
O	上一行插入
```
## 删除
```vim
x	从左往右删除当前位置字符
X	从右往左删除当前位置字符
D	当前位置往后的全删了
dd	删除整行
dw	删除当前位置到下一个单词之间的内容含空格
db	与dw相反
di(	删除括号内的内容delete in ()
类似的还有di[ di{} di[{等
()可以用b代替=>dib,{}可以用B代替=>diB(b是breaket)
u	撤销操作相当于CTRL+Z
ctrl-r	反撤销,相当于ctrl+alt+z
shift-j	两行合并成一行,并添加空格隔开(join)
```
## 复制
```vim
yy	复制整行(y代表yank)
yw	复制一个单词
p	粘贴(paste)
```
## 替换
```vim
cw change当前位置到下一个单词之间的内容,不含空格,并进入插入模式
R从当前位置进入replace模式(*replace模式下,按delete可以单步撤销)
r 替换当前位置字符(R无限个,r仅单个)
同时ci( ci[就是change in ()
```
## 大小写转换
```vim
~		 将光标下的字母改变大小写
3~      将光标位置开始的3个字母改变其大小写
g~~	  改变当前行字母的大小写
gUU		当前行转大写
guu 	当前行转小写
gu=/gU=	大小写转换=是范围
#例如:guw 就是向后一个单词转小写
gU1 	就是向下一行转大写
```
## 搜索
```vim
f-字母	跳到该行的第一个搜索字母上
/word	全篇搜索,查到的结果使用nN进行上下跳转
```
***
# 插入模式
## 代码提示,原生OnmiComplete
```vim
<c-x><c-o>代码提示
<c-n>向下选择
<c-p>向上选择
```
***
# 视图模式
```vim
shlft+v	横向选择
ctrl+v	纵向选择
d	删除选中内容
y	复制选中内容
oO	调整固定点
```
***
# 命令行模式
## 常用
```vim
:tabe-filename	新建一个窗口打开文件(tab edit filename)
:tabn/:tabp		切换窗口
:sp-filename 	水平分割打开文件(split page)
:vs-filename 	垂直分割打开文件(vertical split)
:wa 			保存全部write all
:qa				退出全部quit all
:ls				查看当前已打开的buffer
:b-buffer_number 可切换buffer (num为buffer list中的编号)
:bn 			buffer列表中下一个 buffer
:bp 			buffer列表中前一个 buffer
:b# 			你之前所在的前一个 buffer
```

## 作用范围
>:{作用范围}s/{目标}/{替换}/{替换标志}

作用范围分为当前行、全文、选区等等。
当前行：
>:s/foo/bar/g

全文:
>:%s/foo/bar/g

选区，在Visual模式下选择区域后输入:，Vim即可自动补全为 :'<,'>。
>:'<,'>s/foo/bar/g
>2-11 行
>:5,12s/foo/bar/g
>当前行.与接下来两行+2：
>:.,+2s/foo/bar/g

## 替换标志
上文中命令结尾的g即是替换标志之一，表示全局global替换（即替换目标的所有出现）。 还有很多其他有用的替换标志：

空替换标志表示只替换从光标位置开始，目标的第一次出现：
>:%s/foo/bar

i表示大小写不敏感查找，I表示大小写敏感：
>:%s/foo/bar/i

等效于模式中的\c（不敏感）或\C（敏感）
>:%s/foo\c/bar

c表示需要确认，例如全局查找"foo"替换为"bar"并且需要确认：
>:%s/foo/bar/gc
## 跨文件替换,项目内替换
1.把文件加入到buffer当中
`:arg *.go` 列出当前目录下所有的go文件
`:arg `列出所有的buffer文件
`:argadd *.js`追加所有的js到buffer中

2.执行替换
`:argdo %s/pattern/replace/ge | update`
将所有的.go文件加入到buffer,然后将pattern的内容替换成replace,然后保存文件

