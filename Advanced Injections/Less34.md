# Less-34 **Post Bypass Add SLASHES**
TAG：宽字节注入  
## 漏洞利用  
Less34中需要在post传参方式中使用宽字节注入，这和get方式有所不同  
页面会使用urlencode将get方式传递的参数处理，当使用```%df```时会把转义符```\```合并  
在post传参方式中，就需要使用另外的方法来进行宽字节注入了  
![less34_1](images\less34_1.png)    
  
方法1：使用utf-8的单引号```'```转换为utf-16的```�'```  
![less34_2](images\less34_2.png)    
构造万能密码  
Username:```�'or 1=1#```  
![less34_3](images\less34_3.png)    
  
方法2：使用burpsite进行宽字节注入    
Payload:```uname=%bb' or 1 #&passwd=1```  
![less34_4](images\less34_4.png)    
![less34_5](images\less34_5.png)    
  
可以使用limit实现平行越权  
Payload:uname=```%bb' or 1 limit 1,1 #&passwd=1```  
![less34_6](images\less34_6.png)    
![less34_7](images\less34_7.png)    
  
## 源码分析
### less34源代码  
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<title>Less-34- Bypass Add SLASHES</title>
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

</div>
</div>
<div style=" margin-top:10px;color:#FFF; font-size:23px; text-align:center">
<font size="3" color="#FFFF00">
<center>
<br>
<br>
<br>
<img src="../images/Less-34.jpg" />
</center>

<?php
//including the Mysql connect parameters.
include("../sql-connections/sqli-connect.php");


// take the variables
if(isset($_POST['uname']) && isset($_POST['passwd']))
{
	$uname1=$_POST['uname'];
	$passwd1=$_POST['passwd'];

        //echo "username before addslashes is :".$uname1 ."<br>";
        //echo "Input password before addslashes is : ".$passwd1. "<br>";
        
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Name:'.$uname1);
	fwrite($fp,'Password:'.$passwd1."\n");
	fclose($fp);
        
        $uname = addslashes($uname1);
        $passwd= addslashes($passwd1);
        
        //echo "username after addslashes is :".$uname ."<br>";
        //echo "Input password after addslashes is : ".$passwd;    

	// connectivity 
	mysqli_query($con1, "SET NAMES gbk");
	@$sql="SELECT username, password FROM users WHERE username='$uname' and password='$passwd' LIMIT 0,1";
	$result=mysqli_query($con1, $sql);
	$row = mysqli_fetch_array($result, MYSQLI_BOTH);

	if($row)
	{
  		//echo '<font color= "#0000ff">';	
  		
  		echo "<br>";
		echo '<font color= "#FFFF00" font size = 4>';
		//echo " You Have successfully logged in\n\n " ;
		echo '<font size="3" color="#0000ff">';	
		echo "<br>";
		echo 'Your Login name:'. $row['username'];
		echo "<br>";
		echo 'Your Password:' .$row['password'];
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
		print_r(mysqli_error($con1));
		echo "</br>";
		echo "</br>";
		echo "</br>";
		echo '<img src="../images/slap.jpg" />';	
		echo "</font>";  
	}
}

?>

</br>
</br>
</br>
<font size='4' color= "#33FFFF">
<?php
 
echo "Hint: The Username you input is escaped as : ".$uname ."<br>";
echo "Hint: The Password you input is escaped as : ".$passwd ."<br>";
?>

</font>
</div>
</body>
</html>
```

### 漏洞点分析  
```
...
$uname1=$_POST['uname'];
	$passwd1=$_POST['passwd'];

        //echo "username before addslashes is :".$uname1 ."<br>";
        //echo "Input password before addslashes is : ".$passwd1. "<br>";
        
	//logging the connection parameters to a file for analysis.
	$fp=fopen('result.txt','a');
	fwrite($fp,'User Name:'.$uname1);
	fwrite($fp,'Password:'.$passwd1."\n");
	fclose($fp);
        
        $uname = addslashes($uname1);
        $passwd= addslashes($passwd1);
        
        //echo "username after addslashes is :".$uname ."<br>";
        //echo "Input password after addslashes is : ".$passwd;    

	// connectivity 
	mysqli_query($con1, "SET NAMES gbk");
	@$sql="SELECT username, password FROM users WHERE username='$uname' and password='$passwd' LIMIT 0,1";
	$result=mysqli_query($con1, $sql);
	$row = mysqli_fetch_array($result, MYSQLI_BOTH);
...
```
此less中也是使用```addslashes()```函数进行过滤操作，突破方式与less33相似，只是需要根据传参类型post进行相应的改变  