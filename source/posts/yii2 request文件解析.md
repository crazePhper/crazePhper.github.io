---
layout: post
title: YII2 request文件解析
date: '2016-02-28 20:37'
comments: true
customizeurl: YII2-request
tags:
  - php
  - YII2
abbrlink: 25790
---

###### 1. 在YII2项目中经常用到下面这些方法:

方法 | 解释
---|---
Yii::$app->request->get($key,$defaultValue) | GET方式获取key的值,若key空,则key为$defaultValue
Yii::$app->request->post($key,$defaultValue) | POST方式获取key的值,若key空,则key为$defaultValue
Yii::$app->request->referrer | 返回载入当前页面的来源URL
Yii::$app->request->getHostInfo() | 返回当前URL$_SERVER['HTTP_HOST'],$_SERVER['SERVER_NAME']
Yii::$app->request->setHostInfo() | 设置HostInfo
[![](/img/2032996154.jpg)](/img/2032996154.jpg)
这个就写三个例子吧,看了一下在`\vendor\yiisoft\yii2\web\Request.php`中,有很多方法是封装的不错的.
###### 2.高级一点的方法
<!-- more -->
GET参数操作:

    Yii::$app->request->getQueryParams()	//获取所有URL参数(GET)
    Yii::$app->request->setQueryParams()    //设置URL GET参数
    Yii::$app->request->getQueryParam()	//读取一个GET参数

POST参数操作:

    Yii::$app->request->getBodyParams()	//获取所有URL参数(POST)
    Yii::$app->request->setBodyParams()    //设置URL POST参数
    Yii::$app->request->getBodyParam()	//读取一个POST参数

###### 3.核心验证
csrfToken也是在这里设置,验证的. ` getCsrfToken(),loadCsrfToken()`等等方法,可以自己查看测试一下
判断是不是ajax:`getIsAjax()`,
是不是pajx:`getIsPjax()`,
还有很多方法名称只要略懂英文就能知道什么意思.

