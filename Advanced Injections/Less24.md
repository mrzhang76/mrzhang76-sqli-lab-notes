# Less-24  **POST Second Degree Injections**   
TAG：POST型注入、二次注入  
## 漏洞利用  
打开此页面后发现这是一个登陆页面，有用户注册页面与密码找回页面的链接  
![less24_1](images\less24_1.png)  
  
点击密码找回的链接，发现并没有提供相关的功能  
![less24_2](images\less24_2.png)  
  
注册用户功能可以正常使用，怀疑存在二次注入问题，注册一个可用于二次注入的用户  
![less24_3](images\less24_3.png)  
>Name:admin' #  
>Password:1234  
  
![less24_4](images\less24_4.png)  
  
登陆成功后显示可以修改密码，将密码修改为123  
![less24_5](images\less24_5.png)  

使用修改后的密码登陆admin  
![less24_6](images\less24_6.png)  
成功进行了二次注入攻击  

## 源码分析  
### less24源码：  
此less源码较多，只展示关键部分源码  
  
login_create.php：
```
<html>
<head>
</head>
<body bgcolor="#000000">
<?PHP
session_start();
?>
<div align="right">
<a style="font-size:.8em;color:#FFFF00" href='index.php'><img src="../images/Home.png" height='45'; width='45'></br>HOME</a>
</div>
<?php

//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");



if (isset($_POST['submit']))
{
	

# Validating the user input........

	//$username=  $_POST['username'] ;
	$username=  mysql_escape_string($_POST['username']) ;
	$pass= mysql_escape_string($_POST['password']);
	$re_pass= mysql_escape_string($_POST['re_password']);
	
	echo "<font size='3' color='#FFFF00'>";
	$sql = "select count(*) from users where username='$username'";
	$res = mysql_query($sql) or die('You tried to be smart, Try harder!!!! :( ');
  	$row = mysql_fetch_row($res);
	
	//print_r($row);
	if (!$row[0]== 0) 
		{
		?>
		<script>alert("The username Already exists, Please choose a different username ")</script>;
		<?php
		header('refresh:1, url=new_user.php');
   		} 
		else 
		{
       		if ($pass==$re_pass)
			{
				# Building up the query........
   				
   				$sql = "insert into users ( username, password) values(\"$username\", \"$pass\")";
   				mysql_query($sql) or die('Error Creating your user account,  : '.mysql_error());
					echo "</br>";
					echo "<center><img src=../images/Less-24-user-created.jpg><font size='3' color='#FFFF00'>";   				
					//echo "<h1>User Created Successfully</h1>";
					echo "</br>";
					echo "</br>";
					echo "</br>";					
					echo "</br>Redirecting you to login page in 5 sec................";
					echo "<font size='2'>";
					echo "</br>If it does not redirect, click the home button on top right</center>";
					header('refresh:5, url=index.php');
			}
			else
			{
			?>
			<script>alert('Please make sure that password field and retype password match correctly')</script>
			<?php
			header('refresh:1, url=new_user.php');
			}
		}
}
?>
</body>
</html>
```
   
pass_change.php：
```
<html>
<head>
</head>
<body bgcolor="#000000">
<?PHP
session_start();
if (!isset($_COOKIE["Auth"]))
{
	if (!isset($_SESSION["username"])) 
	{
   		header('Location: index.php');
	}
	header('Location: index.php');
}
?>
<div align="right">
<a style="font-size:.8em;color:#FFFF00" href='index.php'><img src="../images/Home.png" height='45'; width='45'></br>HOME</a>
</div>
<?php

//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");



if (isset($_POST['submit']))
{
	
	
	# Validating the user input........
	$username= $_SESSION["username"];
	$curr_pass= mysql_real_escape_string($_POST['current_password']);
	$pass= mysql_real_escape_string($_POST['password']);
	$re_pass= mysql_real_escape_string($_POST['re_password']);
	
	if($pass==$re_pass)
	{	
		$sql = "UPDATE users SET PASSWORD='$pass' where username='$username' and password='$curr_pass' ";
		$res = mysql_query($sql) or die('You tried to be smart, Try harder!!!! :( ');
		$row = mysql_affected_rows();
		echo '<font size="3" color="#FFFF00">';
		echo '<center>';
		if($row==1)
		{
			echo "Password successfully updated";
	
		}
		else
		{
			header('Location: failed.php');
			//echo 'You tried to be smart, Try harder!!!! :( ';
		}
	}
	else
	{
		echo '<font size="5" color="#FFFF00"><center>';
		echo "Make sure New Password and Retype Password fields have same value";
		header('refresh:2, url=index.php');
	}
}
?>
<?php
if(isset($_POST['submit1']))
{
	session_destroy();
	setcookie('Auth', 1 , time()-3600);
	header ('Location: index.php');
}
?>
</center>  
</body>
</html>
```

