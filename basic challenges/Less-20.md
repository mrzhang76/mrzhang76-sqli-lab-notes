# Less-20 **POST Cookie Injection- Error Based- string**
TAG：POST注入、COOKIE注入
## 漏洞利用
存在```admin1 admin1```弱密码登陆，根据提示可以从cookie下手
![less20_1](images\less20_1.png)  

修改cookie中的value来实现注入  
![less20_2](images\less20_2.png)  

探测一下字段数：
payload:```uname='order by 4#```
![less20_3](images\less20_3.png)  

探测回显点:
payload:```uname='union select 1,2,3#```
![less20_4](images\less20_4.png)  

获取数据库名:
payload:```'union select database(),2,3#```
![less20_5](images\less20_5.png)  

## 源码分析
### less20源码：
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />

<title>Less-20 Cookie Injection- Error Based- string</title>
</head>

<body bgcolor="#000000">
<?php
//including the Mysql connect parameters.
	include("../sql-connections/sql-connect.php");
	error_reporting(0);
if(!isset($_COOKIE['uname']))
	{
	//including the Mysql connect parameters.
	include("../sql-connections/sql-connect.php");

	echo "<div style=' margin-top:20px;color:#FFF; font-size:24px; text-align:center'> Welcome&nbsp;&nbsp;&nbsp;<font color='#FF0000'> Dhakkan </font><br></div>";
	echo "<div  align='center' style='margin:20px 0px 0px 510px;border:20px; background-color:#0CF; text-align:center;width:400px; height:150px;'>";
	echo "<div style='padding-top:10px; font-size:15px;'>";
 

	echo "<!--Form to post the contents -->";
	echo '<form action=" " name="form1" method="post">';

	echo ' <div style="margin-top:15px; height:30px;">Username : &nbsp;&nbsp;&nbsp;';
	echo '   <input type="text"  name="uname" value=""/>  </div>';
  
	echo ' <div> Password : &nbsp; &nbsp; &nbsp;';
	echo '   <input type="text" name="passwd" value=""/></div></br>';	
	echo '   <div style=" margin-top:9px;margin-left:90px;"><input type="submit" name="submit" value="Submit" /></div>';

	echo '</form>';
	echo '</div>';
	echo '</div>';
	echo '<div style=" margin-top:10px;color:#FFF; font-size:23px; text-align:center">';
	echo '<font size="3" color="#FFFF00">';
	echo '<center><br><br><br>';
	echo '<img src="../images/Less-20.jpg" />';
	echo '</center>';




	
function check_input($value)
	{
	if(!empty($value))
		{
		$value = substr($value,0,20); // truncation (see comments)
		}
		if (get_magic_quotes_gpc())  // Stripslashes if magic quotes enabled
			{
			$value = stripslashes($value);
			}
		if (!ctype_digit($value))   	// Quote if not a number
			{
			$value = "'" . mysql_real_escape_string($value) . "'";
			}
	else
		{
		$value = intval($value);
		}
	return $value;
	}


	
	echo "<br>";
	echo "<br>";
	
	if(isset($_POST['uname']) && isset($_POST['passwd']))
		{
	
		$uname = check_input($_POST['uname']);
		$passwd = check_input($_POST['passwd']);
		
	

		
		$sql="SELECT  users.username, users.password FROM users WHERE users.username=$uname and users.password=$passwd ORDER BY users.id DESC LIMIT 0,1";
		$result1 = mysql_query($sql);
		$row1 = mysql_fetch_array($result1);
		$cookee = $row1['username'];
			if($row1)
				{
				echo '<font color= "#FFFF00" font size = 3 >';
				setcookie('uname', $cookee, time()+3600);	
				header ('Location: index.php');
				//echo "I LOVE YOU COOKIES";
				echo "</font>";
				echo '<font color= "#0000ff" font size = 3 >';			
				echo 'Your Cookie is: ' .$cookee;
				echo "</font>";
				echo "<br>";
				print_r(mysql_error());			
				echo "<br><br>";
				echo '<img src="../images/flag.jpg" />';
				echo "<br>";
				}
			else
				{
				echo '<font color= "#0000ff" font size="3">';
				//echo "Try again looser";
				print_r(mysql_error());
				echo "</br>";			
				echo "</br>";
				echo '<img src="../images/slap.jpg" />';	
				echo "</font>";  
				}
			}
		
			echo "</font>";  
	echo '</font>';
	echo '</div>';

}
else
{



	if(!isset($_POST['submit']))
		{
			
			$cookee = $_COOKIE['uname'];
			$format = 'D d M Y - H:i:s';
			$timestamp = time() + 3600;
			echo "<center>";
			echo '<br><br><br>';
			echo '<img src="../images/Less-20.jpg" />';
			echo "<br><br><b>";
			echo '<br><font color= "red" font size="4">';	
			echo "YOUR USER AGENT IS : ".$_SERVER['HTTP_USER_AGENT'];
			echo "</font><br>";	
			echo '<font color= "cyan" font size="4">';	
			echo "YOUR IP ADDRESS IS : ".$_SERVER['REMOTE_ADDR'];			
			echo "</font><br>";			
			echo '<font color= "#FFFF00" font size = 4 >';
			echo "DELETE YOUR COOKIE OR WAIT FOR IT TO EXPIRE <br>";
			echo '<font color= "orange" font size = 5 >';			
			echo "YOUR COOKIE : uname = $cookee and expires: " . date($format, $timestamp);
			
			
			echo "<br></font>";
			$sql="SELECT * FROM users WHERE username='$cookee' LIMIT 0,1";
			$result=mysql_query($sql);
			if (!$result)
  				{
  				die('Issue with your mysql: ' . mysql_error());
  				}
			$row = mysql_fetch_array($result);
			if($row)
				{
			  	echo '<font color= "pink" font size="5">';	
			  	echo 'Your Login name:'. $row['username'];
			  	echo "<br>";
				echo '<font color= "grey" font size="5">';  	
				echo 'Your Password:' .$row['password'];
			  	echo "</font></b>";
				echo "<br>";
				echo 'Your ID:' .$row['id'];
			  	}
			else	
				{
				echo "<center>";
				echo '<br><br><br>';
				echo '<img src="../images/slap1.jpg" />';
				echo "<br><br><b>";
				//echo '<img src="../images/Less-20.jpg" />';
				}
			echo '<center>';
			echo '<form action="" method="post">';
			echo '<input  type="submit" name="submit" value="Delete Your Cookie!" />';
			echo '</form>';
			echo '</center>';
		}	
	else
		{
		echo '<center>';
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo "<br>";
		echo '<font color= "#FFFF00" font size = 6 >';
		echo " Your Cookie is deleted";
				setcookie('uname', $row1['username'], time()-3600);
				header ('Location: index.php');
		echo '</font></center></br>';
		
		}		


			echo "<br>";
			echo "<br>";
			//header ('Location: main.php');
			echo "<br>";
			echo "<br>";
			
			//echo '<img src="../images/slap.jpg" /></center>';
			//logging the connection parameters to a file for analysis.	
		$fp=fopen('result.txt','a');
		fwrite($fp,'Cookie:'.$cookee."\n");
	
		fclose($fp);
	
}
?>

