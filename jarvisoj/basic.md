### 0x00 base64?
#### 题目
> GUYDIMZVGQ2DMN3CGRQTONJXGM3TINLGG42DGMZXGM3TINLGGY4DGNBXGYZTGNLGGY3DGNBWMU3WI===

#### 思路

没有思路，就看着三个=号感觉不是base64，用Python的base64库解码失败，提示padding错误，分次去掉1、2、3个等号都是报同样的错误，没有其他编码类似这样的经验，看别人的writeup知道是base32，然后就有了思路

```
In [18]: import base64

In [19]: base64.b32decode('GUYDIMZVGQ2DMN3CGRQTONJXGM3TINLGG42DGMZXGM3TINLGGY4DG
    ...: NBXGYZTGNLGGY3DGNBWMU3WI===')
Out[19]: b'504354467b4a7573745f743373745f683476335f66346e7d'

In [20]: bytes.fromhex('504354467b4a7573745f743373745f683476335f66346e7d')
Out[20]: b'PCTF{Just_t3st_h4v3_f4n}'
```

后面16进制转字符也不太友好，一下子没有想到，提交需要间隔30s

base32编码后来没有去学习，毕竟应该不常用

#### flag
`PCTF{Just_t3st_h4v3_f4n}`


### 0x01 关于USS Lab.
#### 题目
> USS的英文全称是什么，请全部小写并使用下划线连接_，并在外面加上PCTF{}之后提交

#### 思路

难道是考察搜索引擎使用能力？直接搜索USS有其他内容影响，搜索USS Lab的答案才是，感觉没有意思

#### flag 
`PCTF{ubiquitous_system_security}`

