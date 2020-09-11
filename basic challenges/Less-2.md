# Less-2 **GET Error Based-Intiger**
TAG：GET注入、数字型注入  
## 漏洞利用
此less和less1相似，只是参数类型变成了数字型  
![less2_1](images\less2_1.png)  

简单的进行测试  
payload:```?id=1```
![less2_5](images\less2_6.png)  

payload:```?id=2-1```
![less2_2](images\less2_2.png)  

当参数为```?id=2-1```时返回了与```?id=1```相同的页面，证明此页面接受数字型参数
测试是否存在注入：  
payload:```?id=1 and 1=1```  
![less2_3](images\less2_3.png)  
payload:```?id=1 and 1=2```  
![less2_4](images\less2_4.png)  
这里不同的显示说明payload影响了代码的处理，可以确定存在注入  
探测字段数、回显点与进行数据库注入的操作及payload与less1相似，只需进行部分的修改  
查找回显点payload:  
```127.0.0.1/Less-1/?id=1 union select 1,2,3--+```  
![less2_4](images\less2_6.png)  
因为此时为数字型注入，不需要使用单引号进行闭合

## 源码分析
### less2源码
```<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-2 **Error Based- Intiger**</title>
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
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
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
	else
		{ 	
		echo "Please input the ID as parameter with numeric value";
		}

?>


</font> </div></br></br></br><center>
<img src="../images/Less-2.jpg" /></center>
</body>
</html>
```

### 漏洞点分析   
less2是通过GET方式获取传递到页面的参数id，但它的数据库操作语句与less1有所不同  
```
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
$result=mysql_query($sql);
```  
less2的数据库操作语句中，传递到页面的参数id并没有被单引号'所包裹，这说明它是数字型参数，相应的恶意语句也不用使用单引号包裹了  
除此之外，less2页面的漏洞原因和利用方式都与less1类似  