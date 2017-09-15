---
layout: post
title: mysql自动备份数据库并上传到百度云
date: '2016-03-13 21:12'
comments: true
customizeurl: mysql-backup-to-updalod-baidu
tags:
  - mysql
abbrlink: 13659
---

- 先写一个定时任务,每天凌晨执行:

```bash
crontab -e 
输入:2 1 * * * bash /root/mysqlBackup.sh
```

- 配置bypy(python版本大于2.6)

```bash
wget --no-check-certificate https://github.com/pypa/pip/archive/1.5.5.tar.gz
tar zvxf 1.5.5.tar.gz && cd pip-1.5.5/
/usr/local/python27/bin/python2.7 setup.py install
/usr/local/python27/bin/pip install requests
/usr/local/python27/bin/pip install bypy
/usr/local/python27/bin/bypy info #第一次运行会让你输入授权码对 pyby授权:访问提示的URL,获取授权码.pyby相关命令自行查找.
```

- mysqlBackup.sh内容:
<!-- more -->
```bash
#! /bin/sh
#1.backup mysql data
#2.auto upload to baiduCloud
filePath='/root/mysqlBackup'
fileName=$(date '+%Y%m%d')
file=$filePath"/"$fileName".sql"
/www/wdlinux/mysql-5.5.49/bin/mysqldump -uroot -p***** tableName > $file && /usr/local/python27/bin/bypy upload $file
```