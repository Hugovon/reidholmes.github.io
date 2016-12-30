---
published: true
layout:     post
title:      "react-technology-stack"
date:       2016-11-18 11:22:22
categories:
---

<h4>React 非常先进和强大，但是学习和实现成本都不低</h4>


<h5>JSX 语法</h5>

React 使用 JSX 语法，JavaScript 代码中可以写 HTML 代码。

    let myTitle = <h1>Hello, world!</h1>;
<h5>JSX 语法解释</h5>

（1）JSX 语法的最外层，只能有一个节点。

   
     // 错误

    let myTitle = <p>Hello</p><p>World</p>;</pre>

（2）JSX 语法中可以插入 JavaScript 代码，使用大括号。

    let myTitle = <p>{'Hello ' + 'World'}</p>

## Babel 转码器

JavaScript 引擎（包括浏览器和 Node）都不认识 JSX，需要首先使用 Babel 转码，然后才能运行。

React 需要加载两个库：React 和 React-DOM，前者是 React 的核心库，后者是 React 的 DOM 适配库。
    


    <script src="react.js"></script>
    <script src="react-dom.js"></script>
    <script src="babel.min.js"></script>
    <script type="text/babel">
    </script>

Babel 用来在浏览器转换 JSX 语法，如果服务器已经转好了，浏览器就不需要加载这个库。

**总的来说React属于前端技术栈，需要填的坑还有许多，如果真的要用的话，还是先用vuejs吧，毕竟能简单上手。等技术沉淀到一定程度的时候再说吧。**

最后引用前端界知名大神阮一峰的一段话共勉。“**面对技术的高速发展和百花齐放，我有时也感到疲倦烦躁。但是，每当看到它们带来的生产力的飞跃，让你一个人快速搞定前后端的全部开发时，就觉得这终究还是一条正确的道路。**”


## TODO ##
- React技术栈日后会在这里学习和更新。