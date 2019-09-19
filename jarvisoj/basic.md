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
