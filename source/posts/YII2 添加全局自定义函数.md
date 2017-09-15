---
layout: post
title: YII2 添加全局自定义函数
date: '2016-02-20 16:39'
comments: true
customizeurl: YII2-customize-function
tags:
  - php
  - YII2
abbrlink: 33118
---

## 方法一:

这种方法就是直接在入口文件web/index.php里面写函数，示例代码如下：

##### 全局函数
```php
function pr($var){
    //do something
}

(new yii\web\Application($config))->run();

```
也可以引入一个函数文件.

----------


## 方法二（推荐）:
<!-- more -->
这种方法主要是利用 composer 来实现，在 composer.json 文件里面添加如下代码：
````json
"autoload": {
    "files": [  
        "common/components/GlobalFunctions.php"
    ]
},

````

添加完之后,在common/components下添加文件GlobalFunctions.php,记得用终端在项目根目录下执行 composer update 命令

备注:因为只添加一个文件,我用了方法二,但是只看到`vendor\composer\autoload_files.php`文件的修改;

```php
return array(
    '2cffec82183ee1cea088009cef9a6fc3' => $vendorDir . '/ezyang/htmlpurifier/library/HTMLPurifier.composer.php',
    '2f22d0f2d4e1d504c0f839c579818375' => $baseDir . '/common/components/GlobalFunctions.php',
);
```

把composer update 更新的文件全部取消后手动更改 autoload_files 也是可以加载的

