---
title: Hexo Fluid 使用相关指南
date: 2022-09-21 17:04:34
tags: other
index_img: /img/example.jpg
banner_img: /img/post_banner.jpg
sticky: 100
categories:
- hexo
---

### ❤️使用指南

➡️New Markdown 可以执行命令来创建一篇新文章或者新的页面;当然也可以通过新建完成创建

### ❤️MD属性相关

> title: 这是标题
> date: 2022-09-21 17:04:34
> tags: other
> excerpt: 这是摘要
> index_img: /img/example.jpg
> banner_img: /img/post_banner.jpg
> sticky: 100
>
> contegories：目录

字段`tags`设置文章归属标签；自定义摘要可以使用字段`excerpt`进行设置，手动优先级高于自动设置；`sticky`属性影响文章的排序，数值越大，该文章越靠前，达到类似于置顶的效果，其他未设置的文章依然按默认排序；`index_img` 属性决定了文章在首页显示的封面图，置空时选择默认；`banner_img` 属性文章页配置顶部大图

### ❤️ 脚注

示例如下：

```markdown
这是一句话[^1]
[^1]: 这是对应的脚注

正文

## 参考
[^1]: 参考资料1
[^2]: 参考资料2
```

### ❤️Tag 插件

#### 便签MD语法

```markdown
{% note success %}
文字 或者 `markdown` 均可
{% endnote %}
```

#### 便签 HTML形式

```html
<p class="note note-primary">标签</p>
```

属性选项：primary secondary success danger warning info light

> **注意：**
>
> 使用时 `{% note primary %}` 和 `{% endnote %}` 需单独一行，否则会出现问题

#### 行内标签

在 markdown 中加入如下的代码来使用 Label：

```markdown
{% label primary @text %}
```

或者使用 HTML 形式：

```html
<span class="label label-primary">Label</span>
```

可选 Label：

primary default info success warning danger

> 注意：
>
> 若使用 `{% label primary @text %}`，text 不能以 @ 开头

### 按钮

你可以在 markdown 中加入如下的代码来使用 Button：

```markdown
{% btn url, text, title %}
```

或者使用 HTML 形式：

```html
<a class="btn" href="url" title="title">text</a>
```

url：跳转链接
text：显示的文字
title：鼠标悬停时显示的文字（可选）
