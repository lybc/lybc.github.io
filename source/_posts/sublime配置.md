title: "sublime配置"
date: 2015-10-22 17:29:59
tags: Sublime Text 3
categories: 收藏品
---
<!-- toc -->
# sublime text 3 中运行PHP文件
一、将PHP安装目录放如环境变量PATH
二、添加PHP的build system

<!-- more -->

1）进入如下菜单：

![](http://img.blog.csdn.net/20140614175924156)

2）弹出内容如下：
```json
{
    "cmd": ["make"]
}
```
修改为
```json
{
    "cmd": ["php", "$file"],
    "file_regex": "php$", 
    "selector": "source.php"
}
```
3）保存在默认的目录下即可，注意修改文件名为 php.sublime-build 。

执行快捷键为Ctrl+B。

# sublime text执行Python脚本
安装sublimeREPL插件

在Tools -> sublimeREPL -> Python -> Python - RUN current file 可以运行当前文件
或者在Preferences -> Key Bindings - User中写入如下配置
```json
[
	{ 
		"keys": ["f5"],
		"caption": "SublimeREPL:Python - RUN current file",
		"command": "run_existing_window_command",
		"args": {"id": "repl_python_run","file":"config/Python/Main.sublime-menu"}
	}
]

```
完成后在代码中按F5即可执行当前Python脚本

# OmniMarkupPreviwer
sublime text3 markdown 预览插件，浏览器实时预览无需刷新

1. <kbd>ctrl</kbd>+<kbd>shift</kbd>+<kbd>p</kbd> 
2. install package
3. OmniMarkupPreviwer