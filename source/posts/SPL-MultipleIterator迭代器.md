---
layout: post
title: SPL-MultipleIterator迭代器
date: '2016-04-17 22:47'
comments: true
customizeurl: SPL-MultipleIterator
tags:
  - php
  - SPL
abbrlink: 17389
---

- php合并多个数组,自定义键

```php
$idIter=new ArrayIterator(['001','002','003','004']);

$nameIter=new ArrayIterator(['Bob','Daisy','Jasmine','Ailsa']);

#MIT_KEYS_ASSOC以键排序,还有MIT_NEED_ANY,MIT_NEED_ALL,MIT_KEYS_NUMERIC选项.
$mit = new MultipleIterator(MultipleIterator::MIT_KEYS_ASSOC);

$mit->attachIterator($idIter,"id");

$mit->attachIterator($nameIter,"name");

foreach ($mit as $value) {

	print_r($value);

}

```

- 打印结果

<!-- more -->

```php
array(2) {      
  ["id"]=>           
  string(3) "001"     
  ["name"]=>         
  string(3) "Bob"    
}                    
array(2) {    
  ["id"]=>           
  string(3) "002"           
  ["name"]=>         
  string(5) "Daisy"  
}                    
array(2) {     
  ["id"]=>           
  string(3) "003"         
  ["name"]=>         
  string(7) "Jasmine"
}                    
array(2) {     
  ["id"]=>           
  string(3) "004"          
  ["name"]=>         
  string(5) "Ailsa"  
}

```
