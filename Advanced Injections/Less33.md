# Less-33 **GET Bypass addslashes()[2]**
TAG：宽字节注入  
## 漏洞利用  
Less33可以使用less32一样的payload，或许过滤器代码存在不同，但可以被相同的方式突破  
Payload:```?id=0%df' union select 1,2,3--+```
![less33_1](images\less33_1.png)  
  
## 源码分析
### less33源代码  
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-32 **Bypass addslashes()**</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="5" color="#00FF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");

function check_addslashes($string)
{
    $string= addslashes($string);    
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
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
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
<img src="../images/Less-33.jpg" />
</br>
</br>
</br>
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
function strToHex($string)
{
    $hex='';
    for ($i=0; $i < strlen($string); $i++)
    {
        $hex .= dechex(ord($string[$i]));
    }
    return $hex;
}
echo "Hint: The Query String you input is escaped as : ".$id ."<br>";
echo "The Query String you input in Hex becomes : ".strToHex($id);
?>
</center>
</font> 
</body>
</html>
```
  
### 漏洞点分析  
```
function check_addslashes($string)
{
    $string= addslashes($string);    
    return $string;
}
```  
此less使用了php的```addslashes()```函数  
![less33_2](images\less33_2.png)  
此函数也在某些情况下存在绕过操作：  
1. 字符编码问题导致绕过:  
设置数据库字符为gbk导致宽字节注入  
使用icon,mb_convert_encoding转换字符编码函数导致宽字节注入  
2. 编码解码导致的绕过:  
url解码导致绕过addslashes  
base64解码导致绕过addslashes  
json编码导致绕过addslashes  
3. 一些特殊情况导致的绕过:  
没有使用引号保护字符串，直接无视addslashes  
使用了stripslashes  
字符替换导致的绕过addslashes  
  
本less中就使用了宽字节注入来绕过```addslashes()```函数  