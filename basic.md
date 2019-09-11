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


### 关于USS Lab.
####题目

> USS的英文全称是什么，请全部小写并使用下划线连接_，并在外面加上PCTF{}之后提交

#### 思路

难道是考察搜索引擎使用能力？直接搜索USS有其他内容影响，搜索USS Lab的答案才是，感觉没有意思

#### flag: 
`PCTF{ubiquitous_system_security}`
