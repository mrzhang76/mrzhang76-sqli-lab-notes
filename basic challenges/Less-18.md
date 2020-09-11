# Less-18 **POST Header Injection- Error Based- string**
TAG：POST型注入、HTTP头注入、报错注入  
## 漏洞利用  
根据此less名的提示，此次要进行http头注入，使用弱密码```admin admin```登陆后，发现页面会回显User-Agent的内容，那就可以在http头中的User-Agent中注入恶意语句  
![less18_1](images\less18_1.png)  

payload:```' and updatexml(1,concat(0x7e,(select @@version),0x7e),1) and '1'='1```  
使用burpsuite工具修改U-A写入恶意语句  
![less18_2](images\less18_2.png)  

成功的写入了恶意语句并获得了数据库信息  
![less18_3](images\less18_3.png)  

## 源码分析  
### less18源码： 
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-18 Header Injection- Error Based- string</title>
</head>

<body bgcolor="#000000">

<div style=" margin-top:20px;color:#FFF; font-size:24px; text-align:center"> Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br></div>
<div  align="center" style="margin:20px 0px 0px 510px;border:20px; background-color:#0CF; text-align:center;width:400px; height:150px;">
<div style="padding-top:10px; font-size:15px;">
 

<!--Form to post the contents -->
<form action="" name="form1" method="post">

  <div style="margin-top:15px; height:30px;">Username : &nbsp;&nbsp;&nbsp;
    <input type="text"  name="uname" value=""/>  </div>
  
  <div> Password : &nbsp; &nbsp;
    <input type="text" name="passwd" value=""/></div></br>
    <div style=" margin-top:9px;margin-left:90px;"><input type="submit" name="submit" value="Submit" /></div>
</form>
</div>
</div>
<div style=" margin-top:10px;color:#FFF; font-size:23px; text-align:center">
<font size="3" color="#FFFF00">



<?php
//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");
error_reporting(0);
	
function check_input($value)
	{
	if(!empty($value))
		{
		// truncation (see comments)
		$value = substr($value,0,20);
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



	$uagent = $_SERVER['HTTP_USER_AGENT'];
	$IP = $_SERVER['REMOTE_ADDR'];
	echo "<br>";
	echo 'Your IP ADDRESS is: ' .$IP;
	echo "<br>";
	//echo 'Your User Agent is: ' .$uagent;
// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))

	{
	$uname = check_input($_POST['uname']);
	$passwd = check_input($_POST['passwd']);
	
	/*
	echo 'Your Your User name:'. $uname;
	echo "<br>";
	echo 'Your Password:'. $passwd;
	echo "<br>";
	echo 'Your User Agent String:'. $uagent;
	echo "<br>";
	echo 'Your User Agent String:'. $IP;
	*/

	//logging the connection parameters to a file for analysis.	
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Agent:'.$uname."\n");
	
	fclose($fp);
	
	
	
	$sql="SELECT  users.username, users.password FROM users WHERE users.username=$uname and users.password=$passwd ORDER BY users.id DESC LIMIT 0,1";
	$result1 = mysql_query($sql);
	$row1 = mysql_fetch_array($result1);
		if($row1)
			{
			echo '<font color= "#FFFF00" font size = 3 >';
			$insert="INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES ('$uagent', '$IP', $uname)";
			mysql_query($insert);
			//echo 'Your IP ADDRESS is: ' .$IP;
			echo "</font>";
			//echo "<br>";
			echo '<font color= "#0000ff" font size = 3 >';			
			echo 'Your User Agent is: ' .$uagent;
			echo "</font>";
			echo "<br>";
			print_r(mysql_error());			
			echo "<br><br>";
			echo '<img src="../images/flag.jpg"  />';
			echo "<br>";
			
			}
		else
			{
			echo '<font color= "#0000ff" font size="3">';
			//echo "Try again looser";
			print_r(mysql_error());
			echo "</br>";			
			echo "</br>";
			echo '<img src="../images/slap.jpg"   />';	
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
用户输入的登陆信息都被过滤函数过滤，无法输入恶意语句  
```
	$uname = check_input($_POST['uname']);
	$passwd = check_input($_POST['passwd']);
```
通过分析页面代码可知，在登陆成功后，页面会将用户的cookie信息写入数据库以便下次直接访问
```
$row1 = mysql_fetch_array($result1);
	if($row1){
		echo '<font color= "#FFFF00" font size = 3 >';
        //向数据库写入UA信息
		$insert="INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES ('$uagent', '$IP', $uname)";
		mysql_query($insert);
```
但是写入的UA信息并没有被过滤器过滤，存在恶意注入风险  
同时页面提供了数据库报错信息的回显点，这使得通过UA进行的报错注入成为可能  

