---
layout: post
title: 添加 hexo yilia 主题的文章阅读量
date: '2017-10-26 17:25'
comments: true
customizeurl: hexo-yilia-page-view
tags:
  - hexo
  - yilia
abbrlink: 58129
---

根据此篇博客([点击查看](http://www.icafebolger.com/hexo/hexopostcount.html)) 配置出自己的博客阅读量,里面介绍了如何配置开通 `leancloud` 应用

我说下自己如何配置 `yilia` 显示文章浏览量的.


1. 首先在 `yilia` 主题下修改 `_config.yml` 添加如下配置信息
    
```
# 添加浏览量
leancloud_visitors:
  enable: true
  app_id: ************** 
  app_key: ************

#添加一下js插件的 CDN地址
js_cdn:
  jquery: https://cdn.bootcss.com/jquery/3.2.1/jquery.min.js
  av : //cdn1.lncld.net/static/js/2.5.0/av-min.js
```

2. 找到主题下 `layout\_parial\post` 找到 `title.ejs` 文件,添加显示阅读量的html代码:

```html
<% if (post.link){ %>
  <h1 itemprop="name">
    <a class="<%= class_name %>" href="<%- url_for(post.link) %>" target="_blank" itemprop="url"><%= post.title %></a>
  </h1>
<% } else if (post.title){ %>
  <% if (index){ %>
    <h1 itemprop="name">
      <a class="<%= class_name %>" href="<%- url_for(post.path) %>"><%= post.title %></a>
    </h1>
  <% } else { %>
    <h1 class="<%= class_name %>" itemprop="name">
      <%= post.title %>
    </h1>
  <% } %>
<% } %>
#上面是 yilia 主题的标题 下面是我自己添加的阅读量 html 代码
# span 给的 id 是文章的 url 作为唯一的 key 值
<% if (theme.leancloud_visitors.enable){ %>
  <a href="javascript:;" class="archive-article-date">
      <i class="icon-smile icon"></i> 阅读数:<span id="<%- url_for(post.path) %>" class="pageViews">0</span>次
  </a>
<% } %>

```
<!-- more -->
3.添加 `js` 代码, 在`layout\_parial` 找到 `after.footer.ejs` 添加如下代码:

```html
<script>
    var yiliaConfig = {
        mathjax: <%=theme.mathjax%>,
        isHome: <%=is_home()%>,
        isPost: <%=is_post()%>,
        isArchive: <%=is_archive()%>,
        isTag: <%=is_tag()%>,
        isCategory: <%=is_category()%>,
        open_in_new: <%=theme.open_in_new%>,
        toc_hide_index: <%=theme.toc_hide_index%>,
        root: "<%=config.root%>",
        innerArchive: <%=theme.smart_menu.innerArchive ? true : false%>,
        showTags: <%=(theme.slider && theme.slider.showTags) ? true : false%>
    }
</script>

#上面是 yilia 主题配置自带 下面是我自己添加的js代码
<script src="<%- theme.js_cdn.jquery %>"></script>

<% if (theme.leancloud_visitors.enable){ %>
//这是自定一个js文件,放在 layout\_parial\post 目录 leancloud.ejs中
<%- partial('post/leancloud') %> 
<% } %>

#下面是 yilia 主题配置自带
<%- partial('script') %>

<% if (theme.mathjax){ %>
<%- partial('mathjax') %>
<% } %>
```
`leancloud.ejs` 内容:

```html
<script src="<%- theme.js_cdn.av %>"></script>
<script type="text/javascript"> 
if(<%- theme.leancloud_visitors.enable %>){
    var leancloud_app_id  = '<%- theme.leancloud_visitors.app_id %>';
    var leancloud_app_key = '<%- theme.leancloud_visitors.app_key %>';

    AV.init({
        appId: leancloud_app_id,
        appKey: leancloud_app_key
    });

    var pageViewsLength = $(".pageViews").length;

    var isIndex = $(".pageViews").length > 1 ?true:false;

    function showTime() {
        var Counter = AV.Object.extend("Counter");
        if(isIndex){
            $(".pageViews").each(function(){
                showPageViewsNum($(this),Counter);
                addPageViewsNum($(this));
            });
        }else{
            showPageViewsNum($(".pageViews"),Counter);
            addPageViewsNum($(".pageViews"));
        }
    }

    //显示阅读量
    function showPageViewsNum(ele,Counter){
        var query = new AV.Query("Counter");
        var url = ele.attr('id').trim();
            query.equalTo("words", url);
            query.count().then(function (number) {
                $(document.getElementById(url)).text(number?  number : '0');
            }, function (error) {
                 $(document.getElementById(url)).text('0');
            });
    }

    //追加阅读量
    function addPageViewsNum(ele){
        var url = ele.attr('id').trim();
        var Counter = AV.Object.extend("Counter");
        var query = new Counter;
        query.save({
            words: url
        }).then(function (object) {});
    }

    if(pageViewsLength){ //此处判断等于 1 在执行 可去除循环
        showTime();
    }
}
</script>
```

然后就是熟悉的 `hexo g` `hexo s` 查看,记得要在 `leancloud` [安全中心] [Web 安全域名] 设置2个安全域名,一个是正式环境的域名,一个是调试的域名.不然会报错.
预览查看: [http://easydo.work](http://easydo.work)
