title: "gerrit相关"
date: 2015-08-18 16:13:01
tags: git
categories: git
---
<!-- toc -->
> Gerrit，一种免费、开放源代码的代码审查软件，使用网页界面。利用网页浏览器，同一个团队的软件程序员，可以相互审阅彼此修改后的程序代码，决定是否能够提交，退回或者继续修改。它使用Git作为底层版本控制系统。

<!--more-->
# 与gerrit建立连接
安装好git后先将本地的git信息加入自己的用户名和密码
```bash
$ git config --global user.name "xxx"
$ git config --global user.email "xxx@gmail.com"

```
然后在进入git bash 生成ssh key
```bash
$ ssh-keygen -t rsa -C "xxx@gmail.com"

```
执行这段命令后会生成两个文件，id_rsa和id_rsa.pub，将id_rsa.pub文件打开，把里面的内容复制到gerrit服务器的public key中，就与gerrit服务器建立了连接。
![](http://7xicmj.com1.z0.glb.clouddn.com/QQ截图20150816002313.png)
建立好连接后即可使用git clone命令将代码克隆到本地

# 使用phpStorm中的gerrit插件开发项目
phpstorm中点击`File->settings->plugins`,弹出窗口点击如图所示位置
![](http://7xicmj.com1.z0.glb.clouddn.com/QQ截图20150818164604.png)
在插件库中找到gerrit插件并安装，安装完成后重启phpstorm，配置好后就可以使用gerrit，一切web页面的gerrit操作都可以在插件中完成。

# gerrit中的疑难杂症
## 报conflict的解决方法
```bash
$ git stash （把本地改动暂存起来）
$ git fetch ssh://yhwang@10.20.10.30:29418/sdk1204 refs/changes/46/26746/2 && git checkout -b hotfix FETCH_HEAD（这条命令可以在gerrit上的页面里找到，不过要增加“-b hotfix"）
$ git pull --rebase origin BRANCH_ACTIONTEC_SDK1208_TELUS_T2200H（需要rebase的分支名）
```
会报告冲突，然后运行下述命令：
```bash
git mergetool --tool=vimdiff （解决冲突）
```
解决完所有冲突之后：
```bash
$ git rebase --continue
$ git push gerrit HEAD:refs/for/BRANCH_ACTIONTEC_SDK1208_TELUS_T2200H （重新提交patch set）
$ git checkout BRANCH_ACTIONTEC_SDK1208_TELUS_T2200H (切换回原来的分支）
$ git branch -D hotfix （把临时分支删除）
$ git stash pop （恢复本地暂存）
```

## 避免merge commit的方法
更新代码时使用`git pull --rebase`, 如果有冲突的话，可以手工修改`git mergetool --tools=vimdiff`, 或者直接用你的IDE修改有冲突的文件， 解决完成冲突之后， 用`git rebase --continue`继续

如果是合并分支的话，可以使用 如下命令，这样git merge的时候会避免merge commit生成
```bash
$ git config --global merge.ff only
```
这里是个实例：
```bash
$ git checkout master
$ git checkout -b feature/foo

# make some commits

$ git rebase master
$ git checkout master
$ git merge --ff-only feature/foo
```

## 在原有commit上创建patch set的方法
首先stash本地改动，之后在gerrit上找到当次提交，并fetch下来
```bash
$ git fetch ssh://boylee@redmin.loongjoy.com:18084/LenovoCrm refs/changes/66/366/2 && git checkout -b hotfix FETCH_HEAD
```
此时在本地生成了一个hotfix分支，该分支最后一次提交即为需要修改的提交，然后运行
```bash
$ git reset --soft HEAD~1
```
回退一个版本，则本地最后一次commit信息会被回退，此时在hotfix分支上做修改后重新提交并push到develop分支，即可在原有提交上创建patch set