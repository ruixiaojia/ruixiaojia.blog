---
title: 同时配置 gitlab & github
date: 2019-04-15 10:44:41
tags: git
---

这将是我写的第一篇文章，其实也不能称之为文章吧，就叫分享好了。为什么写这篇分享呢，第一是记性不好，踩过的坑总是忘记，用这种方法记录下来的话可以避免重新踩坑也可以避免忘记了想不起来是怎么回事，也是一种沉淀吧。第二呢就是告诉自己，遇到困难一定尽最大的力去解决，不要总是麻烦别人，学习的道路很艰辛最重要的是成长。第三呢就是希望也能够帮助到同样遇到这样的问题不知该如何解决的你。好了，就不废话了。。。

有些时候我们需要同时使用多个git来进行项目管理，比如不同的项目用了不同的github账号，或者使用gitlab的同时又想在自己的github上面作出一点贡献又能避开红线，那么你就需要配置多个密钥。在此我们默认你已经同时拥有了gitlab和github账号。

#### 第一步，全局配置gitlab/github用户名、邮箱。
```
$ git config --global user.name your_name
$ git config --global user.email your_email@gmail.com
```
##### 接下来，进入你的gitlab/github项目目录，配置本地用户名、邮箱,也可以将其配置为全局（--global）
```
$ git config --local user.name example
$ git config --local user.email youemail@gmail.com
```
##### 可以通过以下命令来查看是否配置成功
```
$ git config user.name
$ git config user.email
```
#### 第二步，生成SSH key
执行以下命令将在～/.ssh 目录下生成对应的gitlab/github的公钥(.pub后缀)、密钥。（～/.ssh/文件夹是默认隐藏的，mac系统使用快捷键"cmd + shift + . " 显示隐藏文件夹）
```
$ ssh-keygen -t rsa -f ~/.ssh/id_rsa_github -C "your_email@gmail.com"
```
```
$ ssh-keygen -t rsa -C "your_email@gmail.com"
```
执行完命令后会在～/.ssh/文件夹下生成id_rsa_github(密钥)、id_rsa_github.pub(公钥)和id_rsa(密钥)、id_rsa.pub(公钥)的文件，需要将.pub后缀的文件内容拷贝到你的github/gitlab内，此过程不再赘述。

##### 执行完以上命令后你只是配置了gitlab和github的用户名、邮箱和公钥，但还是不能进行正常的开发。我们需要让git在操作远程仓库的时候知道它现在是在github还是在gitlab上(通过域名), 所以我们需要通过一个config文件来区分。

#### 第三步，配置config文件
创建config文件
```
$ touch config

$ vi config
```
config内容
```
# github
Host github
User git
HostName github.com
IdentityFile ~/.ssh/id_rsa_github

# gitlab
Host gitlab
User git
HostName gitlab.xxx.com
IdentityFile ~/.ssh/id_rsa
```
- Host: 将要链接的服务器地址。
- User: 默认为git，可不填。
- HostName: 真正链接的服务器地址。
- IdentityFile: 本次链接指定的密钥文件。
- PreferredAuthentications: 指定优先使用哪种方式验证，支持密码和秘钥验证方式。
#### 第四步，添加本地SSH
ssh 连接提示 Permission denied (publickey)；
需要将公钥(publickey)添加到本地SSH环境
```
$ cd ～/.ssh
$ ssh-add id_rsa_github
```
经常不使用会导致本地SSH失效，需要重新执行第四步
#### 第五步，测试
```
ssh -T git@github.com 
```
git@ + "config配置中的HostName"

```
Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
```
那么，恭喜你。