## Phabricator Arcanist 使用指南
phabricator是facebook开发的codereview工具，截止到目前github star 9064，是目前应用最广泛的codereivew项目，仍在持续更新。目前服务已部署到公司内网 http://localhost/，欢迎体验。

### Phabricator注册
 登录phabricator首页，选择 Register New Account 注册，填写的Email为收取phabricator消息的邮箱，暂时不填写公司邮箱，现在用163邮箱发送邮件，公司邮箱目前还无法接受邮件，建议使用qq或163邮箱。之后登录验证邮箱。
 
现在可以在Phabricator使用部分服务，比如创建task，编写wiki等，提交版本变更还需要arcanist工具。

### 安装arcanist
arcanist是facebook开发的命令行工具，功能类似git，区别是git是将diff提交到版本仓库，arcanist是将diff提交到Phabricator进行评审。

* 安装依赖，安装git以及php。

```
mac系统自带php环境，sudo vim /etc/apache2/httpd.conf，
删除LoadModule php5_module libexec/apache2/libphp5.so
前面的注释，重启aphache即可。
```

* 将arcanist下载到本地

```
    $ mkdir somewhere/
    $ cd somewhere/ 
    somewhere/ $ git clone https://github.com/phacility/libphutil.git
    somewhere/ $ git clone https://github.com/phacility/arcanist.git
```

* 将arc的路径加入到系统路径中

```
vim ~/.bash_profile
加入 export PATH=$PATH:/somewhere/arcanist/bin/
```

### 配置arcanist

* 配置arc默认编辑器，推荐使用vim，默认gnu。

```
$ arc set-config editor "vim"
```
* 在项目根目录下创建`.arcconfig`文件，编辑内容如下

```
	"project_id" : "如果有创建project",
	"conduit_uri" : "http://localhost/"
```
* 执行arc install-certificate 命令安装证书

```
$ arc install-certificate

```
安装证书过程中会出现一个链接，点击链接并登陆之前注册的账号，可以获得一个token，输入token，证书安装完成。
arc就可以关联上注册的账号。

基本配置如以上即可，需要其他详细配置参照官方文档。
[https://secure.phabricator.com/book/phabricator/article/arcanist/](https://secure.phabricator.com/book/phabricator/article/arcanist/)

### 项目中使用arc提交评审
使用`arc diff`提交变更到phabricator.

```
Summary: 填写提交描述（必填）

Test Plan: 填写测试计划(必填 没有测试计划随意填写)

Reviewers: 指定评审人，填写评审人用户名。

```

可以使用`arc diff --update <revision_id>`更新评审。


可以使用`arc diff --create`重新创建评审。

之后就可以登陆phabricator [localhost](localhost) 查看评审了。

### 项目中使用phabricator

### 配置ssh (root用户需要配置)

root用户配置

* 本地创建公钥和私钥钥，其中id_rsa.pub为公钥，id_rsa.pub为私钥

```
cd ~/.ssh   //没有.ssh目录就创建一个
ssh-keygen -t rsa
```

setting=》SSH Public Keys=》Upload Public Key 上传 id_rsa.pub 内容。
运行以下命令测试

`echo {} | ssh git@localhost conduit conduit.ping`

返回以下结果表示配置成功

`{"result":"orbital","error_code":null,"error_info":null}`








