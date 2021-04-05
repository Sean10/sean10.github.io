---
title: hexo主题自定义--修改indigo tag显示
subtitle: tech
date: 2019-01-17 20:57:48
updated: 2021-01-16 21:57:48
tags: [hexo, js]
categories: [专业]
---


更换hexo-theme-indigo的时候，发现作者也是个简约主义啊，功能足够就行了，不需要更新，hhh。

# 增加引用
首先，将next主题中一直用的addthis统计与分享功能引入。

官网上很简单，就是一行html和js，不过似乎在我使用时，可能排版上与js中携带的addthis模块排版冲突了，导致没能显示出来


# 修改tag显示

主要的问题在于, 当写的Tag总计多了时, 观察不到哪些`tag`的文章比较多

当时初步想的是`hexo-tag-cloud`或者是雷达图效果.

不过想了下, 这方面如果更换, 可能css也得换, 修改较多.

试试看能不能直接在tag名字后面加一个统计数字.

这个`API`里理论上应该有提供

翻了下代码, 感觉`layout/tags.ejs`有点像显示的代码, 翻到个函数`is_tag`, 全局没搜到, 那应该代表着这里的一些信息是从hexo或者`hexo`的插件里获取到的.

的确,根据[Helpers \| Hexo](https://hexo.io/docs/helpers.html), 在这里找到了这个函数.

对, 和页面上的`id`, `class`比对了下, 的确这里是正文中的`tag`和`post`显示的位置.

``` js
<%
        if(is_tag()) { %>
            <div class="waterfall">
          <% page.posts.each(function(post){ %>

            <%- partial('_partial/archive', {post: post, date_format: config.date_format}) %>

        <% }) %>
            </div>
        <% } else {
            site.tags.each(function(tag){
                if(tag.length){
            %>

            <h3 class="archive-separator" id="tag-<%=tag.name %>"><%=tag.name %></h3>

            <div class="waterfall">
            <% tag.posts.each(function(post){  %>

                <%- partial('_partial/archive', {post: post, date_format: config.date_format}) %>

            <% }) %>
            </div>
            <% }
            })
        }
    %>
```

开启`debug`模式, `hexo s --debug`

实验了下, 还是`header`处增加统计数字更明显一些. `tags-bar.ejs`文件

## 增加tag统计角标
``` js
<a href="<%- url_for(tag.path) %>" style="-webkit-order:<%= order%>;order:<%= order%>" class="tags-list-item waves-effect waves-button waves-light<% if(is_current(tag.path)){%> active<%}%>"><%-tag.name%></a>
```

增加了`sup`选项, 加了个角标,看起来还可以
``` js
<a href="<%- url_for(tag.path) %>" style="-webkit-order:<%= order%>;order:<%= order%>" class="tags-list-item waves-effect waves-button waves-light<% if(is_current(tag.path)){%> active<%}%>"><%-tag.name%><sup><%-tag.length%></sup></a>
```

## 增加排序(未完成)

相对来说, 还是不太直观. 看下面这段代码, 看起来是在指定Tag在这个顺序中的位置.

``` js
var options = [];

            (type === 'tags' ? site.tags : site.categories).each(function(o) {
                if(o.posts.length) {
                    options.push(o)
                }
            })

            var index = _.findIndex(options, function(o) {
                return is_current(o.path)
            })
            var len = options.length
            var order = 0
            options.forEach(function(tag, i){

                if(index <= 1) {
                    order = i
                } else {

                    if( i < index - 1) {
                        order = len - (index - 1) + i
                    } else {
                        order = i - (index - 1)
                    }
                }

            %>
```

`webkit-order`的作用是什么呢... 理解错了, 这个其实还是为了布局用的.

# 问题
## hexo渲染时, Markdown文本中如果出现`\{\{\}\}`, 又没有被转义, 是会转义失败, 报下面的错误的.

```
Template render error: (unknown path) [Line 208, Column 5]
```