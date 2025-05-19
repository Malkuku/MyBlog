# 项目开发日志：技术实践与成长之路

## 前言
回顾这几天的状态，热情总是比我想象中更快被消耗完。比起茫然徘徊的小丑，我更希望自己是对着风车冲锋的疯子。
今天继续深入项目的实际业务。
状态好点的时候，再看自己EMO时写的东西，尬死我了，你真的是要尬死我😰。

## 日程

### 5.11
那天忘记记录了，不过我没有偷懒。

### 5.12
写了一个下午，不用想建表，不用想接口文档就是爽，昨天的我，你的苦没有白吃！十点半，把业务需求写得七七八八了，估计后天就能开始做前端。

### 5.13
我就不该看别人的什么感人爱情小故事😭只有对比别人的幸福的时候才能深刻感觉到自己的破防。（还不知道把耳机丢哪里了，隔几个月就要为蓝牙耳机行业做贡献😫）。被token验证卡了，想用AOP，但是实际实现起来特别特别麻烦，先看看能不能弄出来吧。

### 5.16
有几天没有写blog了，我觉得日常性的记录最好还是写写吧。今天去当了一个会议的观众，我果然还是晕会议桌。前端在写了，在写了，快的话明天能写完，然后就是修修界面。最好周日能把DOCKERFILE跑出来。

### 5.17
emmmmm,昨晚上又熬夜了，最近太太太懒散了。下午，前端基本跑完，然后要一个个微调了。wc，我好像有不少作业没写啊😰。改了一晚上，改得差不多了，明天想想数据加密的问题。

### 5.18
尝试了一下做数据加密，感觉开销很大，对原理也不是太明白，先挂到分支，把这个版本的服务器给部署了先。

### 5.19
昨天的部署没有成功，今天又弄了一天，总算成功了，还是有点小成就感的。但是错过了学科作业，还不能补交，拖延症发力了。警钟长鸣：能及时完成的事情要及时完成（指写blog，今天linux部署docker的部分就没有记到）。

## 学习内容

### 省流
1. 表与实际业务的分析
2. 事务中获取自增id
3. InvocationTargetException
4. MySQL的tinyint(1)细节
5. maven的聚合jar打包

### 1. 表与实际业务的分析
经过思考，我觉得以混合划分法来划分接口文档比较合理。以下是学生和教师接口文档的划分：

#### 学生接口文档
- 课程相关
  - 获取课程列表
  - 获取课程详情
- 练习相关
  - 获取练习列表
  - 获取练习详情
  - 提交练习答案
- 做题相关
  - 获取题目详情
  - 保存临时答案
  - 提交最终答案

#### 教师接口文档
- 练习管理
  - 创建练习
  - 修改练习
  - 复用练习
  - 删除练习
  - 查询练习列表
- 题目管理
  - 添加新题目
  - 从题库选择题目
  - 删除题目
- 批改相关
  - 获取待批改列表
  - 获取学生答题详情
  - 提交批改结果
  - 发送提醒通知

所以按照这样的划分来书写接口文档。

#### 查询所属的课程
学生 -> 学生班级关联 -> 班级 -> 练习班级关联 -> 练习 -> 课程。这样看起来是不是特别麻烦，但是我们可以直接生成视图！视图是数据库中的一个虚拟表，不存储实际数据，实际是一个预定义的查询。

比如创建学生课程视图：
```sql
CREATE VIEW v_student_courses AS
SELECT 
    u.id AS student_id,
    u.name AS student_name,
    c.id AS course_id,
    c.name AS course_name,
    s.id AS semester_id,
    s.name AS semester_name,
    s.start_time AS semester_start,
    s.end_time AS semester_end,
    COUNT(DISTINCT e.id) AS exercise_count
FROM 
    user u
JOIN 
    student_class sc ON u.id = sc.student_id
JOIN 
    exercise_class ec ON sc.class_id = ec.class_id
JOIN 
    exercise e ON ec.exercise_id = e.id
JOIN 
    course c ON e.course_id = c.id
JOIN 
    semester s ON c.semester_id = s.id
WHERE 
    u.role = 0  -- 学生角色
GROUP BY 
    u.id, c.id, s.id;
```

然后就可以通过对视图进行查询：
```sql
-- 查询某学生的所有课程
SELECT * FROM v_student_courses WHERE student_id = 123;
```

### 2. 事务中获取自增id
我之前以为对事务存在一定的误解，认为事务是同时处理sql语句，其实不是，这更像是将更改保存在了一个缓冲区。对于**同一个连接**，是可以获取到上一个插入操作产生的自增id的（尽管它还没有实际插入表中）。

```java
public static Integer getLastInsertId() throws SQLException, FileNotFoundException {
    Connection conn = getConnection();
    boolean isTxActive = ConnectionContext.isActive();
    try (Statement stmt = conn.createStatement()){
        ResultSet rs = stmt.executeQuery("SELECT LAST_INSERT_ID()");
        if (rs.next()) {
            return rs.getInt(1);
        }
    }catch (SQLException e){
        throw new SQLException("无法获取最后插入ID");
    }finally {
        if (conn != null && !isTxActive) {
            conn.close();
        }
    }
    return null;
}
```

### 3. InvocationTargetException
`InvocationTargetException`是反射调用时的“包装异常”。在通过反射调用方法时，目标方法内部抛出了异常，反射机制会将这个异常封装成`InvocationTargetException`抛出。

```java
private Throwable extractRootCause(Throwable e) {
    Throwable rootCause = e;
    while (rootCause instanceof InvocationTargetException && rootCause.getCause() != null) {
        rootCause = rootCause.getCause();
    }
    return rootCause;
}
```

### 4. MySQL的tinyint(1)细节
MySQL 驱动（如 mysql-connector-java）会将 tinyint(1) 自动映射为 Boolean。这是 JDBC 的默认行为，tinyint(1) 被识别为“类似布尔值”的类型（0 → false，非 0 → true）。

### 5. maven的聚合jar打包
配置`maven-shade-plugin`插件，将项目及其所有依赖打包成一个单独的JAR文件。

```xml
<build>
    <resources> <!-- 资源文件  默认复制到target/classes -->
        <resource>
            <directory>src/main/resources</directory> 
        </resource>
        <resource>
            <directory>src/main/webapp</directory>
            <targetPath>webapp</targetPath> <!-- 指定复制到webapp -->
        </resource>
    </resources>

    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.2.4</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal> <!-- 打包所有依赖 -->
                    </goals>
                    <configuration>
                        <transformers> <!-- 设置 JAR 文件的入口 -->
                            <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>com.anyview.xiazihao.TomcatApplication</mainClass>
                            </transformer>
                        </transformers>
                        <filters> <!-- 过滤签名 -->
                            <filter>
                                <artifact>*:*</artifact>
                                <excludes>
                                    <exclude>META-INF/*.SF</exclude>
                                    <exclude>META-INF/*.DSA</exclude>
                                    <exclude>META-INF/*.RSA</exclude>
                                </excludes>
                            </filter>
                        </filters>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 结语
这是一个时间跨度非常长的blog（已经算是半个日记了）。以后就算没有实际内容，也尽量保持这种形式的更新吧。