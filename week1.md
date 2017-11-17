
##### 1. 什么是后台进程，正在运行的进程如果放入到后台?


```
前台作业：占据着一个终端
后台作业：作业执行时不占据终端，作业启动后就释放终端

非守护进程类的程序，启动以后都在前台工作
      如果已经启动：前台-->后台。ctrl+z把前台作业送往后台，作业被”停止“
	  如果尚未启动：COMMAND &

	  退出当前会话，作业也会终止，因为作业与当前终端相关，如果把作业送往后台后，不期望作业随终止结束而停止
            nohup COMMAND &
            
        nohup作用
            1 阻止SIGHUP信号发到这个进程。
		    2 关闭标准输入。该进程不再能够接收任何输入，即使运行在前台。
			3 重定向标准输出和标准错误到文件nohup.out。
	          nohup COMMAND >/tmp/cmd.out 2>&1
	    
	   如何让送往后台的作业继续执行：
	    fg [[%]作业号码]：将作业调回前台继续进行
	    bg [[%]作业号码]：让作业在后台继续进行
	        默认的为最后一个进入后台的任务
       kill %作业号码：终止作业	   
	   查看作业号：
	       jobs
```




##### 2.如何查看当前安装的软件包版本，如何安装/升级指定版本的包?



```
#查看版本

apt-show-versions 
apt-cache show <<package name>>
dpkg -s <<package name>>
dpkg -l <<package name>>

#安装/升级指定版本的包
apt-get install <<package name>>=<<version>>

#查看当前源仓库中软件包所有版本
apt-cache madison <<package name>>
apt-cache policy <<package name>>
aptitude versions <<package name>>

#删除清理
apt-get –purge remove
删除已安装包（不保留配置文件)。
如软件包a，依赖软件包b，则执行该命令会删除a，而且不保留配置文件

apt-get  remove
删除已安装的软件包（保留配置文件）。
如软件包a，依赖软件包b，则执行该命令会删除a，且保留配置文件

apt-get  autoremove 
删除为了满足依赖而安装的，但现在不再需要的软件包（包括已安装包）。
如软件包a，依赖软件包b，则执行该命令会同时删除软件包a,b

apt-get autoclean
APT安装Package时会将 *.deb 放在 /var/cache/apt/archives/中.
apt-get autoclean 只会删除 /var/cache/apt/archives/ 已经过期的deb.

apt-get clean
使用 apt-get clean 会将 /var/cache/apt/archives/ 的 所有 deb 删掉.
类似于 rm /var/cache/apt/archives/*.deb

如果是彻底卸载软件，推荐使用apt-get –purge remove
```


##### 3. 如何查看当前机器的流量，如何查看每个进程占用的io?

```
#查看机器流量

sar –n DEV  INTERVAL COUNTS -p 
ifstat //CentOS 自带

#查看所有占用io的进程
pidstat -d INTERVAL COUNTS
#查看指定进程占用的io
pidstat -d -p PID  INTERVAL COUNTS

```

##### 4.配置一个rsync server，允许1. 1. 1. 1. 2. 2. 2. 2通过test模块进行传输/home/test/rsync下的文件

```
#debian下
vim /etc/default/rsync
RSYNC_ENABLE=false

cp /usr/share/doc/rsync/examples/rsyncd.conf /etc/
vim /etc/rsyncd.conf

[test]
comment= test module
path=/home/test/rsync
hosts allow = 1.1.1.1 2.2.2.2

```


##### 5. logrotate是做什么用的？如果不使用该工具，改用手动处理，需要哪些步骤？
```
日志管理工具: 日志切割、压缩、删除等

#方案1:重命名当前使用的日志文件，创建新的日志文件。

打开一个文件以后，系统以inode号码来识别这个文件，不再考虑文件名。

1重命名程序当前正在输出日志的文件。因为重命名只会修改目录文件的内容，而进程依赖inode操作文件，所以不会影响程序继续输出日志。

2 创建新的日志文件，文件名和原来日志文件一样。虽然新的日志文件和原来日志文件的名字一样，但是inode编号不一样，所以程序输出的日志还是往原日志文件输出。

3 通过某些方式通知程序，重新打开日志文件。程序重新打开日志文件，靠的是文件路径而不是inode编号，所以打开的是新的日志文件。

通知程序重新打开日志的方式:kill掉重开进程 或IPC进程间通信
 

#方案2: copy一份正在输出的日志 再清空(trucate)原来的日志。

1 拷贝程序当前正在输出的日志文件，保存文件名为滚动结果文件名。这期间程序照常输出日志到原来的文件中，原来的文件名也没有变。

2 清空程序正在输出的日志文件。清空后程序输出的日志还是输出到这个日志文件中，因为清空文件只是把文件的内容删除了，文件的inode编号并没有发生变化，变化的是元信息中文件内容的信息。

结果上看，旧的日志内容存在滚动的文件里，新的日志输出到空的文件里。实现了日志的滚动。

Q:如何保证之前的文件内容被删除后还是从文件开头写入日志 而不是大片空白后写日志

A:程序大多以O_APPEND的方式打开文件，O_APPEND标志使得文件每次执行写操作时，文件表项中的当前文件偏移量首先会被设置为inode的文件长度。这就使得每次写入的数据都追加到文件末尾。

```

