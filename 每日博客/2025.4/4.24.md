# 我的学习日志：前端开发练习

## 前言
----------------------------------------
只想畏缩在自己的床上，什么也不想干😰  
我觉得有必要调整一下复习方针，不然容易白学。  
我确实不太爱复习的人，尽量每天抽出时间来复习昨天的内容，周末总体复习一下一个星期学的东西。  
先看看效果如何。

----------------------------------------

## 日程
----------------------------------------
早上写了一会前端，被mac的同步配置浪费时间了。  
现在晚上7点，先学到登录为止吧。  
感觉自己像个犯困的尸体一样无力。  
学到10点多，剩了不少，但是再学就要卡熄灯了，给复习留够时间，其实每天早上就应该复习好的。

----------------------------------------

## 学习记录
----------------------------------------
### 计算机组成原理
1. 储存系统基本概念
2. 存储器分类，性能指标
3. 主存储器基本储存（部分）

----------------------------------------

## 学习内容
----------------------------------------
### 省流
1. Vue watch监听
2. Element 自定义绑定
3. Element Layout布局
4. Element 文件上传

### 1. Vue watch监听
当数据源发生变化时，触发传入的回调：
```javascript
watch(() => employee.value.exprList, (newVal, oldVal) => { // 新/旧数值
  if (employee.value.exprList && employee.value.exprList.length > 0) {
    employee.value.exprList.forEach((item, index) => {
      item.begin = item.exprDate[0];
      item.end = item.exprDate[1];
    });
  }
}, { deep: true }); // 深度监听，当监听的对象为数组时，数组的项的变化会被监听到；默认是浅度监听，即只监听对象的引用变化
```

### 2. Element 自定义绑定
在行内指定数据的呈现形式：
```html
<el-table-column label="" width="180">
  <template #default="scope">
    <!-- 绑定图片 -->
    <img :src="scope.row.image" alt="" style="height: 40px;" />
    <!-- 自定义数据 -->
    {{ scope.row.gender == 1 ? '男' : '女' }}
    <span v-if="scope.row.job == 1">班主任</span>
    <span v-else-if="scope.row.job == 2">讲师</span>
    <span v-else-if="scope.row.job == 3">学工主管</span>
    <span v-else-if="scope.row.job == 4">教研主管</span>
    <span v-else-if="scope.row.job == 5">咨询师</span>
    <span v-else>其他</span>
    <!-- 嵌套其他组件 -->
    <el-button type="primary" size="small" @click="">编辑</el-button>
    <el-button type="danger" size="small" @click="">删除</el-button>
  </template>
</el-table-column>
```

### 3. Element Layout布局
Element采用24分栏，`col`在单列中的大小占比为`24/span`：
```html
<!-- gutter指定列间距 -->
<!-- span指定列宽占比 -->
<el-row :gutter="20"> 
  <el-col :span="12">
    <el-form-item label=""></el-form-item>
  </el-col>
  <el-col :span="12">
    <el-form-item label=""></el-form-item>
  </el-col>
</el-row>
```

### 4. Element 文件上传
`action`指定上传请求路径：
```html
<el-upload
  class="avatar-uploader"
  action="/api/upload" 
  :show-file-list="false"
  :on-success="handleAvatarSuccess"
  :before-upload="beforeAvatarUpload"
>
  <img v-if="employee.image" :src="employee.image" class="avatar" />
  <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
</el-upload>
```
```javascript
// 图片上传成功后触发
const handleAvatarSuccess = (response) => {
  employee.value.image = response.data;
};

// 文件上传之前触发
const beforeAvatarUpload = (rawFile) => {
  if (rawFile.type !== 'image/jpeg' && rawFile.type !== 'image/png') {
    ElMessage.error('只支持上传图片');
    return false;
  } else if (rawFile.size / 1024 / 1024 > 10) {
    ElMessage.error('只能上传10M以内图片');
    return false;
  }
  return true;
};
```

----------------------------------------

## 结语
----------------------------------------
表情包不够用了，有空搜罗一点。

---------------------------------------