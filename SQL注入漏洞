### 原理

本质是将用户的输入当做代码执行，程序将用户的输入拼接到了sql语句中，改变原来sql语句的语义造成攻击。

### 要点

快速方法：

```
使用idea 搜索${ 、like、 in( 、order by、append、+
找到是mybatis的mapping文件
便捷方法：找到调用函数后，alt+f7查看调用链，检查中间是否被过滤
容易忽视的点：HTTP头部参数
```

容易出错的地方：

第一种 like模糊查询：

使用#预编译的方式进行注解的话结果如下,会发生异常:
![mybatis_1.png](https://www.sec-in.com/img/sin/M00/00/14/wKg0C15wWc-Abe_eAAD_6MRSJrk127.png)
正确用法：

```xml
<select id="searchUser" parameterType="String" resultType="com.codeaudit.sqlinject.mybatis.User"> 
 	select * from user where name like '%'||'#param#'||'%' #Oracle
	select * from user where name like CONCAT('%',#param#,'%') #mysql
	select * from user where name like '%'+#param#+'%' #mssql
</select>
```

第二种 in范围查询：

同样的使用#预编译的方式进行注解的话无法进行正常的范围查询操作，与实际业务相悖。
正确用法：

```
<select id="searchUser" parameterType="String" resultType="com.tk.codeAudit.sqlinject.mybatis.User"> 
 	select * from user where id in 
 		<foreach collection="array" index="index" item="item" open="(" separator="," close=")">
            #{item}
        </foreach>
</select
```



第三种ORDER BY：	

如果传入的是引号包裹的字符串，那么 ORDER BY 会失效，如：SELECT * FROM user ORDER BY 'id'。

所以，如果要动态传入 ORDER BY参数，只能用字符串拼接的方式，如：

```
String sql = "SELECT * FROM user ORDER BY " + column;
```

正确用法：

1. column 是字符串型

   这种情况和 `Statement` 中描述的一样，是存在注入的。要防御就必须要手动过滤，或者将字段名硬编码到 `SQL` 语句中，比如：

   ```
   String column = "id";
   String sql ="";
   switch(column){
       case "id":
           sql = "SELECT * FROM user ORDER BY id";
           break;
       case "username":
           sql = "SELECT * FROM user ORDER BY username";
           break;
   }
   ```

2. column 是 int 型

   因为 Java是强类型语言，当用户传递的参数与后台定义的参数类型不匹配，程序会抛出异常，赋值失败。所以，不会存在注入的问题。

第四种程序编写有问题：

```
String id = req.getparameter(id);
String sql = "select * from user where id= ?";
PreparedStatement pstt = conn.prepareStatement(sql);
新手程序容易犯错，编码者以为在上⾯的sql语句中直接使⽤占位符就可以了。忘记加上pstt.setObject(1,id)。
```



### 其他

操作SQL的三种方式：	

- java.sql.Statement
- java.sql.PrepareStatement
- 使用第三方 ORM 框架 —— MyBatis 或 Hibernate

java.sql.Statement

java.sql.Statement是最原始的执行SQL的接口，使用它时，需要手动拼接SQL语句，如下面这样：

```
String sql = "SELECT * FROM user WHERE id = '" + id + "'";
Statement statement = connection.createStatement();
statement.execute(sql);
```

java.sql.PrepareStatement

```
String sql = "SELECT * FROM user WHERE id = ?";
//预编译语句
PreparedStatement preparedStatement = connection.prepareStatement(sql);
//填入参数
preparedStatement.setString(1,reqStuId);
preparedStatement.executeQuery();
```

### 案例

1.获取参数后直接使用append进行拼接，未进行过滤。

![image-20220118162022757](C:\Users\20174\AppData\Roaming\Typora\typora-user-images\image-20220118162022757.png)



### 防御

- 编写sql注入白名单公共类，进行sql注入之前将参数进行过滤
- 使用JDBC底层的sql注入预编译prepareStatement，注意使用#，不能使用$
- 使用第三方工具包common-lang中工具类StringEscapeUtils，过滤的方法是在单引号后面再加一个单引号。但是如果不需要使用单引号就能注入的点，就不能使用StringEscapeUtils
