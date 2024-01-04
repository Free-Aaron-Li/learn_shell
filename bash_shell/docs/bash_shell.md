# 一、Shell的基本功能

一般用于脚本自动化、系统管理及配置。

shell命令通过空格分隔，如果在命令中需要使用到空格，则需要进行转义操作“\ ”；如果希望执行多条语句，则需要用“;”进行分隔。

对于一个程序来讲，一般来讲为：

```text
执行路径或相对路径下程序 > 别名 > Bash内部命令 > 环境变量定义下第一个对应命令
```

在开发过程中，我们往往会使用到一些快捷键以帮助开发，例如：（^表示ctrl,例如`^a`表示`ctrl+a`)

- ^a，将光标移动至行首
- ^e，将光标移动至行尾
- ^c，强制终止当前命令
- ^l，清屏（与直接使用clear命令不同）
- ^u，删除光标之前命令
- ^k，删除光标之后命令
- ^y，粘贴上述两个命令内容
- ^r，搜索历史命令
- ^d，退出当前终端
- ^z，暂停（如程序）
- ^s，暂停屏幕输出
- ^q，恢复屏幕输出

> 补充：
>
> bash采用emacs操作逻辑，所以会发现许多快捷键:)，如果想要使用vi操作逻辑，可以在~/.bashrc下插入`set -o vi`，如果使用的是`fish shell`则插入`set fish_plugins autojump vi-mode`。

---

在正式学习之前，先了解以下基础shell知识点：

- 标准输入/输出
	- 标准输入
		- 一般通过键盘实现，其设备文件名为：/dev/stdin
		- 文件描述符为0
	- 标准输出
		- 一般显示在显示器上，设备文件名为：/dev/stdout
		- 文件描述符为1
	- 标准错误输出
		- 一般显示在显示器上，文件设备名为：/dev/stderr
		- 文件描述符为2

在后续开发中，我们会经常使用到这三者。

不妨做个小练习：

```bash
~/Downloads> echo 'a' 1> a.txt
~/Downloads> echo 'b' 2> b.txt
b
~/Downloads> cat 0< a.txt
a
~/Downloads> cat 0< b.txt
```

> 补充：
>
> `~>Downloads`为笔者的终端提示符，您无需关心。请注意`>`后的命令。

在上面的练习中，笔者使用了两个程序：`echo`和`cat`。二者均是linux操作系统默认提供的，无需自行下载。

简单介绍二者功能：

- echo，打印数据或将数据写入某文件
- cat，打印某文件数据于终端

当然，二者还有许多功能和参数实现，详细可以通过使用`man`程序查看，例如：man echo

回到练习，这里我们使用到了`>`重定向符号，您无需担心，只需要记住该符号即可。

首先，我们`echo 'a' 1> a.txt`，意图将字符a输入至`a.txt`文件中，同理可得，`echo 'b' 2> b.txt`，也是意图将字符b输入至`b.txt`文件中。但是仔细观察发现，第二个命令直接将b打印在终端上了，这是为什么呢❓

还记得之前说的吗？标准输出的文件描述符为1，标准错误输出文件描述符为2。`echo 'b' 2> b.txt`命令中`echo`程序正确执行了，但是却让我们将标准错误输出到`b.txt`文件中，这显然是不正确的。所以程序仅执行`2>`的前半段，而并没有执行后半段。

事实上，当我们能输入`cat 0< a.txt`和`cat 0< b.txt`时，发现：`a.txt`文件中存在字符a，而`b.txt`文件并没有字符b。这里就可以证明第二条命令的后半段并没有被执行。

---

相信您对上述的第一知识点：`标准输入/输出`有一定了解了。那么我们接着来学习第二个关键点：逻辑符

如果您之前对编程有一定了解，相信一定了解`&&`和`||`逻辑符。

`&&`逻辑符表示“与”，例如：`a && b`，意思是当a正确时b才会被执行，a如果错误则b不会被执行。
`||`逻辑符表示“或”，同样例如：`a || b`，意思为当a正确时b不会被执行，a如果错误则b将会被执行

实例：

```bash
~/Downloads> lss && echo 'true' && echo 'false'
bash: lss：未找到命令
~/Downloads> lss && echo 'true' || echo 'false'
bash: lss：未找到命令
false
~/Downloads> lss || echo 'true' || echo 'false'
bash: lss：未找到命令
true
~/Downloads> lss || echo 'true' && echo 'false'
bash: lss：未找到命令
true
false
```

在上面的例子中，我们用到了两个逻辑符，分别组合就可以发现例子：

