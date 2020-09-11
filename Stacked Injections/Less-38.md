# GET-Stacked Query Injection-String  
TAG：堆叠注入、GET型注入
## 漏洞利用  
通过本less题目猜测为堆叠注入  
![less38_1](images\less38_1.png) 
    
按惯例输入一个单引号获取报错信息判断参数包裹方式  
![less38_2](images\less38_2.png)  
此处可知参数被单引号包裹   
  
在进行堆叠注入之前，先测试一下有多少个用户  
![less38_3](images\less38_3.png)  
当?id=17时没有回显，说明数据库中只有16个用户，下面使用堆叠注入添加一个新的用户  
![less38_4](images\less38_4.png)  
  
Payload:```?id=1';insert into users(id,username,password) values ('17','less38','hello')--+```  
![less38_5](images\less38_5.png)  
成功添加编号为17的用户，这标志着堆叠注入完成了对数据库的非法操作  
## 源代码分析  
### less38源代码  
```
<?php
error_reporting(0);
include("../sql-connections/db-creds.inc");
?>
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-38 **stacked Query**</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">


<?php




// take the variables 
if(isset($_GET['id']))
{
$id=$_GET['id'];
//logging the connection parameters to a file for analysis.
$fp=fopen('result.txt','a');
fwrite($fp,'ID:'.$id."\n");
fclose($fp);

// connectivity
//mysql connections for stacked query examples.
$con1 = mysqli_connect($host,$dbuser,$dbpass,$dbname);
// Check connection
if (mysqli_connect_errno($con1))
{
    echo "Failed to connect to MySQL: " . mysqli_connect_error();
}
else
{
    @mysqli_select_db($con1, $dbname) or die ( "Unable to connect to the database: $dbname");
}



$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
/* execute multi query */
if (mysqli_multi_query($con1, $sql))
{
    
    
    /* store first result set */
    if ($result = mysqli_store_result($con1))
    {
        if($row = mysqli_fetch_row($result))
        {
            echo '<font size = "5" color= "#00FF00">';	
            printf("Your Username is : %s", $row[1]);
            echo "<br>";
            printf("Your Password is : %s", $row[2]);
            echo "<br>";
            echo "</font>";
        }
//            mysqli_free_result($result);
    }
        /* print divider */
    if (mysqli_more_results($con1))
    {
            //printf("-----------------\n");
    }
     //while (mysqli_next_result($con1));
}
else 
    {
	echo '<font size="5" color= "#FFFF00">';
	print_r(mysqli_error($con1));
	echo "</font>";  
    }
/* close connection */
mysqli_close($con1);


}
	else { echo "Please input the ID as parameter with numeric value";}

?>
</font> </div></br></br></br><center>
<img src="../images/Less-38.jpg" /></center>
</body>
</html>
```
### 漏洞点分析  
```
...
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
/* execute multi query */
if (mysqli_multi_query($con1, $sql))
{
    
    
    /* store first result set */
    if ($result = mysqli_store_result($con1))
    {
...
```
通过对源代码的分析，发现此页面使用```mysqli_multi_query()```函数来执行SQL语句查询  
![less38_6](images\less38_6.png)  
此函数可以进行一个或多个语句的执行，这意味着可以使用```;```截断当前语句并并列执行构造好的恶意语句实现堆叠注入  