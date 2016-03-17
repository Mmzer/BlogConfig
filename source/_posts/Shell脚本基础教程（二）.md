---
title: Shell脚本基础教程（二）
date: 2016-03-10 17:06:00
tags: Shell编程
---

### shell变量
##### 定义：

- 首个字符必须为字母(a-z,A-Z)
- 中间不能有空格，可以使用下划线(_)
- 不能使用标点符号
- 不能使用bash里边的关键字（可以使用help）命令查看保留关键字

例如：

```
def_name="mmzer"
```

注意一点：变量名和等号之间不能有空格，否则会报错

<!--more-->

##### 使用变量：

- 直接在变量名前加上"$"引用
- 使用类似"${def_name}"来引用

例如:

```
def_name="mmzer"
echo $def_name
echo ${def_name}
```

变量名外加花括号是可选的，加花括号是帮助识别变量的边界，比如这种情况：

```
def_name="java"
echo "My name is $def_namescript"
```

执行结果：
```
My name is
```
加了花括号则执行结果为：
```
My name is javascript
```

所以，在编程时候，一般都给变量加个花括号，这是一个好习惯！

##### 变量操作
1、变量重新赋值

变量重新赋值时候不能写成这样:$def_name="balabala",使用的时候才能加上$，例如：

```
your_name="name"
echo ${your_name}
your_name="mmzer"
echo ${your_name}
```

2、只读变量

使用 readonly 命令可以将变量定义为只读变量，只读变量的值不能被改变。

```
myName="mmzer"
readonly myName
```

2、删除变量

使用 unset 命令可以删除变量。

```
unset var_name
```

##### 变量类型
1、分类

- *局部变量：* 局部变量是在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。
- *环境变量：* 所有的程序，包括shell启动的程序，都能访问换了变量，类似于其它语言的全局变量。有些车型需要环境变量来保证其正常运行，必要时候shell脚本也可以定义环境变量。
- *shell变量：*  shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行

2、shell字符串

- 最常用的类型
- 可以用单引号定义，也可以用双引号定义

单引号字符串的限制：

- 单引号里的任何字符都会原样输出，单引号字符串中的变量是无效的；
- 单引号字串中不能出现单引号（对单引号使用转义符后也不行）。

例如：

```
def_name="java"
echo 'My name is $def_namescript'
```

输出结果为：

```
My name is $def_namescript
```


双引号的优点：

- 双引号里可以有变量
- 双引号里可以出现转义字符

例如：

```
def_name="java"
echo "My name is $def_namescript"
```

输出：

```
My name is javascript
```

3、数值
可以直接赋值，例如：

```
name=10
echo $name
```

输出为10。

