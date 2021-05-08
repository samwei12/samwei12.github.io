---
title: JDBC学习一---JDBC入门
date: 2020-03-07  17:54:52
categories:
  - Server
tags: 
  - JDBC
---

> [原文链接](http://blog.samwei12.cn/2020/03/07/Server/JDBC%E5%AD%A6%E4%B9%A0%E4%B8%80/)

今天开始会写一系列 Java 后端学习的笔记，一方面是为了以后翻阅查看，更主要的原因是通过写作输出的方式让自己的印象更深，避免遗忘。

首先是简单记录下自己学习使用 JDBC 的历程，由于目前基本都是通过一些类似 MyBatis 的框架来进行数据库操作，所以 JDBC 的使用不需要掌握太深入，仅作为了解即可。

## 简介

首先我们学习任何东西之前都需要先了解几个问题，基本上的思路是：
    1. xxx 是什么？
    2. 有什么作用？也就是为什么需要 xxx？
    3. 怎么使用（简单入门即可）？
    4. 分别就主要链路进行知识补充
之后，可以根据实际情况决定是否要进一步深入了解，还是只作为简单学习即可。

JDBC 也不例外。

<!--more-->

### 什么是 JDBC？

> Java数据库连接，（Java Database Connectivity，简称JDBC）是Java语言中用来规范客户端程序如何来访问数据库的应用程序接口，提供了诸如查询和更新数据库中数据的方法。JDBC也是Sun Microsystems的商标[1]。JDBC是面向关系型数据库的。

上面是维基百科的解释，从这里我们获取到的几个有用信息：
1. 简单来说，JDBC 就是 Java 定义的一套用来规范和约束客户端程序如何访问数据库的**接口**，尤其是需要注意接口一词
2. JDBC 是面向关系型数据库的，也就是说对于非关系型数据库用这一套接口是不行的。

### 为什么需要 JDBC？

回答类似这种问题其实套路也比较简单，我们思考一下，假如没有 JDBC 会怎么样？

我们知道，关系型数据库是有很多分类的，例如常见的 MySQL、Oracle、SQL Server等等，那么如果没有一套标准接口的话，意味着我们如果开发 MySQL 数据库，不光需要引入 MySQL 的官方驱动，还需要引入官方的 Jar 包，根据对应的 CRUD 接口编写数据库操作代码，如果这时候你还有另一个使用 Oracle 项目，那么同样的工作需要重新进行一次，可想而知，这个适配成本是随着数据库厂商的增多逐渐上升的。

那么怎么解决这类问题呢？有一句名言是这么说的，**“计算机科学领域的任何问题都可以通过增加一个简介的中间层来解决。”**，回到我们的问题上，Java 开发者就提供了这么一个中间层，也就是 **JDBC**。

[![20200301214356](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20200301214356.png)](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20200301214356.png)

通过这个图可以看到，由于有了这么一个中间层，我们在开发代码时，只需要使用 JDBC 相关的接口即可，不需要关心具体实现底层的是 MySQL 还是 Oracle。

## 如何使用 JDBC？

接下来我们看下如何快速搭建一个 JDBC 的demo。通常在 Java 后端开发中，我们只需要掌握几个核心点就可以快速上手一个框架，它们分别是：

1. jar 包或者 maven坐标
2. 关键的配置文件和属性
3. 核心的类和 API

下面我们就从这个几方面入手快速了解一下：

### jar 包或者 maven坐标

使用数据库需要这么几个比较重要的包：

1. `java.sql`：所有与 JDBC 访问数据库相关的接口和类,可以说有了这里面提供的接口和类已经足够搭建一个 demo 实现基本功能
2. `javax.sql`：数据库扩展包，提供数据库额外的功能，如：连接池。接下来我们只是实现基本功能，可以不使用这个包。
3. 数据库的驱动：由各大数据库厂商提供，需要额外去下载，是对 JDBC 接口实现的类。

这里面只有数据库的驱动是需要我们自己使用 jar 包或者 maven 依赖的。

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.17</version>
</dependency>
```

这里由于我本地的 MySQL 是 8.x 版本，因此使用了 8.0.17， 如果你的 MySQL 是 5.x 版本可以使用 5.1.39。

### 关键的配置文件和属性

JDBC 中可以通过使用配置文件来加载数据库相关配置，例如指定数据库 URL、账户、密码等，这个等下详细介绍。

### 核心的类和 API

JDBC 中几个比较核心的类和作用如下：

- DriverManager 类
    - 管理和注册数据库驱动
    - 得到数据库连接对象
- Connection 接口
    - 一个连接对象，可用于创建 Statement 和 PreparedStatement 对象
- Statement 接口
    - 一个 SQL 语句对象，用于将 SQL 语句发送给数据库服务器。
- PreparedStatemen 接口
    - 一个 SQL 语句对象，是 Statement 的子接口
- ResultSet 接口
    - 用于封装数据库查询的结果集，返回给客户端 Java 程序

简单了解即可，等下具体搭建工程会详细介绍。

### 搭建测试工程

这里我们先简单建一个表，然后插入几条数据用于查看。

```sql
CREATE TABLE `bank` (
    `name` varchar(10) DEFAULT NULL,
    `money` double DEFAULT NULL
)

insert into bank
values (money = 100, name = '张三'),
       (money = 200, name = '李四'),
       (money = 300, name = '王五'),
       (money = 400, name = '赵六');
```

这样我们的表就建好了，数据也有了，我们就正式开始通过编写代码查找表中的数据。

[![20200307202152](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20200307202152.png)](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20200307202152.png)

其实说白了使用 JDBC 的核心步骤就是这些：

1. 导入驱动jar包
2. 注册驱动
3. 获取数据库连接对象
4. 定义sql
5. 获取执行sql语句的对象 Statement
6. 执行sql，接受返回结果
7. 处理结果
8. 释放资源

那么我们看下，这里其中导入 jar 包我们已经通过引入 maven 坐标解决了，接下来看看如何注册驱动。

```java
Class.forName("com.mysql.cj.jdbc.Driver");
```

这不就是加载类吗？哪里有注册驱动呢？别着急，我们看下源码。跳到 `com.mysql.cj.jdbc.Driver` 中，我们发现 `Driver` 中的静态代码块调用了 `DriverManager` 的`registerDriver`方法，然后创建了一个`Driver`对象作为入参传入。这也跟我们之前提到的 `DriverManager` 有管理和注册驱动的作用是吻合的。

```java
static {
    try {
        DriverManager.registerDriver(new Driver());
    } catch (SQLException var1) {
        throw new RuntimeException("Can't register driver!");
    }
}
```

我们看下 `DriverManager` 的官方文档，可以看到

> JDBC 4.0 Drivers must include the file META-INF/services/java.sql.Driver. This file contains the name of the JDBC drivers implementation of java.sql.Driver.

> Applications no longer need to explicitly load JDBC drivers using Class.forName(). Existing programs which currently load JDBC drivers using Class.forName() will continue to work without modification.

这几句话是什么意思呢？简单来说就是可能 JDK 觉得大家老是这么加载驱动比较繁琐，在 JDBC4.0 之后新增了一个约定，数据库驱动实现方必须要在 jar 包的 `META-INF/services/java.sql.Driver` 路径里面写明指定的 `Driver` 类，这样 JVM 会自动将对应的驱动进行加载，而不再需要程序员在开发过程中手动调用 `Class.forName()` 了，当然已经调用了的代码可以正常运行而不会受到影响，所以这一步我们可以省略了。

接下来是获取数据库连接对象 `Connection`，前面我们已经提到，这个类是用来创建 `Statement` 对象的，而 `Statement` 对象就是真正进行数据库操作的类，

```java
final Connection connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/samwei12", "root","root");
```

通过调用 `DriverManager` 的`getConnection`方法，我们就得到了一个 `Connection` 对象。它的几个入参也比较显而易见，就是数据库连接时的 url、账户、密码等信息。这也是 `DriverManager` 的第二个功能，就是获取数据库连接对象 `Connection` 的实例。

接下来是定义 sql 语句，这个最好理解，这里我们把张三的存款设置成 1000。

```java
String sql = "update bank set money=1000 where name='张三'";
```

接下来是创建 `Statement` 对象，

```java
final Statement statement = connection.createStatement();
```

执行 sql 语句：

```java
final int count = statement.executeUpdate(sql);
```

入参代表我们需要执行的 sql 语句，出参代表本条 update 语句影响的行数。

```java
System.out.println(count);
```
这里我们简单打印下就可以，如果查询到的是需要封装的对象，那么还需要其他处理，这里不展开。

最后就是释放资源，这个一定要记住。

```java
statement.close();
connection.close();
```

之后我们执行一下，可以看到打印结果是 1，同时数据库中对应数据也发生了变动。

[![20200307192219](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20200307192219.png)](https://learner.oss-cn-hangzhou.aliyuncs.com/img/20200307192219.png)


示例代码地址：
- https://github.com/samwei12/jdbc-demo

## 详细 API 了解

### DriverManager

前面已经讲得比较清楚，它的主要作用就是注册驱动+管理数据库连接，这里只说一下 MySQL 的 URL 写法。

- `jdbc:mysql://ip地址(域名):端口号/数据库名称`
    - 例子：`jdbc:mysql://localhost:3306/samwei12`
    - 细节：如果连接的是本机mysql服务器，并且mysql服务默认端口是3306，则url可以简写为：`jdbc:mysql:///数据库名称`

### Connection

数据库连接对象，它的主要功能包括创建一个 `Statement` 和管理事务。

1. 获取执行sql 的对象
    * `Statement createStatement()`
    * `PreparedStatement prepareStatement(String sql)`
2. 管理事务：
    * 开启事务：`setAutoCommit(boolean autoCommit)` ：调用该方法设置参数为false，即开启事务
    * 提交事务：`commit()`
    * 回滚事务：`rollback()`

### Statement

执行sql的对象，CRUD 都可以执行：

1. `boolean execute(String sql)` ：可以执行任意的sql语句
2. `int executeUpdate(String sql)` ：执行DML（insert、update、delete）语句、DDL(create，alter、drop)语句
    * 返回值：影响的行数，可以通过这个影响的行数判断DML语句是否执行成功 返回值>0的则执行成功，反之，则失败。
3. `ResultSet executeQuery(String sql)`：执行DQL(select) 语句

这里我们最常用的的一般就是最后一个方法了，示例代码：

```java
//4. 定义sql
String sql = "select * from bank";
// 5. 获取执行sql语句的对象 Statement
final Statement statement = connection.createStatement();
// 6. 执行sql，接受返回结果
final ResultSet resultSet = statement.executeQuery(sql);
// 7. 处理结果
while (resultSet.next()) {
    final double money = resultSet.getDouble("money");
    final String name = resultSet.getString("name");
    System.out.println(name+"银行账户："+money);
}
```

通过 `ResultSet` 的 `next` 方法遍历结果，之后得到的就是一行的数据，分别取出每一列的数据处理即可。

### ResultSet

结果集对象，封装查询结果

* `boolean next()`: 游标向下移动一行，判断当前行是否是最后一行末尾(是否有数据)，如果是，则返回false，如果不是则返回true
* `getXxx(参数)`:获取数据
    * Xxx：代表数据类型，如： `int getInt() , String getString()`
- 具体的使用方法 `executeQuery` 这里已经展示过了

### PreparedStatement

执行sql的对象，继承自 `Statement`。看到这里你可能会比较奇怪，我们不是已经有了 `Statement` 类了嘛，为什么还需要再搞一个子类出来呢？我们先通过一个示例介绍一下什么是 **SQL 注入**。

创建一个用户登录信息表，包括用户名和密码。

```sql
create table user (

id int primary key auto_increment,

name varchar(20),

password varchar(20)

)

insert into user values (null,'jack','123'),(null,'rose','456');
```

然后模拟一下用户的登录，如果用户输入的用户名和密码能够匹配到一条记录，那么我们认为登录成功，否则认为登录失败。

代码比较简单，核心代码如下：

```java
String sql = "select * from user where name='"+userName+"' and password='"+password+"'";
final Statement statement = connection.createStatement();
final ResultSet resultSet = statement.executeQuery(sql);
if (resultSet.next()) {
    System.out.println("登录成功，欢迎您：" + userName);
} else {
    System.out.println("登录失败");
}
```

验证一下：

```bash
请输入用户名：
JACK
请输入用户密码：
123
登录成功，欢迎您：JACK
```

```bash
请输入用户名：
jack1
请输入用户密码：
123
登录失败
```

看起来好像一切正常，没毛病，那么我们尝试一下如下输入：

```bash
请输入用户名：
我是一个不存在的用户
请输入用户密码：
我是密码' or 'a'='a
```

这时我们发现，神奇的事情出现了：

```bash
登录成功，欢迎您：我是一个不存在的用户
```

这是为什么呢？？？其实我们 debug 一下就会发现，原因很简单，我们传给 `Statement` 对象的是一个字符串，那么上述参数真正执行的 SQL 语句是 `select * from user where name='我是一个不存在的用户' and password='我是密码' or 'a'='a'`。 大家发现了吗？我们的原始 SQL 只是查询用户名和密码信息，但是如果用户输入了类似上面这种奇怪的密码，在我们的 SQL 后面拼接上其他字符串，就可以绕过我们的检查，随意输入任何用户名和密码都可以获得登录授权，这无疑是很危险的。

> SQL注入（英语：SQL injection），也称SQL注入或SQL注码，是发生于应用程序与数据库层的安全漏洞。简而言之，是在输入的字符串之中注入SQL指令，在设计不良的程序当中忽略了字符检查，那么这些注入进去的恶意指令就会被数据库服务器误认为是正常的SQL指令而运行，因此遭到破坏或是入侵。

以上是维基百科对应的词条解释，那么怎么解决这种问题呢？JDK 的作者们提供的 `PreparedStatement` 就是用来处理这种问题的。我们来看一下它的用法。

1. Connection 接口中的方法
    1. `PreparedStatement prepareStatement(String sql)`: 指定预编译的 SQL 语句，SQL 语句中使用占位符 `?` 创建一个语句对象
2. PreparedStatement 接口中的方法
    1. `int executeUpdate()`: 执行 DML，增删改的操作，返回影响的行数。
    2. `ResultSet executeQuery()`: 执行 DQL，查询的操作，返回结果集

使用步骤：

3. 编写 SQL 语句，未知内容使用?占位：`"SELECT * FROM user WHERE name=? AND password=?";`
4. 获得 `PreparedStatement` 对象
5. 设置实际参数：`setXxx(占位符的位置, 真实的值)`
    1. 对应一系列重载的方法
6. 执行参数化 SQL 语句
7. 关闭资源

具体的代码示例如下：

```java
// 1. 构造一个带参数的 SQL 语句，这里不需要加单引号了，而是使用占位符 ？
String sql = "select * from user where name=? and password=?";
// 2. 获取 PreparedStatement 对象
final PreparedStatement preparedStatement = connection.prepareStatement(sql);
// 3. 设置参数，这里需要注意的是索引是从 1 开始的，而不是 0
preparedStatement.setString(1, userName);
preparedStatement.setString(2, userName);
// 4. 执行参数化 SQL 语句，这里由于 sql 语句已经准备好了，不再需要传入参数，直接执行即可
final ResultSet resultSet = preparedStatement.executeQuery();
if (resultSet.next()) {
    System.out.println("登录成功，欢迎您：" + userName);
} else {
    System.out.println("登录失败");
}
```

使用起来还是比较简单的，需要注意的几个点已经写在了注释里面。
执行结果如下：

```bash
请输入用户名：
我是一个不存在的用户
请输入用户密码：
我是密码' or 'a'='a
登录失败
```

可以看到使用 `PreparedStatement` 有效的解决了 SQL 注入的问题，它还有以下好处：

1. `prepareStatement()` 方法会先将 SQL 语句发送给数据库预编译。`PreparedStatement` 会引用着预编译后的结果。 可以多次传入不同的参数给 `PreparedStatement` 对象并执行。减少 SQL 编译次数，提高效率。
2. 提高了程序的可读性

所以我们尽可能全部都使用 `PreparedStatement` 的方式，而不是 `Statement` 类。

## 总结

至此，一个简单的 JDBC demo 就搭建完毕了。由于目前已经不会有人直接使用 JDBC 进行数据库操作了，我们只是为了接下来学习 MyBatis 进行一个铺垫，简单了解即可，不需要深入。

示例代码地址：
- https://github.com/samwei12/jdbc-demo

## Relations

- https://zh.wikipedia.org/wiki/Java%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5
- https://zh.wikipedia.org/wiki/SQL%E6%B3%A8%E5%85%A5