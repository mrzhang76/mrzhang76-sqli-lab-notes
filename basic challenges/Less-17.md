# Less-17 **POST Update Query- Error based - String**
TAG：POST型注入、二次注入、报错注入
## 漏洞利用  
![less17_1](images\less17_1.png)  
这个less打开就是密码修改，此处做出的修改一定会写入到数据库中，如果对于此处写入的数据没有进行检测或过来就可能导致恶意代码注入    
尝试在username中进行注入  

Payload:```1' or updatexml(1,concat(0x7e,(select(database())),0x7e),1)#```  
![less17_1](images\less17_2.png)  

从回显来看，页面对username输入进行了过滤，使用同样的注入语句在password栏测试  
Payload:```1' or updatexml(1,concat(0x7e,(select(database())),0x7e),1)#```
![less17_1](images\less17_3.png)  
成功获得数据库名信息，现在可以通过构造不同的payload在password栏进行注入了  

## 源码分析  
### less17源码： 
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-17 Update Query- Error based - String</title>
</head>

<body bgcolor="#000000">

<div style=" margin-top:20px;color:#FFF; font-size:24px; text-align:center"><font color="#FFFF00"> [PASSWORD RESET] </br></font>&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br></div>

<div  align="center" style="margin:20px 0px 0px 520px;border:20px; background-color:#0CF; text-align:center; width:400px; height:150px;">

<div style="padding-top:10px; font-size:15px;">
 

<!--Form to post the contents -->
<form action="" name="form1" method="post">

  <div style="margin-top:15px; height:30px;">User Name &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;: &nbsp;&nbsp;&nbsp;&nbsp;
    <input type="text"  name="uname" value=""/>  </div>
  
  <div> New Password : &nbsp; &nbsp;
    <input type="text" name="passwd" value=""/></div></br>
    <div style=" margin-top:9px;margin-left:90px;"><input type="submit" name="submit" value="Submit" /></div>
</form>
</div>
</div>
<div style=" margin-top:10px;color:#FFF; font-size:23px; text-align:center">
<font size="6" color="#FFFF00">



<?php
//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");
error_reporting(0);

function check_input($value)
	{
	if(!empty($value))
		{
		// truncation (see comments)
		$value = substr($value,0,15);
		}

		// Stripslashes if magic quotes enabled
		if (get_magic_quotes_gpc())
			{
			$value = stripslashes($value);
			}

		// Quote if not a number
		if (!ctype_digit($value))
			{
			$value = "'" . mysql_real_escape_string($value) . "'";
			}
		
	else
		{
		$value = intval($value);
		}
	return $value;
	}

// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))

{
//making sure uname is not injectable
$uname=check_input($_POST['uname']);  

$passwd=$_POST['passwd'];


//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'User Name:'.$uname."\n");
fwrite($fp,'New Password:'.$passwd."\n");
fclose($fp);


// connectivity 
@$sql="SELECT username, password FROM users WHERE username= $uname LIMIT 0,1";

$result=mysql_query($sql);
$row = mysql_fetch_array($result);
//echo $row;
	if($row)
	{
  		//echo '<font color= "#0000ff">';	
		$row1 = $row['username'];  	
		//echo 'Your Login name:'. $row1;
		$update="UPDATE users SET password = '$passwd' WHERE username='$row1'";
		mysql_query($update);
  		echo "<br>";
	
	
	
		if (mysql_error())
		{
			echo '<font color= "#FFFF00" font size = 3 >';
			print_r(mysql_error());
			echo "</br></br>";
			echo "</font>";
		}
		else
		{
			echo '<font color= "#FFFF00" font size = 3 >';
			//echo " You password has been successfully updated " ;		
			echo "<br>";
			echo "</font>";
		}
	
		echo '<img src="../images/flag1.jpg"   />';	
		//echo 'Your Password:' .$row['password'];
  		echo "</font>";
	


  	}
	else  
	{
		echo '<font size="4.5" color="#FFFF00">';
		//echo "Bug off you Silly Dumb hacker";
		echo "</br>";
		echo '<img src="../images/slap1.jpg"   />';
	
		echo "</font>";  
	}
}

?>


</font>
</div>
</body>
</html>
```

### 漏洞点分析   
```
...
if(isset($_POST['uname']) && isset($_POST['passwd'])){
    $uname=check_input($_POST['uname']);
    $passwd=$_POST['passwd'];
...
```
页面会将传输来的uname,passwd参数写入uname,passwd变量，在此过程中对uname进行了检测与过滤，使uname携带的恶意语句无效化了，此时仍可以在passwd中携带恶意语句来实现数据库注入  
```
...
function check_input($value)
	{
	if(!empty($value))
		{
		// truncation (see comments)
		$value = substr($value,0,15);
		}

		// Stripslashes if magic quotes enabled
		if (get_magic_quotes_gpc())
			{
			$value = stripslashes($value);
			}

		// Quote if not a number
		if (!ctype_digit($value))
			{
			$value = "'" . mysql_real_escape_string($value) . "'";
			}
		
	else
		{
		$value = intval($value);
		}
	return $value;
	}
...
```
页面中使用了如上的过滤函数来过滤恶意语句  
先将输入的变量限制在15字符以内，然后使用get_magic_quotes_gpc()函数判断是否为用户的输入添加的反斜杠转义，如果有就去除反斜杠，并对不是数字型的字符添加单引号  
但是这样的过滤器并没有被应用到passwd变量上，这使得恶意语句可以通过passwd进入执行过程  
页面还进行了数据库报错信息的输出，这就导致了二次注入的存在，使得数据库信息通过报错信息泄露  