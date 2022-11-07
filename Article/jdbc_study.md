初次接触jdbc，一些比较简单的操作。

# 1. 小试牛刀

使用jdbc去连接mysql，或者是任何数据库，都需要这几个对象：

* JDBC Driver：连接驱动，提供创建java和数据库连接的基本API。
* Connection：连接对象，类似HttpUrlConnection对象。
* Statement：语句执行官，用来执行sql语句。
* Resultset：结果集，用来存放结果结合，本质是一个[[db#5.1.4 Cursors|cursor]] ^c627c2

那么接下来，就给出每一步操作。首先是引入需要加载的驱动类。这个类并不是java自带的，而是要到各个数据库厂商对应的地方下载，比如如果要下载connector-java的MySQL版本，就可以到下面的网站：

[MySQL :: Download Connector/J](https://dev.mysql.com/downloads/connector/j/8.0.html)

下载到jar包之后，需要导入到我们的项目之中。然后就可以正常使用它了：

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

这句话通过一个反射去加载了这个类。有了驱动之后，接下来就开始获得连接对象来真正创建java程序和数据库的连接：

```java
Connection connection = DriverManager.getConnection(url, userName, password);
```

很好理解，但是重点是这三个参数都是什么：

```java
/**
url – a database url of the form jdbc:subprotocol:subname 
user – the database user on whose behalf the connection is being made 
password – the user's password
*/
String url = "jdbc:mysql://127.0.0.1:3306/?serverTimezone=GMT&useUnicode=true&characterEncoding=utf8&useSSL=false";  
String userName = "root";  
String password = "spreadzhao";
```

后面两个很好理解，但是这个url后面跟了很多参数。这些参数在我其他很多项目都做过，所以就只是在这里展示一下：

#TODO 展示一下

在有了连接之后，我们要使用具体的某一个数据库。在mysql中，可以直接使用如`use database1`这种语句去使用，那么在jdbc中，就需要这样：

```java
/**
setCatalog(String catalog);

Sets the given catalog name in order to select a subspace of this Connection object's database in which to work.

If the driver does not support catalogs, it will silently ignore this request.

Calling setCatalog has no effect on previously created or prepared Statement objects. It is implementation defined whether a DBMS prepare operation takes place immediately when the Connection method prepareStatement or prepareCall is invoked. For maximum portability, setCatalog should be called before a Statement is created or prepared.

Params:
catalog – the name of a catalog (subspace in this Connection object's database) in which to work

Throws:
SQLException – if a database access error occurs or this method is called on a closed connection
*/
connection.setCatalog("bank303");
```

然后就是在这个数据库中，创建我们执行sql语句的“执行官”了：

```java
/**
Creates a Statement object for sending SQL statements to the database. SQL statements without parameters are normally executed using Statement objects. If the same SQL statement is executed many times, it may be more efficient to use a PreparedStatement object.

Result sets created using the returned Statement object will by default be type TYPE_FORWARD_ONLY and have a concurrency level of CONCUR_READ_ONLY. The holdability of the created result sets can be determined by calling getHoldability.

Returns:
a new default Statement object

Throws:
SQLException – if a database access error occurs or this method is called on a closed connection
*/
Statement statement = connection.createStatement();
```

> createStatement()有许多的重载，我们这里只是最简单的执行sql语句，其实还可以以游标的方式去执行，后面有可能会介绍。

再然后，就可以真正去执行我们的sql语句了。这里因为非常简单，所以直接给整个的代码。如果我们的数据库中某张表是这样的：

![[Pasted image 20221107144409.png|300]]

那么我们就可以这样写：

```java
String selectall = "select * from account303";  
ResultSet resultSet = statement.executeQuery(selectall);

while (resultSet.next()){  

    System.out.println("account_number: " + resultSet.getString(1) + " | branch_name: " + resultSet.getString(2) + " | balance: " + resultSet.getDouble(3));  
    
    System.out.println("----------------------------------------");  
}
```

然后就能得到这样的结果：

```shell
account_number: A-001 | branch_name: Downtown | balance: 2122.0
----------------------------------------
account_number: A-101 | branch_name: Downtown | balance: 2122.0
----------------------------------------
account_number: A-102 | branch_name: Perryridge | balance: 1698.0
----------------------------------------
account_number: A-201 | branch_name: Brighton | balance: 3821.0
----------------------------------------
account_number: A-215 | branch_name: Mianus | balance: 2970.0
----------------------------------------
account_number: A-217 | branch_name: Brighton | balance: 3189.0
----------------------------------------
account_number: A-222 | branch_name: Redwood | balance: 2970.0
----------------------------------------
account_number: A-300 | branch_name: Perryridge | balance: 1698.0
----------------------------------------
account_number: A-301 | branch_name: Brighton | balance: 3821.0
----------------------------------------
account_number: A-302 | branch_name: Mianus | balance: 2970.0
----------------------------------------
account_number: A-303 | branch_name: Downtown | balance: 2122.0
----------------------------------------
account_number: A-304 | branch_name: Perryridge | balance: 1698.0
----------------------------------------
account_number: A-305 | branch_name: Round Hill | balance: 1492.0
----------------------------------------
account_number: A-306 | branch_name: Redwood | balance: 2970.0
----------------------------------------
account_number: A-307 | branch_name: Downtown | balance: 2122.0
----------------------------------------
account_number: A-308 | branch_name: Pownal | balance: 1698.0
----------------------------------------
account_number: A-309 | branch_name: Brighton | balance: 3821.0
----------------------------------------
account_number: A-310 | branch_name: Mianus | balance: 2970.0
----------------------------------------
account_number: A-311 | branch_name: Downtown | balance: 2122.0
----------------------------------------
account_number: A-312 | branch_name: Perryridge | balance: 1698.0
----------------------------------------
account_number: A-313 | branch_name: Brighton | balance: 3821.0
----------------------------------------
account_number: A-314 | branch_name: Mianus | balance: 2970.0
----------------------------------------
account_number: A-315 | branch_name: Downtown | balance: 2122.0
----------------------------------------
account_number: A-316 | branch_name: Perryridge | balance: 1698.0
----------------------------------------
account_number: A-317 | branch_name: Brighton | balance: 3821.0
----------------------------------------
account_number: A-318 | branch_name: Mianus | balance: 2970.0
----------------------------------------
account_number: A-319 | branch_name: Downtown | balance: 2122.0
----------------------------------------
account_number: A-320 | branch_name: Perryridge | balance: 1698.0
----------------------------------------
account_number: A-321 | branch_name: Brighton | balance: 3821.0
----------------------------------------
account_number: A-322 | branch_name: Mianus | balance: 2970.0
----------------------------------------
account_number: A-323 | branch_name: Downtown | balance: 2122.0
----------------------------------------
account_number: A-324 | branch_name: Perryridge | balance: 1698.0
----------------------------------------
account_number: A-325 | branch_name: Brighton | balance: 3821.0
----------------------------------------
account_number: A-326 | branch_name: Mianus | balance: 2970.0
----------------------------------------
account_number: A-327 | branch_name: Downtown | balance: 2122.0
----------------------------------------
account_number: A-328 | branch_name: Perryridge | balance: 1698.0
----------------------------------------
account_number: A-329 | branch_name: Brighton | balance: 3821.0
----------------------------------------
account_number: A-330 | branch_name: Mianus | balance: 2970.0
----------------------------------------
account_number: A-331 | branch_name: Downtown | balance: 2122.0
----------------------------------------
account_number: A-332 | branch_name: Perryridge | balance: 1698.0
----------------------------------------
account_number: A-333 | branch_name: Brighton | balance: 3821.0
----------------------------------------
account_number: A-334 | branch_name: Mianus | balance: 2970.0
----------------------------------------
account_number: A-335 | branch_name: Downtown | balance: 2122.0
----------------------------------------
account_number: A-336 | branch_name: Round Hill | balance: 1699.0
----------------------------------------
account_number: A-337 | branch_name: Brighton | balance: 3821.0
----------------------------------------
account_number: A-338 | branch_name: Mianus | balance: 2970.0
----------------------------------------

Process finished with exit code 0

```

注意这里resultSet和迭代器的用法非常像，就是因为[[#^c627c2|前面]]提到过，它本质是一个游标。

最后，我们需要**反向**关闭创建的资源：

```java
resultSet.close();  
statement.close();  
connection.close();
```

这样一个最简单的jdbc测试案例就写好了。下面给出完整代码：

```java
import java.sql.*;  

public class Main {  
    public static void main(String[] args) throws ClassNotFoundException, SQLException {  // 注意try-catch或者抛异常
        Class.forName("com.mysql.cj.jdbc.Driver");  
        String url = "jdbc:mysql://127.0.0.1:3306/?serverTimezone=GMT&useUnicode=true&characterEncoding=utf8&useSSL=false";  
        String userName = "root";  
        String password = "spreadzhao";  
        Connection connection = DriverManager.getConnection(url, userName, password);  
        connection.setCatalog("bank303");  
        Statement statement = connection.createStatement();  
        String selectall = "select * from account303";  
        ResultSet resultSet = statement.executeQuery(selectall);  
        while (resultSet.next()){  
            System.out.println("account_number: " + resultSet.getString(1) + " | branch_name: " + resultSet.getString(2) + " | balance: " + resultSet.getDouble(3));  
            System.out.println("----------------------------------------");  
        }  
        resultSet.close();  
        statement.close();  
        connection.close();  
    }  
}
```