# 二轮考核学习计划与总结

## 前言
----------------------------------------
二轮考核的内容下来了，由整体项目构建转为实现特定模块的功能。对细节的要求更高了，而且有手搓线程池、手搓依赖注入等进阶要求，又有得学力。嘻嘻，太简单了，只要我手搓 Spring Boot 框架......又幻想了家人们。  
我决定先自闭几天，把现在该学的学学了，不然满脑子都是项目 😰

--------4.23-------
做梦梦到我把依赖注入一步步手搓出来了 😋，然后就被肚子痛醒了。  
来迟到地说说这几天的安排，因为 blog 越做越多，所以要弄一个目录来方便我找需要的知识；这几天把前端学完，大概 2 - 3 天吧，然后 2 天左右学习项目的部署，就可以开始看看二轮项目要怎么入手了。

测试git提交

----------------------------------------

## 日程
----------------------------------------
最绝望的一集，因为之前移动了硬盘导致 VMware 卸也卸不干净，安装又安装不了，弄到 9 点多才弄完，今天可能学不了什么新东西了。  
如果没有内容的话，大概会放在明天做成合订版。

--------4.23-------
现在下午 1 点，上课偷偷把 Maven 进阶部分学了，写了 blog 就去睡觉。  
6 点 40，摸了一会🐟，先来复习一下前端内容，提醒自己别忘记看完计算机网络剩下的一点点内容。  
欧 shift，差点忘记要统计奖项了。  
发现前端有部分笔记没写，今天补补吧。  
做完笔记应该快 10 点，要处理些琐事了，结果弄了半天也没把学校邮箱弄出来。

----------------------------------------

## 学习记录
----------------------------------------
记录一下今天学的知识，方便复习。

### 操作系统
1. 程序链接的三种方式，装入内存的三种方式
2. 内存保护，越界检查
3. 连续内存分配管理方式
4. 动态分区分配算法
5. 分页储存，地址变换的计算
6. 基本地址变换机构，快表优化速度
7. 多级页表的结构，注意点

--------4.23-------
### 计算机网络
1. 网络层的功能
2. IP 数据报的结构
3. TTL
4. IPv4 协议

----------------------------------------

## 学习内容
----------------------------------------
### 省流
1. Maven 进阶
2. 请求 js 封装
3. ElementPlus 表单验证
4. ElMessageBox 消息弹框

### 1. Maven 进阶
#### 1）分模块设计
在实际项目开发中，一个人往往只负责某个模块的开发。将模块按功能/层进行拆分，再通过 Maven 进行导入。

#### 2）继承，聚合
类似 Java 的继承，Maven 子工程可以继承父工程的配置信息以及依赖。设计一个仅带 Maven 配置的模块作为父工程：
```xml
<parent> 
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.4.4</version>
    <relativePath/>
</parent>

<groupId>com.itheima</groupId>
<artifactId>tlias-parent</artifactId>
<version>1.0-SNAPSHOT</version>
<packaging>pom</packaging><!-- 设置打包方式为 pom -->
```

在子工程中引入继承：
```xml
<parent>
    <groupId>com.itheima</groupId>
    <artifactId>tlias-parent</artifactId>
    <version>1.0-SNAPSHOT</version>
    <relativePath>../tlias-parent/pom.xml</relativePath> <!-- 父工程相对路径 -->
</parent>
```

#### 依赖管理
通过 `<dependencyManagement>` 来控制子工程的依赖版本：
```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.36</version>
            <optional>true</optional>
        </dependency>
    </dependencies>
</dependencyManagement>
```
**注意**：  
- 此时子工程的依赖可以不需要指定版本，如果指定，以子工程的版本为准。  
- 依赖管理必须指定版本，而不能默认继承父工程的版本，这只在 `<dependencies>` 中生效。

#### 聚合
当父类构建 Maven 工程时，会将 `<modules>` 作为它的子工程自动构建：
```xml
<modules>
    <module>../tlias-utils</module>
    <module>../tlias-pojo</module>
    <module>../tlias-web-management</module>
</modules>
```

### 2. 请求 js 封装
解耦实现基本的前端发送请求路径功能。

#### 1）在 `vite.config.js` 配置 api 路径信息
```javascript
server: {
    proxy: {
        '/api': {
        target: 'http://localhost:8080',
        secure: false,
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api/, ''),
        }
    }
}
```

#### 2）实现异步发送 api 请求的工具类
```javascript
import axios from 'axios'

// 创建 axios 实例对象
const request = axios.create({
  baseURL: '/api',
  timeout: 600000
})

// axios 的响应 response 拦截器
request.interceptors.response.use(
  (response) => { // 成功回调
    return response.data
  },
  (error) => { // 失败回调
    return Promise.reject(error)
  }
)
export default request
```

#### 3）在对应模块 `.js` 实现 api 请求方法
```javascript
import request from "@/utils/request";

// 查询全部部门数据
export const queryAllApi = () =>  request.get('/depts');

// 新增
export const addApi = (dept) =>  request.post('/depts', dept);

// 根据 ID 查询
export const queryByIdApi = (id) =>  request.get(`/depts/${id}`);

// 修改
export const updateApi = (dept) =>  request.put('/depts', dept);

// 删除
export const deleteByIdApi = (id) =>  request.delete(`/depts?id=${id}`);
```

#### 4）在对应 Vue 组件中调用 api 方法
```javascript
// 查询
const deptList = ref([])
const search = async () => {
  const result = await queryAllApi();
  if(result.code){
    deptList.value = result.data;
  }
}
```

### 3. ElementPlus 表单验证
#### 1）表单验证 ref 数据
```javascript
const rules = ref({
  name:[
    {required: true,message:'必填项',trigger:'blur'}, // trigger 事件监听
    {min:2,max:10,message:'长度在 2 - 10 之间',trigger:'blur'}
  ]
})
```

#### 2）设置校验参数
```javascript
const deptFormRef = ref();

if(!deptFormRef.value) return;
  deptFormRef.value.validate(async (valid) => {
    if(valid){ // 表单验证是否通过
      // 通过
    }else{
      // 不通过
    }
  })
```

#### 3）绑定校验规则 `rules` 和数据名称 `prop`
```html
<el-form :model="dept" :rules="rules" ref="deptFormRef"> <!-- 绑定校验规则和校验参数 -->
  <el-form-item label="部门名称" label-width="80px" prop="name"> <!-- 绑定数据名称 -->
    <el-input v-model="dept.name"/>
  </el-form-item>
</el-form>
```

#### 4）重置规则
```javascript
if(deptFormRef.value){
  deptFormRef.value.resetFields();
}
```

### 4. ElMessageBox 消息弹框
```javascript
const delById = async(id)=>{
  ElMessageBox.confirm("确认删除？","提示",
    { confirmButtonText:'确认',cancelButtonText:'取消',type:'warning'}
  ).then(async ()=>{
    const result = await deleteByIdApi(id);
    if(result.code){
      ElMessage.success("删除成功");
      search();
    }else{
      ElMessage.error(result.msg);
    }
  }).catch(()=>{
    ElMessage.info("取消删除");
  })
}
```

----------------------------------------

## 结语
----------------------------------------
我竟无言以对。

---------------------------------------