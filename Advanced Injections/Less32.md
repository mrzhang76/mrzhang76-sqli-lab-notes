# Less-32 **GET Bypass addslashes()[1]**
TAG：宽字节注入  
## 漏洞利用  
![less32_1](images\less32_1.png)  
使用单引号测试，发现页面对于单引号进行了转义，在单引号前添加了斜杠转义符  
同时页面还显示了参数的HEX编码，这提示了此页面可以尝试宽字节注入   
![less32_2](images\less32_2.png)  
  
在单引号前添加```%df```，利用mysql读取GBK编码字符的特点来“吃掉”转义符  
Payload:```?id=%df'```  
![less32_3](images\less32_3.png)  
  
通过报错信息获知参数的包裹方式，构造出获取回显点的  
payload:```?id=0%df' union select 1,2,3--+```  
![less32_4](images\less32_4.png)  
  
获取数据库名payload:```?id=0%df' union select 1,database(),3--+```  
![less32_5](images\less32_5.png)  

## 源码分析
### less32源代码  
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
    $string = preg_replace('/'. preg_quote('\\') .'/', "\\\\\\", $string);          //escape any backslash
    $string = preg_replace('/\'/i', '\\\'', $string);                               //escape single quote with a backslash
    $string = preg_replace('/\"/', "\\\"", $string);                                //escape double quote with a backslash
      
    
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
<img src="../images/Less-32.jpg" />
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
echo "The Query String you input in Hex becomes : ".strToHex($id). "<br>";

?>
</center>
</font> 
</body>
</html>
```

### 漏洞点分析  
```
...
function check_addslashes($string){
    $string = preg_replace('/'. preg_quote('\\') .'/', "\\\\\\", $string);         
    $string = preg_replace('/\'/i', '\\\'', $string);                               
    $string = preg_replace('/\"/', "\\\"", $string);    
    return $string;
}

...
```
页面中的过滤函数将：  
把```\```替换成```\```   
把```'```替换成```\'```   
把```"```替换成```\"```  
  
mysql在使用GBK编码的时候，会认为两个字符为一个汉字，例如```%aa%5c```就是一个汉字（前一个ascii 码大于128才能到汉字的范围）  
在过滤```'```的时候，往往利用的思路是将```'```转换为```\'```  
  
进行恶意利用时需要将```'```前面添加的```\```除掉，一般有两种方法：  
1、```%df```吃掉```\```  
具体的原因是 ```urlencode(') = %5c%27```，在```%5c%27```前面添加```%df```，形成```%df%5c%27```，而上面提到的mysql在GBK编码方式的时候会将两个字节当做一个汉字，此时```%df%5c```就是一个汉字，```%27```则作为一个单独的符号在外面  
2、将```\'```中的```\```过滤掉  
例如可以构造```%**%5c%5c%27```的情况，后面的```%5c```会被前面的```%5c```给注释掉。这也是bypass的一种方法。

