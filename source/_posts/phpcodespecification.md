title: "自己的php代码规范"
date: 2015-05-06 16:00:00
categories: PHP
tags:

-----
<!-- toc -->
在这里写自己的PHP编码规范，仅代表个人习惯，也是对自己的约束。
<!--more-->
# 代码规范 :
**规范的代码很重要！！！**
**规范的代码很重要！！！**
**规范的代码很重要！！！**

## 1.大括号和类名同一行
大括号和类名同一行，和if， else同一行，和函数名同一行，大括号前后空格
```php
class example {
    //..
}

public function foo() {
    //..
}

if(a == 0) {
    //..
} else {
    //..
}
```

## 2.缩进用空格不用tab
代码缩进用四个空格不用tab制表符
```php
class example {
    function a() {
        echo 'hello world';
    }
}
```

## 3.运算符前后空格
比较运算符，算数运算符，赋值运算符前后空格
```php
a == b
a > b
a && b
```

## 4.驼峰法命名
类名首字母大小，函数名、变量名、类名按照驼峰法规范
```php
class ServiceOrder {
    public function getUserById() {
        //..
    }
}
```

## 5.定义常量所有字母大写单词间下划线
```php
const BAR_BAZ = 0;

define('BAR_BAZ', 0);
```

## 6.单行代码不要超过80个字符,注释也是
/* width is within 80 characters */

## 7.定义变量和数组时等号齐平
```php
$name   = 'xiaoming';
$age    = 18;
$height = 175;

$arr = array(
    'name'   => 'xiaoming',
    'age'    => 18,
    'height'
);
```


# 注释规范 :
**注释很重要！！！**
**注释很重要！！！**
**注释很重要！！！**

自从维护了一个1000行的方法而且没有注释之后，我深刻的体会到了注释的重要性

## 1.类文件上面注明日期作者和该类的用途
```php
/**
 * 这里写类的用途
 * @Date   2015-05-06
 * @Author xxxx <aaa.bb@gmail.com>
 */
class Controller {
    //....
}
```

## 2.方法注释写明参数和返回值类型
```php
/**
 * 这里写函数用途
 * @param string $a
 * @param string $b
 * @return string
 */
 public function foo() {
     //....
 }
```

## 3.行内注释与代码齐平
```php
 public function foo() {
     // 这是一个if语句
     if(a == b) {
         // 这是返回值
         return a;
     } else {
         return b;
     }
 }
```
------

*此编码规范暂时适用于我会的其他编程语言，严格遵守编码规范会给程序员带来巨大的好处，要规范！要规范！要规范！（重要的事情说三遍）*