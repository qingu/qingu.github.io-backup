---
layout: post
title: 新博客模板
date: 2023-05-06
tags: jekyll
---

使用基于Jekyll的leopardpan模板创建新博客界面代替原先的Octopress模板。

## vika图片不能正常显示问题

在页面布局`_layout/post.html`中`<head> </head>`之间加上

```html
 <meta name="referrer" content="no-referrer" />
```
