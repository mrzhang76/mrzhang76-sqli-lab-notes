# Less-25 **GET Trick with OR & AND**
TAG：GET型注入、双写突破  
## 漏洞利用  
打开页面就可知页面对```or```和```and```进行了过滤
![less25_1](images\less25_1.png)  
  
在这个页面中，输入的内容被输出到页面上，这方便了对于过滤器工作方式的探查  
![less25_2](images\less25_2.png)  
  
输入一个常见的探测语句payload:?id=1 and 1=1 
![less25_3](images\less25_3.png)  
  
通过回显可以得知and被过滤，经过测试or也被过滤，使用双写尝试绕过  
Payload:```?id=-1 oorr 1=1```
![less25_4](images\less25_4.png)  
  
虽然成功完成了对过滤的突破，但是并没有获取到想要的页面，使用一个单引号来获取报错信息  
![less25_5](images\less25_5.png)  
  
根据报错信息修改 payload:```?id=-1' oorr '1'='1```  
![less25_6](images\less25_6.png)  
  
## 源码分析  
### less25源码：  
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-25 Trick with OR & AND</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:40px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sql-connect.php");


// take the variables 
if(isset($_GET['id']))
{
	$id=$_GET['id'];
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'ID:'.$id."\n");
	fclose($fp);

	//fiddling with comments
	$id= blacklist($id);
	//echo "<br>";
	//echo $id;
	//echo "<br>";
	$hint=$id;

// connectivity 
	$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
	$result=mysql_query($sql);
	$row = mysql_fetch_array($result);
	if($row)
	{
	  	echo "<font size='5' color= '#99FF00'>";	
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
else 
{ 
	echo "Please input the ID as parameter with numeric value";
}


function blacklist($id)
{
	$id= preg_replace('/or/i',"", $id);			//strip out OR (non case sensitive)
	$id= preg_replace('/AND/i',"", $id);		//Strip out AND (non case sensitive)
	
	return $id;
}




?>
</font> </div></br></br></br><center>
<img src="../images/Less-25.jpg" />
</br>
</br>
</br>
<img src="../images/Less-25-1.jpg" />
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
echo "Hint: Your Input is Filtered with following result: ".$hint;
?>
</font> 
</center>
</body>
</html>
```
### 漏洞点分析   
此页面进行了对```or```和```and```的过滤，但是很容易被突破  
```
function blacklist($id)
{
	$id= preg_replace('/or/i',"", $id);			
	$id= preg_replace('/AND/i',"", $id);		
	return $id;
}
```
  
通过观察用于过滤的blacklist()函数可以发现，使用了preg_replace()正则替换函数进行了风险字符串的替换，但正则表达式```/or/i```与```/AND/i```只是匹配了大小写不敏感的```or```与```and```，这使得只要简单的双写```oorr```,```aandnd```就规避了对敏感字符的过滤  

