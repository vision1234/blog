---
title: zufang
tags: 租房
categories: 爬虫
---

## 项目背景：

因作者最近想换房子住，但是工资微薄只能等有优惠的特价房字，又不想每天通过租房app去找一遍，想让他自己推送到邮箱，下班路上看。

## 功能介绍：

本项目是通过爬取某家和某爱某家的网站的租房信息页，读取照片、标题、价格等关键信息并通过邮件推送的方式实现绕过烦人的中介找到心仪的房子。

## 步骤：

### 1.下载代码到本地

```
git pull https://github.com/vision1234/zufang_crawler.git
```

### 2.使用浏览器进入某家和某爱某家的网站，设置好筛选条件，复制url替换对应.py上的url

```
https://bj.5i5j.com/zufang/subway/ss2159/b4000e7300h110l70r2r3t1u1/ 
https://bj.lianjia.com/ditiezufang/li656s43137676/rt200600000001l1l2ra2ra3ra4brp4000erp7300/
```

### 3.修改send_mail.py发件人邮箱和授权码，授权码可以从邮箱运营商网站获取

```python
my_sender = 'xxxxxx@qq.com'  # 发件人邮箱账号
my_pass = "xxxxxxx"  # 口令
```

![mail_sc.png](https://github.com/vision1234/images/blob/master/blog_img/mail_sc.png)

### 4.昵称改不改都可以

```python
msg['From'] = formataddr(["666", my_sender])  # 括号里的对应发件人邮箱昵称、发件人邮箱账号
msg['To'] = formataddr(["666", u])  # 括号里的对应收件人邮箱昵称、收件人邮箱账号
```

### 5.修改main.py的收件人邮箱

```python
send_mail.mail("xxxxxx@qq.com", mail_data)
```

![mail_sc.png](https://github.com/vision1234/images/blob/master/blog_img/main_sc.png)

### 6.在命令行安装python库（python3.8）

```shell
pip install -r requirements.txt
```

低版本的话，需要依次手动安装arrow、lxml、requests这三个库

### 7.运行main.py

```python
python mian.py
```

### 8.可以在crontab 中添加定时任务，每天定时跑一遍

```
0 18 * * * cd /home/user_name/workspace/zufang_crawler;python mian.py;
```