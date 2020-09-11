# POST-Error based-string-Stacked 
TAG：POST型注入、堆叠注入  
## 漏洞利用
![less42_1](images\less42_1.png)  

一个标准的登陆界面，找回密码与注册功能均未提供，只能在登陆页面进行hack  
  
在password框输入单引号会获得一个报错信息  
![less42_2](images\less42_2.png)  
通过这个报错信息可知password框的参数被单引号包裹  
在知道包裹方式后可以利用堆叠注入添加一个用户以登陆进页面  
Payload-password:```1';insert into users(id,username,password) values ('21','less42','hello')--+```  
![less42_3](images\less42_3.png)  

成功创建用户并完成了登陆
![less42_4](images\less42_4.png)  
## 源代码分析  
### less42源代码
```
<html>
<head>
</head>
<body bgcolor="#000000">
<font size="3" color="#FFFF00">
<div align="right">
<a style="font-size:.8em;color:#FFFF00" href='index.php'><img src="../images/Home.png" height='45'; width='45'></br>HOME</a>
</div>
<?PHP

session_start();
//including the Mysql connect parameters.
include("../sql-connections/db-creds.inc");






function sqllogin($host,$dbuser,$dbpass, $dbname){
   // connectivity
//mysql connections for stacked query examples.
$con1 = mysqli_connect($host,$dbuser,$dbpass, $dbname);
   
   $username = mysqli_real_escape_string($con1, $_POST["login_user"]);
   $password = $_POST["login_password"];

   // Check connection
   if (mysqli_connect_errno($con1))
   {
       echo "Failed to connect to MySQL: " . mysqli_connect_error();
   }
   else
   {
       @mysqli_select_db($con1, $dbname) or die ( "Unable to connect to the database ######: ");
   }


   /* execute multi query */

   
   $sql = "SELECT * FROM users WHERE username='$username' and password='$password'";
   if (@mysqli_multi_query($con1, $sql))
   {
        /* store first result set */
      if($result = @mysqli_store_result($con1))
      {
	 if($row = @mysqli_fetch_row($result))
	 {
	    if ($row[1])
	    {
	       return $row[1];
	    }
	    else
	    {
	       return 0;
	    }
	 }
      }
      
      else 
      {
	echo '<font size="5" color= "#FFFF00">';
	print_r(mysqli_error($con1));
	echo "</font>";  
      }
   }
   else 
   {
	echo '<font size="5" color= "#FFFF00">';
	print_r(mysqli_error($con1));
	echo "</font>";  
    }
}





$login = sqllogin($host,$dbuser,$dbpass, $dbname);
if (!$login== 0) 
{
	$_SESSION["username"] = $login;
	setcookie("Auth", 1, time()+3600);  /* expire in 15 Minutes */
	header('Location: logged-in.php');
} 
else
{
?>
<tr><td colspan="2" style="text-align:center;"><br/><p style="color:#FF0000;">
<center>
<img src="../images/slap1.jpg">
</center>
</p></td></tr>
<?php
} 
?>

</body>
</html>
```
### 漏洞点分析  
观察一下用于登陆的php代码
```
...
 $username = mysqli_real_escape_string($con1, $_POST["login_user"]);
   $password = $_POST["login_password"];

   // Check connection
   if (mysqli_connect_errno($con1))
   {
       echo "Failed to connect to MySQL: " . mysqli_connect_error();
   }
   else
   {
       @mysqli_select_db($con1, $dbname) or die ( "Unable to connect to the database ######: ");
   }


   /* execute multi query */

   
   $sql = "SELECT * FROM users WHERE username='$username' and password='$password'";
   if (@mysqli_multi_query($con1, $sql))
...
```
通过观察代码可以发现：  
1. ```username```栏的输入被```mysqli_real_escape_string()```函数过滤了 
2. ```password```栏的输入没有被过滤  
3. 页面使用```mysqli_multi_query()```函数来执行SQL语句查询  
  
显然```password```栏存在堆叠注入可能，可以在此执行高危语句  