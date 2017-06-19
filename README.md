# 我的一些工具
## iterms2 work with expect 
一般情况下，公司所有的服务器都在内网，公网访问、管理服务器都要先通过登录一台跳板机，然后再由跳板机登录到相应的服务器进行操作，跳板机与服务器的连接都是内网地址。我们经常看到的现象就是下图这样（博主 Mac 自带的终端做的演示），每次都要通过 ssh 登录两次，输入两次密码，密码也经常输错，不胜其烦。

这时候我们就需要用比较好的工具来解决这个问题，能够实现自动登录，避免时间耗到这种无意义的事情上。我所用到的工具是 iTerm2，iTerm2 是一款非常好用的 Mac 终端工具，具体介绍及基本用法可自行搜索。当然只有iTerm2 还不够，还要配合 Linux expect 的脚本才能实现自动登录。
解决方法
expext 脚本
通过跳板机登录内网服务器，如果只登陆有外网的服务器，把有关内网的部分删掉就可以啦，例如跳板机就是有外网的服务器。

```shell
#!/usr/bin/expect
set host [lindex $argv 0]

set TERMSERV 跳板机IP
set USER 跳板机用户名
set PASSWORD 跳板机密码
set UATUN 内网服务器用户名
set UATPWD 内网服务器密码

# 登录跳板机
spawn ssh -l $USER $TERMSERV
expect {
        "yes/no" {send "yes\r";exp_continue;}
         "*password:*" { send "$PASSWORD\r" }
        }
# 登录内网
expect "*跳板机用户名@*" {send "ssh -l $UATUN $host\r"}
expect {
        "yes/no" {send "yes\r";exp_continue;}
        "*password:*" { send "$UATPWD\r" }
        }
interact
```
把上面的脚本保存成一个文件，比如 login_inner ，保存在你指定一个文件夹下，我文件的路径是 /Users/wangyongzhi/.ssh/login_inner，wangyongzhi 是我本机用户名。

别忘了加上可执行权限，方式是 chmod -R a+x /Users/wangyongzhi/.ssh/login_inner，

不然等下执行的时候会报

permission denied: /Users/wangyongzhi/.ssh/login_inner

解释以上 shell 脚本都是什么意思
#!/usr/bin/expect    
注意：这一行必须在脚本的第一行，告诉操作系统脚本里的代码使用哪一个 shell 来执行

set host [lindex $argv 0]  
这一行是设置一个变量的意思，变量名随便起，尽量有意义，后面表示的是传入的参数，0 表示第一个参数，后面会用到。
下面几个 set 类似，跳板机用户名密码一般都一样，可以放到这个脚本里面，内网服务器的可以选择性的放，也可以把他们放到参数里面依次传进来，本文只拿一个做例子。

spawn ssh -l $USER $TERMSERV
spawn 是 expect 环境的内部命令，它主要的功能是给 ssh 运行进程加个壳，用来传递交互指令。

expect {
        "yes/no" {send "yes\r";exp_continue;}
         "*password:*" { send "$PASSWORD\r" }
        }
expect 也是 expect 环境的一个内部命令。判断上一个指令输入之后的得到输出结果是否包含""双引号里的字符串，比如后面的"*password:*"，表示上一个输出结果包含password:
*通配符表示前后可以是任意字符。
如果是第一次登录，会出现"yes/no"的字符串，就发送（send）指令"yes\r"，然后继续（exp_continue）。
下面的类似。

interact：执行完成后保持交互状态，把控制权交给控制台。
iTerm2 配置
解释完上面的脚本，下面就要配置 iTerm2 了。

在 preferences->profiles 里面选左下角的 + 号增加一条，配置如下图所示，

Name: 随便起，尽量有意义，你自己知道是哪台服务器
Tags: 标签，可写可不写，服务器多的话建议设置一个
send text at start: /Users/wangyongzhi/.ssh/login_inner 内网服务器IP
也可以写成 ~/.ssh/login_inner 内网服务器IP
内网服务器IP就是上面shell脚本里面的参数,跟前面的文件有空格哈
好了，到此为止，你就完成了所有的准备工作，这时候再登录服务器，只需要三步就可以了
1、打开 iTerm2
2、快捷键 Command + o 打开如下图所示的Profiles

3、选中你要进入的服务器名字，就可以进入啦