```
title: 图片网站
tags: web
categories: 前端
```

### 项目地址

```
https://github.com/vision1234/images_website
```

### 使用说明

1. 左上角三条杠点击出现导航

2. 搜索框输入结束回车开始搜索

3. 点击首页、搜索结果页、分类明细页缩略图即可查看大图

4. 大图左右滑动可在本页范围内切换图片

5. 点击大图回到缩略图页

6. 右键菜单保存下载图片

### 代码说明

1. 在项目根目录创建config.py  

2. 写入mongo和redis配置项  

```
username = '' # mongo用户名  
password = '' # mongo密码  
r_password = '' # redis密码  
hostname = "" # mongo和redis的地址  
port =     # mongo端口  
r_port =   # redis端口  
r_db =    # redis库序号  
database = "" # mongo库  
coll = ''  # mongo集合  
r_cate_key = '' # redis分类集合，list  
r_tag_key = '' # redis标签集合，set,暂时无用  
```

3. 启动上传服务

```
python upload.py  
```

4. 启动应用

```
python app.py  
```
