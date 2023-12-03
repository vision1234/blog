---
title: 后台管理
tags: 

- vue3
- elementplus
- js
categories: 前端
date: 2023/12/03 10:40:01
---

# vue3+elementplus+js的后台管理系统

## 项目地址
```
https://github.com/vision1234/image_manage
```
## 项目启动前需要手动在根目录创建一个 config.js

```
const config = {
    apiBaseUrl: process.env.VUE_APP_API_BASE_URL || 'http://xxxxxxxx/'
  };

  export default config;
```

## 项目安装

```
npm install
```

### 本地启动调试

```
npm run serve
```

### 打包生成静态文件

```
npm run build
```

### 捕捉潜在的错误、风格问题或不规范的代码

```
npm run lint
```

### 部署

```
将build的静态文件dist目录复制到某个路径，然后配置nginx
server {
    listen 80;
    server_name your_domain.com; # 替换为你的域名或服务器 IP

    root /path/to/your/vue/project/dist; # 替换为你 Vue 项目的 dist 目录路径

    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # 其他配置...
}
注意要给到nginx用户dist下所有文件读取权限，并且不能放到/home目录下
```
