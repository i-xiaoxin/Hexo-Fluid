---
title: Blog日志
date: 2022-10-13 21:09:02
tags: 
- Other
index_img: 
banner_img: /img/post_banner.webp
categories:
- Other
---

## i-xiaoxin.GitHub.io

几经辗转终于还是敲定了这个blog；版本迭代了N-1次，开篇记录变化吧！

### 2022年10月10日18点30分

引入waline comment: waline

```html
<div>
<hr>
<script src="https://unpkg.com/@waline/client@v2/dist/waline.js"></script> 
<link
  rel="stylesheet"
  href="https://unpkg.com/@waline/client@v2/dist/waline.css"
/>
<div id="waline"></div>
  <script>
    Waline({
      el: '#waline',
      serverURL: 'https://vercel-project-4d7haxk1c-i-xiaoxin.vercel.app',
    });
  </script>
```

以上为[waline](https://waline.js.org/)标准使用

### 2022年10月10日20点45分

 友链页 Links Page

```yaml
title: "🌐网站/Blog/ID",
intro: "📶信息简介",
link: "🔗https://host",
avatar: "🧩网站/Blog提供的原创链接"
```

### 2022年10月10日20点33分

抛开GitHub异域世界不谈，在挂上工具的环境下多次访问[站点](https://i-xiaoxin.github.io/)；即使配上[Hexo Fluid](https://hexo.fluid-dev.com/docs/)懒加载还是图片加载缓慢，因为使用的是墙纸jpg；于是尝试使用经济方案wbep；情况好转80%

### 2022年10月10日20点39分

>约束：
>
>- tags&&categories 配置项首字母大写🔠
>- index_img: /img/article1-10.webp 随机🎲
>- banner_img: /img/post_banner.webp ❓
>- To Do add 📈

### 2022年10月16日20点22分

首页💻端加速基本🉑️以了，📱端速度不太理想，图片也是重复的，最终还是先考虑低成本解决方案，取消index_img