### 0x02 veryeasy
#### 题目
> 使用基本命令获取flag
> [veryeasy.d944f0e9f8d5fe5b358930023da97d1a](https://dn.jarvisoj.com/challengefiles/veryeasy.d944f0e9f8d5fe5b358930023da97d1a)

#### 思路
下载文件用vim打开是乱码，file命令看是可执行文件，加执行权限执行无效，用Python读文件，然后试各种编码失败，看writeup知道strings命令，之前不知道这个命令，还可以看编译后可执行文件文件内容，多了解了strings命令的知识点

#### flag
`PCTF{strings_i5_3asy_isnt_i7}`

### 0x03 段子
#### 题目
> 程序猿圈子里有个非常著名的段子：
> 手持两把锟斤拷，口中疾呼烫烫烫。
> 请提交其中"锟斤拷"的十六进制编码。(大写)
> FLAG: PCTF{你的答案}

#### 思路
上Python3的encode，不对，考虑编码问题，用gbk，可以

```
>>> '锟斤拷'.encode('gbk')
b'\xef\xbf\xbd\xef\xbf\xbd'
```

提交时候格式和加不加`\x`等问题非常恼人

#### flag
`PCTF{EFBFBDEFBFBD}`

### 0x04 手贱
#### 题目
> 某天A君的网站被日，管理员密码被改，死活登不上，去数据库一看，啥，这密码md5不是和原来一样吗？为啥登不上咧？
> d78b6f302l25cdc811adfe8d4e7c9fd34
> 请提交PCTF{原来的管理员密码}

#### 思路
网页上看不出来，复制到命令行，302l25中是小写l，md5不含l，数位数多了1位，去掉后去cmd5.com上找原文，是hack

#### flag
`PCTF{hack}`

### 0x05 美丽的实验室logo
#### 题目
> 出题人丢下个logo就走了，大家自己看着办吧
> [logo.jpg.8244d3d060e9806accc508ec689fabfb](https://dn.jarvisoj.com/challengefiles/logo.jpg.8244d3d060e9806accc508ec689fabfb)

#### 思路
16进制打开没有任何发现，StegSolve打开各种按钮试试，frame browser可以看到两张图片，其中一张有flag，看来是两张图片拼接而成，对图片的格式认识不够

#### flag
`PCTF{You_are_R3ally_Car3ful}`

### 0x06 veryeasyRSA
#### 题目
> 已知RSA公钥生成参数：
> p = 3487583947589437589237958723892346254777 q = 8767867843568934765983476584376578389
> e = 65537
> 求d = 
> 请提交PCTF{d}
> Hint1: 有好多小伙伴问d提交什么格式的，现在明确一下，提交十进制的d
#### 思路
已知p和q，知道n=(p-1)*(q-1)，已知e，求e关于n的逆元

翻书找逆元求法

```
def exgcd(a, b):
    if not b:
        return a, 1, 0
    d, y, x = exgcd(b, a % b)
    y -= a // b * x
    return d, x, y

def get_inverse(a, p):
    d, x, y = exgcd(a, p)
    return (x + p) % p

p = 3487583947589437589237958723892346254777
q = 8767867843568934765983476584376578389
e = 65537
n = (p - 1) * (q - 1)
d = get_inverse(e, n)
print(d)
```

#### flag
`PCTF{19178568796155560423675975774142829153827883709027717723363077606260717434369}`

### 0x07 神秘的文件
#### 题目
> 出题人太懒，还是就丢了个文件就走了，你能发现里面的秘密吗？
> (haha.f38a74f55b4e193561d1b707211cf7eb)[https://dn.jarvisoj.com/challengefiles/haha.f38a74f55b4e193561d1b707211cf7eb]

#### 思路
strings可以看到/mnt/temp，尝试挂载，获取大量数字编号文件，每个文件有1个字母

```
s = ''
for i in range(254):
    with open(str(i)) as f:
        s += f.read()
print(s)
```
得到
> Haha ext2 file system is easy, and I know you can easily decompress of it and find the content in it.But the content is spilted in pieces can you make the pieces together. Now this is the flag PCTF{P13c3_7oghter_i7}. The rest is up to you. Cheer up, boy.

#### flag
PCTF{P13c3_7oghter_i7}


### 0x08 公倍数
#### 题目
> 请计算1000000000以内3或5的倍数之和。
> 如：10以内这样的数有3,5,6,9，和是23
> 请提交PCTF{你的答案}

#### 思路
高斯求和，数字为x，总数为n

(x+n//x*x)/2*(n//x)

优化成

(1+n//x)*x*(n//x)/2

```
>>> n = 1000000000 - 1
>>> f = lambda x: (1+n//x)*x*(n//x)/2
>>> f(3) + f(5) - f(15)
```

#### flag
PCTF{233333333166666668}


### 0x09 Easy Crackme
#### 题目
> 都说逆向挺难的，但是这题挺容易的，反正我不回，大家来挑战一下吧~~:)
> [easycrackme.6dbc7c78c9bb25f724cd55c0e1412617](https://dn.jarvisoj.com/challengefiles/easycrackme.6dbc7c78c9bb25f724cd55c0e1412617)

#### 思路
反正我不会，先不写

#### flag
PCTF{}


### 0x10 Secret
#### 题目
> 传说中的签到题
> 题目入口：http://web.jarvisoj.com:32776/
> Hint1: 提交格式PCTF{你发现的秘密}

#### 思路
访问，返回包体无内容，头部有secret头部，不知道怎么办，后来看到其他writeup上写这个头部就是flag...

#### flag
PCTF{Welcome_to_phrackCTF_2016}


### 0x11 爱吃培根的出题人
#### 题目
> 听说你也喜欢吃培根？那我们一起来欣赏一段培根的介绍吧：
> bacoN is one of aMerICa'S sWEethEartS. it's A dARlinG, SuCCulEnt fOoD tHAt PaIRs FlawLE
> 什么，不知道要干什么？上面这段巨丑无比的文字，为什么会有大小写呢？你能发现其中的玄机吗？
> 提交格式：PCTF{你发现的玄机}

#### 思路
培根密码，可以通过http://ctf.ssleye.com/baconian.html解密，这样的题也感觉没什么意义

aaaab aaaaa aaaba abbab abbaa abaaa baaab abbaa abbab baaba aabab abbab abbab aaabb

根据https://en.wikipedia.org/wiki/Bacon%27s_cipher表里的映射得到答案

#### flag
PCTF{baconisnotfood}


### 0x12 Easy RSA
#### 题目
> 还记得veryeasy RSA吗？是不是不难？那继续来看看这题吧，这题也不难。
> 已知一段RSA加密的信息为：0xdc2eeeb2782c且已知加密所用的公钥：
> (N=322831561921859 e = 23)
> 请解密出明文，提交时请将数字转化为ascii码提交
> 比如你解出的明文是0x6162，那么请提交字符串ab
> 提交格式:PCTF{明文字符串}

#### 思路
看别人的writeup发现kali有质因数分解的工具

```
$ factor 322831561921859
322831561921859: 13574881 23781539
```

然后用欧几里得求逆元得到d

```
def exgcd(a, b):
    if not b:
        return a, 1, 0
    d, y, x = exgcd(b, a % b)
    y -= a // b * x
    return d, x, y

def get_inverse(a, p):
    d, x, y = exgcd(a, p)
    return (x + p) % p

e = 23
n = (13574881 - 1) * (23781539 - 1)
d = get_inverse(e, n)
```

d = 42108459725927

用Python的`pow(m, d)/n`运行太慢，不知道多久能求出来，所以又回去翻书找优化方法

```
n = 322831561921859
d = 42108459725927
m = 0xdc2eeeb2782c

def fpow(a, b, c):
    a = a % c
    r = 1
    while b:
        if b & 1:
            r = r * a % c
        a = a * a % c
        b = b >> 1
    return r

plain = fpow(m, d, n)
print(bytes.fromhex(format(plain, 'x')).decode('utf-8'))
```

#### flag
PCTF{3a5Y}

### 0x13 ROPGadget
#### 题目
> 都说学好汇编是学习PWN的基础，以下有一段ROPGadget的汇编指令序列，请提交其十六进制机器码(大写，不要有空格)
> XCHG EAX,ESP
> RET
> MOV ECX,[EAX]
> MOV [EDX],ECX
> POP EBX
> RET
> 提交格式：PCTF{你的答案}

#### 思路
反正我不会，先不写

#### flag
PCTF{}


### 0x14 取证
#### 题目
> 有一款取证神器如下图所示，可以从内存dump里分析出TureCrypt的密钥，你能找出这款软件的名字吗？名称请全部小写。
> Conatiner: \??\F:\level2
> Hidden Volumn: Yes
> Removable: No
> Read Only: No
> ...

#### 思路
百度：volatility

真没用过，反正真正需要的时候也能百度到吧

#### flag
PCTF{volatility}


### 0x15 熟悉的声音
#### 题目
> 两种不同的元素，如果是声音的话，听起来是不是很熟悉呢，据说前不久神盾局某位特工领便当了大家都很惋惜哦
> XYYY YXXX XYXX XXY XYY X XYY YX YYXX
> 请提交PCTF{你的答案}

#### 思路
摩尔斯电码加凯撒密码

摩尔斯电码解出来有两种可能

JBLUWEWNZ和BJYGDTDA#

JBLUWEWNZ位移一位后得到PHRACKCTF

其实蛮难的，没有别人writeup的话凯撒密码位移了也不知道是不是对的，只能一个个试过去，很耗费精力

#### flag
PCTF{PHRACKCTF}


### 0x16 Baby's Crack
#### 题目
> 既然是逆向题，我废话就不多说了，自己看着办吧。
> [babyscrack.rar.831d813059fb7a7eb9cd0e9904726977](https://dn.jarvisoj.com/challengefiles/babyscrack.rar.831d813059fb7a7eb9cd0e9904726977)

#### 思路
反正我不会，先不写

#### flag
PCTF{}


### 0x17 Help!!!
#### 题目
> 出题人硬盘上找到一个神秘的压缩包，里面有个word文档，可是好像加密了呢~让我们一起分析一下吧！
> [word.zip.a5465b18cb5d7d617c861dee463fe58b](https://dn.jarvisoj.com/challengefiles/word.zip.a5465b18cb5d7d617c861dee463fe58b)

#### 思路
首先打开word，发现一张图片，用二进制打开没发现什么问题，可能需要隐写分析，先不看

知道word就是zip文件，改后缀为zip打开，找一些文件看看，结果media目录下发现第二张图片，打开有flag

#### flag
PCTF{You_Know_moR3_4boUt_woRd}

### 0x18 Shellcode
#### 题目
> 作为一个黑客，怎么能不会使用shellcode?
> 这里给你一段shellcode，你能正确使用并最后得到flag吗？
> [shellcode.06f28b9c8f53b0e86572dbc9ed3346bc](https://dn.jarvisoj.com/challengefiles/shellcode.06f28b9c8f53b0e86572dbc9ed3346bc)

#### 思路
不会做，看别人用某个工具执行一下，不清楚原理所以暂时不做

#### flag
PCTF{}


### 0x19 A Piece of Cake
#### 题目
> nit yqmg mqrqn bxw mtjtm nq rqni fiklvbxu mqrqnl xwg dvmnzxu lqjnyxmt xatwnl, rzn nit uxnntm xmt zlzxuuk mtjtmmtg nq xl rqnl. nitmt vl wq bqwltwlzl qw yivbi exbivwtl pzxuvjk xl mqrqnl rzn nitmt vl atwtmxu xamttetwn xeqwa tsftmnl, xwg nit fzruvb, nixn mqrqnl ntwg nq gq lqet qm xuu qj nit jquuqyvwa: xbbtfn tutbnmqwvb fmqamxeevwa, fmqbtll gxnx qm fiklvbxu ftmbtfnvqwl tutbnmqwvbxuuk, qftmxnt xznqwqeqzluk nq lqet gtamtt, eqdt xmqzwg, qftmxnt fiklvbxu fxmnl qj vnltuj qm fiklvbxu fmqbtlltl, ltwlt xwg exwvfzuxnt nitvm twdvmqwetwn, xwg tsivrvn vwntuuvatwn rtixdvqm - tlftbvxuuk rtixdvqm yivbi evevbl izexwl qm qnitm xwvexul. juxa vl lzrlnvnzntfxllvldtmktxlkkqzaqnvn. buqltuk mtuxntg nq nit bqwbtfn qj x mqrqn vl nit jvtug qj lkwnitnvb rvquqak, yivbi lnzgvtl twnvnvtl yiqlt wxnzmt vl eqmt bqefxmxrut nq rtvwal nixw nq exbivwtl.
> 提交格式：PCTF{flag}

#### 思路
猜测是词频统计，nit=>the，vl=>is, x=>a，估计这种题目的词频统计结果和每个字母使用频率应该是刚好对应的，本来想统计一下，结果网上一搜索，有直接可以给答案的网站https://quipqiup.com/，直接输入文本，有一些统计不对，把nit=the vl=is, x=a mqrqn=robot输入就好了，搜索flag关键字，得到结果

#### flag
PCTF{substitutepassisveryeasyyougotit}


### 0x20 -.-字符串
#### 题目
> 请选手观察以下密文并转换成flag形式
> ..-. .-.. .- --. ..... ..--- ..--- ----- .---- ---.. -.. -.... -.... ..... ...-- ---.. --... -.. .---- -.. .- ----. ...-- .---- ---.. .---- ..--- -... --... --... --... -.... ...-- ....- .---- -----
> flag形式为32位大写md5
> 题目来源：CFF2016

#### 思路
看起来是摩尔斯电码，直接使用http://ctf.ssleye.com/morse.html，得到flag

#### flag
522018D665387D1DA931812B77763410

这道题flag不用加PCTF，感觉非常不友好


### 0x21 德军的密码
#### 题目
> 已知将一个flag以一种加密形式为使用密钥进行加密，使用密钥WELCOMETOCFF加密后密文为 000000000000000000000000000000000000000000000000000101110000110001000000101000000001 请分析出flag。Flag为12位大写字母
> 题目来源：CFF2016

#### 思路
密文长度为84，84/12=7，密钥长度为12，猜测明文和密文异或，因为是字母，七位就够用了

```
In [59]: c = "0000000000000000000000000000000000000000000000000001011100001100
    ...: 01000000101000000001"

In [60]: c1 = [i for i in 'WELCOMETOCFF']

In [61]: c2 = [c[i*7:i*7+7] for i in range(12)]

In [62]: c1
Out[62]: ['W', 'E', 'L', 'C', 'O', 'M', 'E', 'T', 'O', 'C', 'F', 'F']

In [63]: c2
Out[63]:
['0000000',
 '0000000',
 '0000000',
 '0000000',
 '0000000',
 '0000000',
 '0000000',
 '0010111',
 '0000110',
 '0010000',
 '0010100',
 '0000001']

In [65]: list(map(lambda x, y: chr(ord(x) ^ int(y, 2)), c1, c2))
Out[65]: ['W', 'E', 'L', 'C', 'O', 'M', 'E', 'C', 'I', 'S', 'R', 'G']
```

提交成功

#### flag
WELCOMECISRG


### 0x22 握手包
#### 题目
> 给你握手包，flag是Flag_is_here这个AP的密码，自己看着办吧。
> 提交格式：flag{WIFI密码}
> [wifi.cap.d4e4d22bc8fe925bf0ccb9382056ce8e](https://dn.jarvisoj.com/challengefiles/wifi.cap.d4e4d22bc8fe925bf0ccb9382056ce8e)

#### 思路
wireshark打开，找到所有SSID为Flag_is_here的包，发现两个包的长度不为277，不同，观察，发现不了问题

看不出来，对WIFI包没有了解

看网上使用工具aircrack-ng破解，包的格式都看不明白，目前肯定不知道破解的原理，先记着

#### flag
flag{11223344}
