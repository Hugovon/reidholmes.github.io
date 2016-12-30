---
published: true
layout:     post
title:      "NodeJs之Express"
date:       2016-11-18 11:22:22
categories:
---

<h3>express是什么?</h3>

<p><code>express</code> 是nodejs一个很著名的框架。 其地位就有点像<code>react</code>在js里面的地位。就算你不打算使用它，也可以学习下，取长补短。express的核心就是<strong>中间件</strong></p>
<ol>
<li>创建一个http服务器</li>
</ol>
<pre><code>var express = require('express');
var app = express();
app.use(function(req, res){
    res.send('hello express!');
});
app.listen(8001,function(){
    console.log('监听8001端口');
});
</code></pre>
ok 上述代码就实现功能一。默认监听的本地IP 网址:http://localhost 它对于一切返回值都返回了hello express!


定义一个 Web 应用实例，并且启动它。

```javascript
var express    = require('express');
var app        = express();

var port = process.env.PORT || 8080;

app.listen(port);
console.log('Magic happens on port ' + port);
```

---

定义一个路由

```javascript
var router = express.Router();

router.get('/', function(req, res) {
  res.send('<h1>Hello World</h1>');
});

app.use('/home', router);
```

---

中间件：对 HTTP 请求进行加工。

```javascript
router.use(function(req, res, next) {
  console.log('Thers is a requesting.');
  next();
});
```

