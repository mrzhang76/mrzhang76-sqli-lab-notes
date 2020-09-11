# Less-27 **GET Trick with SELECT & UNION** 
TAG：GET型注入、SELECT/UNION过滤
## 漏洞利用  
此less过滤了一些危险字符串，但并不完善，存在突破方式  
![less27_1](images\less27_1.png)  
输入单引号获取报错信息来进行页面对参数包裹方式的猜测  
![less27_2](images\less27_2.png)  
页面提示过滤了select和union，双写测试发现select无法通过简单双写突破，union可以   
  
select双写后也被过滤了
![less27_3](images\less27_3.png)  
针对union的过滤可以使用双写突破  
![less27_4](images\less27_4.png)  
  
经过实际测试，less27也过滤了:注释符，空格符，UNION,SELECT  

针对双写的过滤可以使用三写突破，这种过滤方式本质上就存在很多问题，大小写混写也可以突破，SQL对大小写并不敏感   
![less27_5](images\less27_5.png)  
![less27_6](images\less27_6.png)  
  
针对本页面的过滤，同样使用URI编码进行突破  
确认回显点位置  
Payload:```?id=0'%0auniunionon%a0SeLEct%0a1,2,3;%00```
(%00进行截断)  
![less27_7](images\less27_7.png)  
  
获取数据库名  
Payload:```?id=0'%0auniunionon%a0SeLEct%0a1,database(),3;%00```  
![less27_7](images\less27_7.png)  

## 源码分析  
### less27源码： 
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-27 Trick with SELECT & UNION</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:40px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");

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
	$result=mysqli_query($con1, $sql);
	$row = mysqli_fetch_array($result, MYSQLI_BOTH);
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
		print_r(mysqli_error($con1));
		echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}




function blacklist($id)
{
$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
$id= preg_replace('/[--]/',"", $id);		//Strip out --.
$id= preg_replace('/[#]/',"", $id);			//Strip out #.
$id= preg_replace('/[ +]/',"", $id);	    //Strip out spaces.
$id= preg_replace('/select/m',"", $id);	    //Strip out spaces.
$id= preg_replace('/[ +]/',"", $id);	    //Strip out spaces.
$id= preg_replace('/union/s',"", $id);	    //Strip out union
$id= preg_replace('/select/s',"", $id);	    //Strip out select
$id= preg_replace('/UNION/s',"", $id);	    //Strip out UNION
$id= preg_replace('/SELECT/s',"", $id);	    //Strip out SELECT
$id= preg_replace('/Union/s',"", $id);	    //Strip out Union
$id= preg_replace('/Select/s',"", $id);	    //Strip out select
return $id;
}



?>
</font> </div></br></br></br><center>
<img src="../images/Less-27.jpg" />
</br>
</br>
</br>
<img src="../images/Less-27-1.jpg" />
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
本less的页面主要的漏洞存在于它的过滤函数的不严谨上  
```
function blacklist($id)
{
$id= preg_replace('/[\/\*]/',"", $id);		//strip out /*
$id= preg_replace('/[--]/',"", $id);		//Strip out --.
$id= preg_replace('/[#]/',"", $id);			//Strip out #.
$id= preg_replace('/[ +]/',"", $id);	    //Strip out spaces.
$id= preg_replace('/select/m',"", $id);	    //Strip out spaces.
$id= preg_replace('/[ +]/',"", $id);	    //Strip out spaces.
$id= preg_replace('/union/s',"", $id);	    //Strip out union
$id= preg_replace('/select/s',"", $id);	    //Strip out select
$id= preg_replace('/UNION/s',"", $id);	    //Strip out UNION
$id= preg_replace('/SELECT/s',"", $id);	    //Strip out SELECT
$id= preg_replace('/Union/s',"", $id);	    //Strip out Union
$id= preg_replace('/Select/s',"", $id);	    //Strip out select
return $id;
}
```
通过阅读过滤函数```blacklist()```可知，此页面使用了```preg_replace()```进行了正则的匹配替换，实现了对```/*,--,#,select,union```等风险字符的过滤  
虽然过滤函数中包含了union与select的一些大小写变种，但仍有漏洞，可以通过大小写变种，URL编码，双写，三写等方法突破  