</body>
</html>
```

### 漏洞点分析： 
```
...
$uname = check_input($_POST['uname']);
$passwd = check_input($_POST['passwd']);
...
```
页面将用户输入的uname与passwd送入过滤器进行了检测和过滤，去除了恶意语句的危害，使从此处进行注入变得困难  
但是本页面会在用户登陆成功后为用户设定cookie  
```
...
$sql="SELECT  users.username, users.password FROM users WHERE users.username=$uname and users.password=$passwd ORDER BY users.id DESC LIMIT 0,1";
		$result1 = mysql_query($sql);
		$row1 = mysql_fetch_array($result1);
		$cookee = $row1['username'];
			if($row1)
				{
				echo '<font color= "#FFFF00" font size = 3 >';
				setcookie('uname', $cookee, time()+3600);	
				header ('Location: index.php');
...
```
并在下次访问页面时将cookie传输给数据库查询来进行cookie登陆  
```
...
echo "<br></font>";
			$sql="SELECT * FROM users WHERE username='$cookee' LIMIT 0,1";
			$result=mysql_query($sql);
			if (!$result)
  				{
  				die('Issue with your mysql: ' . mysql_error());
  				}
			$row = mysql_fetch_array($result);
			if($row)
				{
			  	echo '<font color= "pink" font size="5">';	
			  	echo 'Your Login name:'. $row['username'];
			  	echo "<br>";
				echo '<font color= "grey" font size="5">';  	
				echo 'Your Password:' .$row['password'];
			  	echo "</font></b>";
				echo "<br>";
				echo 'Your ID:' .$row['id'];
			  	}
...
```
这也就导致了问题，在用户本地保存的cookie直接的写入了数据库操作语句，让恶意语句的注入变得可能  