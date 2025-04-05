# 项目构建日志：电影院售票系统开发之旅

## 前言
----------------------------------------
现在是早上九点30分，从今天开始就要构建自己的项目了。今天主要是来需求分析，绘制表结构，对项目的流程进行一个整体的安排，完成工具类的构建和测试。

## 日程
----------------------------------------
- **12点左右**：完成了整体的时间安排。
- **14:23**：开始构建项目。
- **18:00**：基本的工具类搞定了，准备尝试配置服务器。
- **19:00**：被Tomcat的路径折磨了一个小时，终于是扫描到了。
- **23:00**：今天的主要目标大部分都达成了，最后试着写了写接口文档。

## 学习内容
----------------------------------------
### 1. 需求分析
这是一个电影院售票系统，要将用户分为普通用户和管理员。

#### 普通用户功能
- 完成注册、登录、提交修改个人信息。
- 查看电影信息，查看订单历史，购买电影票。
- （提高功能）模拟支付、观影提醒、高并发处理、密码加密、可视化选座、申请退票、评论区。

#### 管理员功能
- 编辑电影信息，查看订票信息。
- （提高功能）处理退票申请、拉黑用户、关停/启用某个放映厅、数据分析看板、管理评论区。

### 2. 建表
经过和鲸鱼朋友的激烈讨论（大嘘），目前的表结构如下：

#### 用户相关
- **用户基础信息表 `users`**
  - 在用户注册时添加。
  - 在用户登录时对信息进行校验。
  - 管理员可以ban掉普通用户。
- **用户额外信息表 `user_profiles`**
  - 由普通用户增加、修改、删除相关信息。

#### 电影相关
- **电影基础信息表 `movies`**
  - 由管理员增加、修改、删除相关信息。
  - 被所有用户查询。
- **电影详细信息表 `movies_detail`**
  - 由管理员增加、修改、删除相关信息。
  - 被所有用户查询。
- **放映厅表 `halls`**
  - 由管理员增加、修改、删除相关信息。
  - 被所有用户查询。
- **场次表 `screenings`**
  - 由管理员增加、修改、删除相关信息。
  - 被所有用户查询。
- **电影评论表（待用）`comments`**
  - 因为对整体功能的影响不大，所以来不及就不做了。
  - 由普通用户增加、修改、删除自己的评论。
  - 管理员可以对评论进行封禁、删除。
  - 所有人能看见当前电影的评论。

#### 订单相关
- **订单信息表 `orders`**
  - 在普通用户进行购票后添加。
  - 考虑自动清除过期的信息。
  - 管理员可以查询。
- **订单座位表 `order_seats`**
  - 在普通用户进行购票后添加。
  - 普通用户可以查询自己的。
  - 管理员可以查询所有普通用户的。
  - 考虑自动删除过期的。
- **支付记录表 `payments`**
  - 考虑自动删除过期的（正常来说是应该留个记录的吧）。
  - 普通用户提交要求，由服务器进行增删改。
- **退票申请表 `refunds`**
  - 普通用户提交要求，管理员可以查询和处理。
  - 普通用户能看到自己的申请状态。
- **观影提示表 `reminders`**
  - 由系统进行增删改。
  - 普通用户在登录时进行时间检测。

#### 系统相关
- **系统日志表 `logs`**
  - 对每次操作进行记录。
  - 管理员可以查询和删除记录。

考虑再增加：信息统计相关表。

**13张表，怎么想都做不完罢 😨**

### 3. 安排项目流程
我将项目分为了几个大版本：

#### Beta-0.x
- 完成基础的工具类的搭建，如动态SQL语句的构建，数据库的连接，日志管理，JSON解析等。
- 项目的大致骨架，导入实体类。

#### Beta-1.x
- 在完成对应的功能前，先写好对应的接口文档。
- 系统日志表。
- 电影基础信息表（管理员部分）。
- 放映厅表（管理员部分）。
- 场次表（管理员部分）。
- 用户基础信息表。
- 登录信息校验。
- 电影基础信息表、放映厅表、场次表（普通用户部分）。

#### Beta-2.x
- 实现前端的大致框架。
- 依次完成连接上述功能的前端界面。
- 部署前端，进行联调测试。
- 发布能通过网页进行操作的Release-2.0版本。

#### Beta-3.x
- 用户额外信息表。
- 订单信息表。
- 订单座位表。
- 支付记录表。
- 发布能进行购票操作的Release-3.0版本。

#### Beta-4.x
- 观影提示表。
- 电影详细信息表。
- 电影评论表。
- 解决高并发问题。

#### Beta-5.x
- 考虑实现一些额外的功能。

**感觉时间非常紧张，还有13天，每个大版本的时间要控制在4天内，要加油啊 🔥😡🫵**

### 4. 部署Tomcat
因为弄了很久，来稍微写写记一下。

#### 引入依赖
```xml
<!-- 嵌入式Tomcat核心 -->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-core</artifactId>
    <version>9.0.65</version>
</dependency>
<!-- 必须添加的JSP支持 -->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <version>9.0.65</version>
</dependency>
<!-- JSP API -->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>4.0.1</version>
    <scope>provided</scope>
</dependency>
```
我这里用了9.0版本的Tomcat，因为这个支持对@WebServlet("")注解进行自动扫描。

#### 示例程序
```java
@Slf4j
public class TomcatApplication {
    public static void main(String[] args) throws Exception {
        // 1. 创建Tomcat实例
        Tomcat tomcat = new Tomcat();
        tomcat.setPort(8080);

        // 2. 配置webapp目录并获取Context
        String webappDir = new File("src/main/webapp").getAbsolutePath();
        Context ctx = tomcat.addWebapp("", webappDir);

        // 确保 target/classes 目录被添加到类加载路径中
        File classesDir = new File("target/classes");
        ctx.setReloadable(true); // 允许热加载
        ctx.setResources(new StandardRoot(ctx) {{
            addPreResources(new DirResourceSet(this,
                    "/WEB-INF/classes", classesDir.getAbsolutePath(), "/"));
        }});

        // 3. 启动服务器
        tomcat.start();
        log.info("Server running at http://localhost:{}", tomcat.getConnector().getPort());
        tomcat.getServer().await();
    }
}
```
这里一定要对类加载路径进行重定向，因为Tomcat默认会扫描`src/main/webapp/WEB-INF`下的字节码文件，但是我的字节码文件是放在`target/classes`目录下的。

#### 测试程序
```java
// 通过注解注册
@WebServlet("/auto")
public class AutoRegisteredServlet extends HttpServlet {
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws IOException {
        resp.getWriter().println("Auto-registered by annotation");
    }
}
```

## 结语
----------------------------------------
当我自己想构建项目又忘掉一些东西的时候，才发现以前写的blog在细节上有不少遗漏，以后还是应该注意一下。
