---
title: RPM、SRPM与YUM 常用命令
date: 2018-05-14 15:26:14
tags: [linix,rpm,srpm,yum]
categories: linux
---
# rpm
## rpm 安装（install）
`rpm -ivh  package-name`
> 参数：
* -i：install的意思
* -v：查看更详细的安装信息画面
* -h：以安装信息栏显示安装进度

## rpm 升级与更新（upgrade/freshen）
以`-Uvh` 或 `-Fvh` 来升级
* -Uvh：后面接的软件即使没有安装过，则系统将予直接安装，若安装过，则系统自动更新到新版
* -Fvh：后面接的软件并未安装到你的Linux系统上，则该软件不会被安装。

## rpm 查询（query）
RPM在查询的时候，其实查询的地方是`/var/lib/rpm`这个目录下的数据库文件

* rpm -qa                             <==已安装软件
* rpm -q[licdR]  已安装软件的名称      <== 已安装软件
* rpm -qf 存在于系统上面的某个文件名    <==已安装软件
* rpm -qp[licdR] 未安装的某个文件名称   <== 查阅RPM文件

参数：<br>
查询已安装软件的信息
* -q：仅查询，后面接的软件名称是否有安装
* -qa：列出所有的已经安装在本机Linux系统上面的所有软件名称
* -qi：列出该软件的详细信息，包含开发商，版本与说明等
* -ql：列出该软件所有的文件与目录所在的完整文件名（list）
* -qc：列出该软件的所有设置文件（找出在/etc/下面的文件名而已）
* -qd：列出该软件的所有帮助文档（找出与man有关的文件而已）
* -qR：列出与该软件有关的依赖软件所含的文件名（Required的意思）
* -qf：由后面接的文件名称找出该文件属于哪一个已安装的软件
* -qp[licdR]：后面接的所有的参数以上面的说明一致，但用途仅在于找出某个RPM文件内的信息

## rpm 验证与数字证书（Verify/Signature）
rpm -Va

rpm -V 已安装的软件名称

rpm -Vp 某个RPM文件的文件名

rpm -Vf 在系统上面的某个文件
> 参数：
* -V：后面加的是软件名称，若该软件所含的文件被改动过，才会列出来
* -Va：列出目前系统上面所有可能被改动过的文件
* -Vp：后面接的是文件名称，列出该软件内可能被改动过的文件
* -Vf：列出某个文件是否被改动过

## rpm 卸载
rpm -e 软件名称

## 重建数据库
rpm --rebuilddb

# srpm文件的安装
`rpmrebuild  --rebuild src-package-name` -----> 将后面的SRPM进行编译与打包，最后产生RPM的文件（仅编译并打包）

`rpmrebuild  --recompile src-package-name` -----> 直接编译、打包并且安装（不仅进行编译打包，还进行安装）


# yum
## yum查询功能 yum [ list | info | search | provides | whatprovides ] 参数
yum [option] [查询工作项目] [相关参数]

参数:  <br>
[option]：主要的参数，包括有：
* -y：当yum要等待用户输入时，这个选项可以自动提供yes的响应
* --installroot=/some/path ： 将该软件安装在 /some/path 中而不适用默认路径

[查询工作目录] [相关参数] ：这方面的参数有：
* search：搜索某个软件名称或者是描述（description）的重要关键字
* list：列出目前yum所管理的所有的软件名称与版本，有点类似于rpm -qa
* info：同上，类似于rpm -qai 的运行结果
* provides：从文件中去搜索软件，类似于 rpm -qf的功能

例子：
* yum search raid：搜索磁盘阵列（raid）相关的软件有哪些
* yum info mdadm ： 找出mdadm这个软件的功能为何
* yum list updates ： 列出目前服务器上可供本机进行升级的软件有哪些
* yum provides passwd：列出提供passwd这个文件的软件有哪些

## yum安装/升级功能：yum [install | update] 软件
yum [option] [查询工作项目] [相关参数]
参数：
install：后面接要安装的软件
update：后面接要升级的软件，若要整个系统都更新，就直接 update 即可

## yum删除功能：yum [ remove ] 软件

## yum的设置文件
vi /etc/yum.repos.d/CentOS-Base.repo

## 列出目前 yum server 所使用的容器有哪些
yum repolist all

## yum的软件组功能
yum [组功能] [软件组]

>参数：
* grouplist：列出所有可使用的组列表，例如Development Tools之类的
* groupinfo：后面接group name，则可了解该group内含的所有组名称
* groupinstall：可以安装一整组软件
* groupremove：移除某个组

例如：
> yum groupinstall “Development Tools” 安装开发软件