### 漏洞点分析   
此less的二次注入漏洞主要出现在用户注册与账户密码修改页面中  
```
login_create.php:
...
    $username= mysql_escape_string($_POST['username']) ;
	$pass= mysql_escape_string($_POST['password']);
	$re_pass= mysql_escape_string($_POST['re_password']);
	echo "<font size='3' color='#FFFF00'>";
	$sql = "select count(*) from users where username='$username'";
	$res = mysql_query($sql) or die('You tried to be smart, Try harder!!!! :( ');
  	$row = mysql_fetch_row($res);
...
```
这段代码被用于用户注册功能，在用户注册阶段使用了```mysql_escape_string()```函数  
![less24_7](images\less24_7.png)  
这个函数会对用户输入的字符串进行过滤，规避了某些注入风险，但并没有对二次注入相关的利用字符进行过滤  
  
```
login_create.php:
...
if ($pass==$re_pass){
    $sql = "insert into users ( username, password) values(\"$username\", \"$pass\")";
   	mysql_query($sql) or die('Error Creating your user account,  : '.mysql_error());
	echo "</br>";
	echo "<center><img src=../images/Less-24-user-created.jpg><font size='3' color='#FFFF00'>";   				
	//echo "<h1>User Created Successfully</h1>";
	echo "</br>";
	echo "</br>";
	echo "</br>";					
	echo "</br>Redirecting you to login page in 5 sec................";
	echo "<font size='2'>";
	echo "</br>If it does not redirect, click the home button on top right</center>";
	header('refresh:5, url=index.php');
}
...
```
上述代码实现了用户注册页面将注册用户的用户名和密码输入数据库的功能  
```
$sql = "insert into users (username, password) values(\"$username\", \"$pass\")";
```  
注意这条用于将用户名和密码写入数据库的语句，它将用户名和密码使用了双引号包裹并在双引号前添加了转义符\，这将导致两字符串中的单引号不会引起报错，通常情况下会导致二次注入问题    
此段代码也没有对二次注入的利用字符进行过滤，导致了风险  
这样的注册会被直接写入数据库并不会导致报错  
>Name:admin' #  
>Password:1234  

现在再来看看密码修改部分的代码  
```
pass_change.php：
...
    $username= $_SESSION["username"];
	$curr_pass= mysql_real_escape_string($_POST['current_password']);
	$pass= mysql_real_escape_string($_POST['password']);
	$re_pass= mysql_real_escape_string($_POST['re_password']);
	
	if($pass==$re_pass)
	{	
		$sql = "UPDATE users SET PASSWORD='$pass' where username='$username' and password='$curr_pass' ";
		$res = mysql_query($sql) or die('You tried to be smart, Try harder!!!! :( ');
		$row = mysql_affected_rows();
		echo '<font size="3" color="#FFFF00">';
		echo '<center>';
		if($row==1)
		{
			echo "Password successfully updated";
...
```
在修改密码阶段，获取用户名是靠获取会话COOKIE中存储的用户名  
这样的用户名  ```admin' #```  置入修改密码的语句时看起来是这样的  
```
$sql = "UPDATE users SET PASSWORD='$pass' where username='admin' #' and password='$curr_pass' ";
```
此时语句中的注释符 ```#``` 将后半句语句注释掉，使对旧密码的确认机制失效，同时语句中查询的用户名变成了  ```admin```，这就实现了一次越权的密码修改，对  ```admin' #```  的密码修改将作用于  ```admin```  用户上  
这样就完成了一次二次注入  
只要在注册用户名使构造二次注入语句就可以达到不同的注入效果  
