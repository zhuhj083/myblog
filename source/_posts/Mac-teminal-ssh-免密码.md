---
title: Mac teminal ssh 免密码
date: 2018-12-21 19:13:16
tags: [mac,ssh]
categories: Mac
---
# 一、创建expect脚本
1. 在/usr/local/bin目录下创建item2ssh.sh脚本
```bash
#!/usr/bin/expect

set timeout 15
set port [lindex $argv 0]
set username [lindex $argv 1]
set host [lindex $argv 2]
set passwd [lindex $argv 3]
spawn ssh -p $port $username@$host
expect {
        "(yes/no)?"
        {send "yes\n";exp_continue}
        "password:"
        {send "$passwd\n"}
}
interact
```

2. 赋予可执行权限
```bash
cd /usr/local/bin
sudo chmod +x item2ssh.sh
```

# 二、创建alias
3. 创建alias
在~目录下，创建.bash_alies文件，内容如下
```bash
alias ssh2std='item2login.sh 22 username ip passwd'
```
4. 修改.bash_profile
添加以下几行
```bash
if [ -f ~/.bash_aliases ]; then
    source ~/.bash_aliases
fi
```

5.source使之生效
```bash
source ~/.bash_profile
```
这样以后，直接敲ssd2std 就可以免密码ssh登录到std机器上了。

# 三、item2中使用
在item2软件中，`comand`+`,` 弹出Preferences，
然后可以添加Profiles,如下图所示
![https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/item2.jpg](https://raw.githubusercontent.com/zhuhj083/storehouse/master/pictures/hexo/item2.jpg)