1. `lss`程序并不存在，所以无法输出true，同样由于前者是错误的，同样无法输出false
2. 与例子1的差别在于，第二个`&&`变为`||`，由于前者是错误的，根据`||`的特性，false语句能够被执行
3. 同样根据`||`特性，true能够被执行，但是由于`echo 'true'`的正确执行，最后一条语句则不能被执行，因为`||`逻辑符要求如果`a`正确，则`b`不能执行
4. `lss`程序不存在，所以第一条语句错误，那么第二条语句正确，true被输出；第二条语句的正确，让第三条语句能够被执行，false也被输出

---

在上面我们使用到了`||`和`&&`两个逻辑符，这两个符号在shell中属于特殊符号（因为其具有特定的功能），除此之外还有以下特殊符号[^2]：

- `''`，单引号。用于将单引号内的文本设置为字符串形式，其中的特殊符号转换为普通字符
- `""`，双引号。与单引号类似，但是对于`$`、` `` `、`\ `例外，三者仍然具有特殊意义（调用变量的值、引用命令、转移符）
- `$`，美元符号。用于调用变量值
- ` `` `，反引号。其中内容为系统命令，在Bash中会优先执行。与`$()`类似，但是推荐使用后者
- `$()`，与反引号类似。用于引用系统命令
- `#`，井号。在Bash中用于注释
- `\ `，反斜杠号。用于转义，将特殊字符转义为普通字符
- `()`，小括号。用于命令执行，括号内命令会在子Shell（重新开启一个子Shell）中执行
- `{}`，大括号。用于命令执行，括号内命令会在当前Shell中执行
- `[]`，中括号。用于变量测试

了解玩基本的特殊字符，我们测试一下：

```bash
$ a=data
$ echo '$a' # test ' '
$a
$ echo "a" # test " "
a
$ echo "$a" # test $
data
$ a=`date` # test ` `               
$ echo $a
2024年 01月 04日 星期四 17:23:20 CST
data
$ echo $a # test $
2024年 01月 04日 星期四 17:14:55 CST
$ a=$(date) # test $()
$ echo $a
2024年 01月 04日 星期四 17:15:09 CST
$ (date) # test ()
2024年 01月 04日 星期四 17:15:14 CST
$ a=`#date` # test #
$ echo $a
$ a=aa
$ b="$a"
$ echo $b
aa
$ b="\$a" # test \ 
$ echo $b
$a
```

> 补充
> 
> `()`和`{}`二者具有一定区别：
> 
> - `()`和`{}`都将一串命令放在括号内，并且命令之间用；号隔开。
> - `()`最后一个命令可以不用分号结尾
>    - `$ ( name=lm; echo $name )`
> - `{}`中最后一个命令要用分号结尾
>    - `$ { 空格 name=lm; echo $name; }`
> - `{}`中的第一个命令和左括号之间必须要有一个空格；
>    - `$ { 空格 name=lm; echo $name; }`
> - `()`里的各命令不必和括号有空格；
> - `()`和`{}`中，括号里面的某个命令的重定向只影响该命令，但括号外的重定向则影响到括号里的所有命令。

---

其实`()`和`{}`深究下来还有一些差别，这就需要知道**父子Shell**的概念

简单来说，当我们启动终端后，当前Shell我们成为“父Shell”，而当我们运行一个shell脚本时，当前Shell就会创建一个“子Shell”，当脚本介绍，其子Shell也就被销毁了。子Shell由父Shell派生而来，例如，当前Shell为Bash,那么其子Shell也为Bash。

示例：

```bash
[I] aaron@aaron-PC ~> ps -f  # 查看当前进程
UID          PID    PPID  C STIME TTY          TIME CMD
aaron      28901   10411  1 17:46 pts/3    00:00:00 /usr/bin/fish
aaron      28937   28901 99 17:46 pts/3    00:00:00 ps -f
[I] aaron@aaron-PC ~> bash   # 创建一个bash子Shell  
(base) aaron@aaron-PC:~$ ps -f # 查看当前进程
UID          PID    PPID  C STIME TTY          TIME CMD
aaron      28901   10411  0 17:46 pts/3    00:00:00 /usr/bin/fish
aaron      28947   28901  0 17:46 pts/3    00:00:00 bash
aaron      28965   28947 99 17:46 pts/3    00:00:00 ps -f
(base) aaron@aaron-PC:~$ bash # 再创建一个bash子Shell
(base) aaron@aaron-PC:~$ ps --forest # 查看Shell关系图
    PID TTY          TIME CMD
  28901 pts/3    00:00:00 fish # 父shell
  28947 pts/3    00:00:00  \_ bash # 子shell
  28970 pts/3    00:00:00      \_ bash # 子shell（孙shell）
  28991 pts/3    00:00:00          \_ ps # 当前ps进程
