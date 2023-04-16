---
title: CSS常见选择器
date: 2023-04-15 18:36:00
categories: 
- web前端
- css
---

# 简单选择器

* 元素选择器(type selectors)：使用元素的名称

  > div { color: red }

* 类选择器(class selectors)：使用类名

  > .box { color: red }

* id选择器(id selectors)：使用id

  > #main { color: red }





# 属性选择器

* [attr]：拥有某一个属性

  > [title] { color: blur; }

* [attr=val]：属性等于某个值

  > [title=box] { color: red; }

* [attr*=val]：属性值包含某一个值val
* [attr^=val]：属性值以val开头
* [attr$=val]：属性值以val结尾
* [attr|=val]：属性值等于val或以val开头后面紧跟连字符
* [attr~=val]：属性值包含val，如果有其他值必须以空格和val分割





# 后代选择器

* 所有后代选择器：选择器间以空格分割

  > .box span { color: red; }

* 直接子代选择器：选择器间以>分割

  > .box > span { color: red; } 





# 兄弟选择器

* 相邻兄弟选择器：使用 + 连接

  > .one + div { color: red; }

* 普遍兄弟选择器：使用 ~ 连接

  > .one ~ div { color: red; }





# 选择器组

* 交集选择器：需要同时符合两个选择器条件，选择器紧密连接

  > div.one { color: red; }

* 并集选择器：符合一个选择器条件即可，选择器以逗号分割

  > .one, .two { color: red; }





# 伪类

## 概述

伪类（pseudo-classes）是选择器的一种，用于选择处于特定状态的元素。

常见的伪类有：

* 动态伪类，如 :link
* 目标伪类，如 :target
* 语言伪类，如 :lang()
* 元素状态伪类，如 :checked
* 结构伪类，如 :nth-child()
* 否定伪类，:not()



所有伪类：https://developer.mozilla.org/zh-CN/docs/Web/CSS/Pseudo-classes





## 动态伪类

常用动态伪类：

* a:link 未访问的链接
* a:visited 已访问的链接
* a:hover 鼠标挪动到链接上
* a:active 激活的链接（鼠标在链接上长按住未松开）
* :focus 当前拥有输入焦点的元素

动态伪类编写顺序建议为  :link 、:visited、:focus、:hover、:active



## 伪元素

常用的伪元素：

* ::first-line  针对首行文本设置属性
* ::first-letter  针对首字母设置属性
* ::before  在一个元素的内容前插入其他内容
* ::after  在一个元素的内容后插入其他内容

也可以使用一个冒号，但是为了区分伪元素和伪类，建议使用两个冒号。

```css
.box::before {
    content: "123";
    color: red;
}

.box::after {
    content: "321";
    color: blue;
}
```



