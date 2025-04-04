# 从 JDBC 到 MyBatis：Java 数据库开发之旅

## 前言
----------------------------------------
结果还是 9 点才睡醒……好像还落枕了，睡得不是很好。  
看看今天的安排：JDBC、MyBatis，还有一场牛客的周练。还得把学科作业写写（差点忘了）。有多余时间就向后推进。

## 日程
----------------------------------------
- 把 JDBC 看完，继续看 MyBatis  
- 下午，打了一场比赛  
- C 盘空间老是爆炸，要先做一个数据迁移了，估计要花不少时间，各种环境变量也可能要出问题，要老命了，长痛不如短痛吧。  
- 总算是迁移完了，现在快 8 点了，要抓紧时间学习  
- 9 点 30 了，写一会 blog
- 快11点30，今天下班

## 学习内容
----------------------------------------
### 1. JDBC
Java 语言操作数据库的一套 API。

#### Maven 依赖
首先，在 Maven 中导入依赖项：
```xml
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.0.33</version>
</dependency>
```

#### 入门程序
```java
public class JdbcTest {
    @Test
    public void testUpdate() throws ClassNotFoundException, SQLException {
        // 1. 加载驱动
        Class.forName("com.mysql.cj.jdbc.Driver");
        // 2. 获取连接
        String url = "jdbc:mysql://localhost:3306/web01";
        String username = "root";
        String password = "pin666";
        Connection connection = DriverManager.getConnection(url, username, password);
        // 3. 创建语句对象
        Statement statement = connection.createStatement();
        // 4. 执行 SQL 语句
        int i = statement.executeUpdate("update user set age = 25 where id = 1");
        System.out.printf("影响了 %d 行数据", i);

        // 5. 释放资源
        statement.close();
        connection.close();
    }
}
```

#### 查找数据
为了防御 SQL 注入，通常使用预编译的 SQL 语句：
```java
String sql = "select id, username, password, name, age from user where username = ? and password = ?";
```
语句对象需要换成 `PreparedStatement`：
```java
PreparedStatement statement = null;
```
通过以下方式进行连接:
```java
statement = connection.prepareStatement(sql);
statement.setString(1, "daqiao");
statement.setString(2, "123456");
```
`ResultSet` 用于储存查询返回的结果：
```java
ResultSet resultSet = null;
resultSet = statement.executeQuery();
// 处理结果集
while (resultSet.next()) {
    User user = new User(
            resultSet.getInt("id"),
            resultSet.getString("username"),
            resultSet.getString("password"),
            resultSet.getString("name"),
            resultSet.getInt("age")
    );
    // 输出 User 对象的数据
    System.out.println(user);
}
```

### 2. MyBatis
用于简化 JDBC 开发的持久层框架。

#### 入门程序
在 Spring Boot 工程中引入 MyBatis 依赖，并在 `application.properties` 中配置数据库连接信息：
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/数据库名
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.username=MySql 用户名
spring.datasource.password=MySql 密码
```

创建相关接口（通常包名叫 `mapper`）：
```java
@Mapper // 自动创建代理对象（实现类），并将其放入 IOC 容器（bean）中
public interface UserMapper {
    @Select("select * from user")
    public List<User> findAll(); // 查询所有用户信息
}
```
并创建对应的测试类（测试类所在包需要与引导类同包名）：
```java
@SpringBootTest // 标记当前类是一个 Spring Boot 测试类，自动加载 Spring Boot 的上下文环境
class SpringbootMybatisQuickstartApplicationTests {
    @Autowired
    private UserMapper userMapper;

    @Test
    public void testFindAll() {
        List<User> userList = userMapper.findAll();
        userList.forEach(System.out::println);
    }
}
```
**提示**：可以在 IDEA 中进行数据库的连接。

#### 数据库连接池
管理数据库连接的容器。Spring Boot 默认使用 Hikari，也可以手动切换到其他连接池，例如 Druid：
- 添加依赖：
```xml
<!-- Druid 连接池 -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid-spring-boot-starter</artifactId>
    <version>1.2.19</version>
</dependency>
```
- 添加配置：
```properties
spring.datasource.type=com.alibaba.druid.pool.DruidDataSource
```

#### 增删查改操作
在接口中添加注解，并添加对应方法：
```java
@Delete("delete from user where id = #{id}")
void deleteById(Integer id); // 根据 id 删除用户信息，MyBatis 会自动将参数传入到 #{id} 中
```

**符号说明**：
| 符号       | 说明                                                                 | 场景                           | 优缺点                   |
|------------|----------------------------------------------------------------------|--------------------------------|--------------------------|
| `#{...}`   | 占位符。执行时，会将 `#{...}` 替换为 `?`，生成预编译 SQL              | 参数值传递                     | 安全、性能高（推荐）     |
| `${...}`   | 拼接符。直接将参数拼接在 SQL 语句中，存在 SQL 注入问题               | 表名、字段名动态设置时使用     | 不安全、性能低           |

示例：
```java
@Delete("delete from dept where id = #{id}")
@Select("select id,name,score from ${tableName} order by ${sortField}")
```

`@Param` 注解：为接口的方法形参起名字（因为编译后的字节码文件不会保留形参的名字）：
```java
@Select("select * from user where username = #{username} and password = #{password}")
List<User> findAllByUsernameAndPassword(@Param("username") String username, @Param("password") String password); // 根据用户名和密码查询用户信息
```

#### XML 映射
除了直接注解以外，可以通过 XML 文件进行配置：
```java
//@Select("select * from user") // 代替注解
List<User> findAll(); 
```
对应的配置文件可以在 [MyBatis 中文网官网](https://mybatis.p2hp.com/) 找到：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="Mapper 接口全类名">
    <select id="方法名" resultType="返回值（自定义类）的全类名">
        select id,username,password,age from user
    </select>
</mapper>
```
**注意事项**：
- XML 映射文件必须放置在 Mapper 接口的同名包下，名称必须与 Mapper 接口一致。  
- 可以在配置文件中指定 XML 映射文件的位置：
```properties
mybatis.mapper-locations=classpath:文件位置/*.xml
```
**提示**：MyBatisX 插件可以便捷地找到方法对应的 XML 映射位置。

### 3. YAML 配置文件
可以将 Spring 的配置文件换成层级更清晰的 YAML 格式。

#### 格式规范
- 数值前边必须有空格，作为分隔符。  
- 使用缩进表示层级关系，缩进时，不允许使用 Tab 键，只能用空格（IDEA 中会自动将 Tab 转换为空格）。  
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可。  
- `#` 表示注释，从这个字符一直到行尾，都会被解析器忽略。

#### 定义对象 / Map 集合
```yaml
user:
  name: 张三
  age: 18
  password: 123456
```

#### 定义数组 / List / Set 集合
```yaml
hobby:
  - java
  - game
  - sport
```
**提示**：如果配置项的值以 0 开头，需要用 `' '` 引起来，否则会被认为是 8 进制数。

## 结语
----------------------------------------
明天又要开始上课了，会变得比较累吧。不说明天加油了，应该留到明天说：“今天加油！”