(base) aaron@aaron-PC:~$ exit # 退出
(base) aaron@aaron-PC:~$ exit # 退出
[I] aaron@aaron-PC ~> ps -f # 查看当前进程                                 
UID          PID    PPID  C STIME TTY          TIME CMD
aaron      28901   10411  0 17:46 pts/3    00:00:00 /usr/bin/fish
aaron      29009   28901 99 17:46 pts/3    00:00:00 ps -f
```

从上面示例就可以看出父子shell之间的关系。

回到`()`和`{}`之间的差别，在介绍特殊符号时简单说明了二者都会包含命令。深究以下：`()`执行括号内命令时，会创建一个子shell执行，而`{}`则是直接在当前shell中执行。

示例：

```bash
(base) aaron@aaron-PC:~$ a=aa
(base) aaron@aaron-PC:~$ ( a=bb; echo $a;) 
bb                                                                 
(base) aaron@aaron-PC:~$ echo $a                                   
aa                                                                 
(base) aaron@aaron-PC:~$ { a=bb; echo $a;}                         
bb                                                                 
(base) aaron@aaron-PC:~$ echo $a                                   
bb               
```

从上面的示例我们就可以看出，`()`会单独创建一个子shell，所以`a=bb`命令并在当前shell生效，而当在`{}`写上`a=bb`时，当前shell生效了。说明`{}`括号中命令运行在当前shell中。

> 注意：
> 
> 生成子shell的成本并不低且生成速度慢，创建嵌套子shell去处理命令进程性能低效。所以在开发中应该尽量少的使用`()`方式存放命令。

---

在前面我们了解父子进程时，不知道您是否注意到执行`ps -f`和`ps --forest`命令时，总是会产生一个有关`ps`的进程。为什么会产生呢？

原因是`ps`命令为**外部命令**。何谓外部命令？外部命令也叫做**文件系统命令**，简单来说就是存在于Bash shell之外的程序，并不是Bash shell内嵌的程序。通过这些命令存放在：`/usr/bin`、`/bin`、`sbin`或者`/usr/sbin`目录中，当然也有可能存在于`~/.local/bin`目录中。

当选择执行外部命令时，Bash shell就会创建一个子进程（也叫做衍生）来执行该程序。所以，当我们执行`ps`程序时才会在进程中看到该程序名。

在前面介绍，子进程的创建和使用对资源消耗是比较大的。所以执行外部命令通常开销较大。

与之相对的，便是那些存在与Bash shell中的命令，也成为**内建命令**。那么有那些内置命令呢？您可以查看[Linux Bash内置命令大全详细介绍](https://www.cnblogs.com/11hwu2/p/3724986.html)

> 如果想立即查看自己正在使用的命令是否属于内置命令，可以通过`type`程序查看：
> 
> ```bash
> $ type pwd
>  pwd 是 shell 内建
> $ type ps
>  ps 是 /usr/bin/ps
> ```
> 
> 有些命令既有内建命令，也有外部命令。那么便可以加上`-a`参数查看该命令的所有命令位置：
> 
> ```bash
> $ type pwd -a
>  pwd is a builtin                   
>  pwd is /usr/bin/pwd                  
>  pwd is /bin/pwd 
> ```
> 
> 与之类似的命令还有：
> - `which`，但是该程序仅能查询外部命令：
> - `whereis`，也是仅能查询外部命令，但是可以查找该命令的man手册

---

最后，我们以**别名**作为结尾吧。

别名可以说在shell的使用中属于相当频繁的了。因为这有效地解决了程序参数设置过长、程序较为复杂的情况。

例如：我希望能够通过笔记本的摄像头拍一张照片并保存为“jpg”格式，那么我可以输入下面的命令：

`ffmpeg -i /dev/video0 -frames 1  -r 1  -f image2 image.jpg`

但是这个命令似乎太长了，这个时候就可以使用别名方式：

```bash
$ alias Cheese='ffmpeg -i /dev/video0 -frames 1  -r 1  -f image2 image.jpg'
```

这样，当想要拍照时，输入`Cheese`命令即可。是不是很方便！

> 补充
> 
> 由于`alias`本身就是内建命令，其产生的别名也是内建命令，即仅在当前shell中有效。当换一个shell时，该别名便失效了。
> 
> 对此的解决思路是，将别名存放在shell的配置文件中，比如存放在Bash shell的配置文件`~/.bashrc`。

----

参考文献：

[基于deepin的shell编程-上](https://bbs.deepin.org/post/266587)

[Bash脚本进阶指南](https://linuxstory.gitbook.io/advanced-bash-scripting-guide-in-chinese/zheng-wen/part1)

---

索引：

[^1]：[在 Linux 上自定义 bash 命令提示符](https://zhuanlan.zhihu.com/p/50993989)

[^2]：[Shell基础——Bash的特殊字符](https://zhuanlan.zhihu.com/p/558775217)