##### 6. systemd是什么，跟sysvinit有什么区别?
```
Systemd:系统初始化与服务管理工具  

历史上，Linux 的启动一直采用init进程。

$ sudo /etc/init.d/apache2 start  或 $ service apache2 start
这种方法有两个缺点。
一是启动时间长。init进程是串行启动，只有前一个进程启动完，才会启动下一个进程。
二是启动脚本复杂。init进程只是执行启动脚本，不管其他事情。脚本需要自己处理各种情况，编写复杂。

Systemd:
    尽可能启动更少进程；尽可能将更多进程并行启动。
    尽可能减少对shell脚本的依赖。
传统sysvinit使用inittab来决定运行哪些shell脚本，大量使用shell脚本被认为是效率低下无法并行的原因。

Systemd 的核心是单元（unit），它是一些存有关于服务（service）（在运行在后台的程序）、设备、挂载点、和操作系统其他方面信息的配置文件。



```
##### 7. git vs svn ，有什么相似？有什么区别？
```
都是版本控制工具
git 分布式 可离线 分支管理 快
svn 集中式 需联网
```
##### 8. git 未commit的修改如何回滚，已经commit的如何回
```
#未commit的修改回滚
1 工作区修改 未添加至暂存区
git checkout -- file

2 工作区修改 已添加至暂存区
git reset HEAD file  //把暂存区的修改撤销掉（unstage），重新放回工作

#已经commit回滚:
git reset --hard <commitId>


```
##### 9. git merge出现冲突，如何解决

```
git merge 时会先试图合并，无法自动解决冲突时需人工解决
vim 编辑文件，文件中冲突的部分会以"<<<<" ">>>>" "====="标记出不同分支的内容 编辑一致后

git add file.txt
git commit -m "fix conflct " file.txt

```
##### 10. 配置rsyslog，将本地文件 /var/log/messages 传输到 10. 2. 2. 1 机器的 10001端口，使用tcp协议
```
vim /etc/rsyslog.conf


module(load="imfile" PollingInterval="5") 

input(type="imfile"
      File="/var/log/messages"
      Tag="messages"
      ruleset="messages")

ruleset(name="messages"){
    action( 
        type="omfwd" Target="10.2.2.1" Port="10001" Protocol="tcp" 
    )
}



```
##### 11. supervisor通过什么信号来停止进程，如果该信号被进程忽略会怎么？


```
stopsignal 可以是 TERM, HUP, INT, QUIT, KILL  默认是TERM

若超过stopwaitsecs未收到回应 supervisord 将尝试使用 SIGKILL 终止目标进程

```



##### 12. 修改了supervisor配置文件之后，如何在尽量不影响它管理的进程的前提下生效新的配置文件
```
1 supervisorctl  reread
2 supervisorctl update //先用 update 命令让 supervisor 重新读取配置，然后再启动
```


##### 13. 编写一个puppet模块，管理rsync，需要以下功能：
安装rsync包
设置配置文件
配置文件变更时，进程需要自动reload


```
mkdir -vp /etc/puppet/modules/rsync/{files,templates,manifests}
```
/etc/puppet/modules/rsync/manifests下

vim site.pp
```
class rsync{
        include rsync::params,rsync::config,rsync::service,rsync::install
}
```
vim install.pp

```
class rsync::install{
        package { ["rsync"]:
                ensure => installed,
        }
}
```

vim config.pp
```
class rsync::config{
        file {  "/etc/rsyncd.conf":
                source => "puppet:///modules/rsync/rsyncd.conf",
                require => Class["rsync::install"],
                notify => Class["rsync::service"],
        }
}
```

vim service.pp

```
class rsync::service{
        service { "rsync":
                ensure => running,
                hasstatus => true,
                hasrestart => true,
                enable => true,
                require => Class["rsync::config"],
        }
}

```
 

##### 14. 需要在/etc/hosts中添加一条记录，如何在不影响现有配置情况下完成这一工作 

```
echo "1.1.1.1 www.test.com" >> /etc/hosts   //不知是不是这个意思

```
