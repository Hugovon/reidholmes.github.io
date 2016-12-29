---
published: true
layout:     post
title:      "webpack模块化"
date:       2016-10-21 17:15:00
categories:
---
Webpack 是德国开发者 Tobias Koppers 开发的模块加载器兼打包工具，在webpack中，它能把各种资源，例如JS（含JSX）、coffee、样式（含less/sass）、图片等都作为模块来使用和处理。因此, Webpack 当中 js 可以引用 css, css 中可以嵌入图片 dataUrl。<br>对应各种不同文件类型的资源, Webpack 有对应的模块 loader比如vue用的是<code>vue-loader</code>当然这是后话，在后面我们再来说。
<p>请看下图：<br><img src="/Misc-Img/webpack-base.png" alt="webpack"></p>
<p>官网查看：<a href="https://github.com/webpack/webpack" target="_blank" rel="external">https://github.com/webpack/webpack</a></p>
<h3>1. 安装</h3>
<p>首先要安装 Node.js， Node.js 自带了软件包管理器 npm，Webpack 需要 Node.js v0.6 以上支持，建议使用最新版 Node.js。
用 npm 安装 Webpack：</P>
<pre><code class='language-javascript'>npm install -g webpack</code></pre>

<h3>2. 基本使用</h3>

<p>首先创建一个静态页面 index.html 和一个 JS 入口文件 entry.js：</p>

    <!-- index.html -->
    <html>
    <head>
      <meta charset="utf-8">
    </head>
    <body>
      <script src="bundle.js"></script>
    </body>
    </html>
    // entry.js
    document.write('It works.')
然后编译 entry.js 并打包到 bundle.js：

`$ webpack entry.js bundle.js`

打包过程会显示日志：

    `Hash: e964f90ec65eb2c29bb9Version: webpack 1.12.2Time: 54ms
    Asset     Size  Chunks             Chunk Namesbundle.js  1.42 kB       0  [emitted]  main[0] ./entry.js 27 bytes {0} [built]`
用浏览器打开 index.html 将会看到 It works. 。

接下来添加一个模块 module.js 并修改入口 entry.js：
    <pre>// module.js
    module.exports = 'It works from module.js.</pre>
<pre>// entry.js
document.write('It works.')
document.write(require('./module.js')) // 添加模块</pre>
重新打包 `webpack entry.js bundle.js` 后刷新页面看到变化 `It works.It works from module.js.`
<pre>Hash: 279c7601d5d08396e751
Version: webpack 1.12.2
Time: 63ms
    Asset     Size  Chunks             Chunk Names
bundle.js  1.57 kB       0  [emitted]  main
   [0] ./entry.js 66 bytes {0} [built]
   [1] ./module.js 43 bytes {0} [built]</pre>

<h3>3. 配置文件</h3>

Webpack 在执行的时候，除了在命令行传入参数，还可以通过指定的配置文件来执行。默认情况下，会搜索当前目录的 webpack.config.js 文件，这个文件是一个 node.js 模块，返回一个 json 格式的配置信息对象，或者通过 --config 选项来指定配置文件。

继续我们的案例，在根目录创建 `package.json` 来添加` webpack` 需要的依赖：
<pre>{
  "name": "webpack-example",
  "version": "1.0.0",
  "description": "A simple webpack example.",
  "main": "bundle.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "webpack"
  ],
  "author": "zhaoda",
  "license": "MIT",
  "devDependencies": {
    "css-loader": "^0.21.0",
    "style-loader": "^0.13.0",
    "webpack": "^1.12.2"
  }
}</pre>
如果没有写入权限，请尝试如下代码更改权限

<pre>chflags -R nouchg .
sudo chmod  775 package.json</pre>
别忘了运行 `npm install`。

然后创建一个配置文件 `webpack.config.js`：
<pre>var webpack = require('webpack')

module.exports = {
  entry: './entry.js',
  output: {
    path: __dirname,
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {test: /\.css$/, loader: 'style!css'}
    ]
  }
}</pre>
同时简化` entry.js `中的 `style.css `加载方式：
    <pre>require('./style.css')</pre>
最后运行 `webpack`，可以看到 webpack 通过配置文件执行的结果和上一章节通过命令行 `webpack entry.js bundle.js --module-bind 'css=style!css'` 执行的结果是一样的。
<h3>4. 故障处理</h3>
Webpack 的配置比较复杂，很容出现错误，下面是一些通常的故障处理手段。

一般情况下，webpack 如果出问题，会打印一些简单的错误信息，比如模块没有找到。我们还可以通过参数 `--display-error-details` 来打印错误详情。
<pre>$ webpack --display-error-details

Hash: a40fbc6d852c51fceadb
Version: webpack 1.12.2
Time: 586ms
    Asset     Size  Chunks             Chunk Names
bundle.js  12.1 kB       0  [emitted]  main
   [0] ./entry.js 153 bytes {0} [built] [1 error]
   [5] ./module.js 43 bytes {0} [built]
    + 4 hidden modules

ERROR in ./entry.js
Module not found: Error: Cannot resolve 'file' or 'directory' ./badpathmodule in /Users/zhaoda/data/projects/webpack-handbook/examples
resolve file
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule doesn't exist
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.webpack.js doesn't exist
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.js doesn't exist
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.web.js doesn't exist
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.json doesn't exist
resolve directory
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule doesn't exist (directory default file)
  /Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule/package.json doesn't exist (directory description file)
[/Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule]
[/Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.webpack.js]
[/Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.js]
[/Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.web.js]
[/Users/zhaoda/data/projects/webpack-handbook/examples/badpathmodule.json]
 @ ./entry.js 3:0-26</pre>
Webpack 的配置提供了 `resolve` 和 `resolveLoader` 参数来设置模块解析的处理细节，`resolve` 用来配置应用层的模块（要被打包的模块）解析，`resolveLoader` 用来配置 `loader `模块的解析。

当引入通过 npm 安装的 node.js 模块时，可能出现找不到依赖的错误。Node.js 模块的依赖解析算法很简单，是通过查看模块的每一层父目录中的 `node_modules` 文件夹来查询依赖的。当出现 Node.js 模块依赖查找失败的时候，可以尝试设置 `resolve.fallback` 和 `resolveLoader.fallback` 来解决问题。
<pre>module.exports = {
  resolve: { fallback: path.join(__dirname, "node_modules") },
  resolveLoader: { fallback: path.join(__dirname, "node_modules") }
};</pre>
Webpack 中涉及路径配置最好使用绝对路径，建议通过 `path.resolve(__dirname, "app/folder") `或 `path.join(__dirname, "app", "folder")` 的方式来配置，以兼容 Windows 环境。


<h3>5. 使用loader</h3>

<p>很多模块打包工具只是针对js文件，而webpack的强大之处在于将模块的概念进行了扩展，认为一切静态文件都是模块，包括css、html模板、字体、CoffeeScript等等。虽然webpack本身依然是只能够处理js文件，但是通过一系列的loader，就可以处理其它文件了。</p>

<p>下面以<code class='language-javascript'>css-loader</code>和<code class='language-javascript'>style-loader</code>为例，演示如何打包样式文件。首先执行如下命令安装依赖模块：</p>

<pre><code class='language-javascript'>npm install css-loader style-loader --save-dev</code></pre>

<p>然后在<code class='language-javascript'>app</code>目录下新建<code class='language-javascript'>style.css</code>文件，内容如下：</p>

<pre><code class='language-javascript'>body {
    background: red;
}</code></pre>

<p>然后修改<code class='language-javascript'>main.js</code>如下：</p>

<pre><code class='language-javascript'>require('./mymodule.js')();
require('style!css!./style.css');</code></pre>

<p>因为webpack不能够直接处理css文件，因此在<code class='language-javascript'>require</code>语句中需要指明需要的loader，一个文件可以经由多个loader依次处理，loader与loader之间，以及loader与文件名之间用<code class='language-javascript'>!</code>分隔。在这个例子中，也可以看出，如果使用了多个loader的话，数据流向是从右向左的，也就是从<code class='language-javascript'>style.css</code>开始，依次经过<code class='language-javascript'>css-loader</code>和<code class='language-javascript'>style-loader</code>。</p>

<p>但是假如有多个css文件的话，每个<code class='language-javascript'>require</code>语句都需要加上loader说明，很不方便，因此可以在<code class='language-javascript'>webpack.config.js</code>文件中进行配置，配置如下：</p>

<pre><code class='language-javascript'>loaders: [{
    test: /\.css$/,
    loader: 'style!css'
}]

// or

loaders: [{
    test: /\.css$/,
    loaders: ['style', 'css']
}]</code></pre>


<h3>6. 外部依赖</h3>

<p>现在假如该例子中需要用到angular，首先在<code class='language-javascript'>index.html</code>中通过<code class='language-javascript'>&lt;script&gt;</code>标签引入angular库，然后修改<code class='language-javascript'>mymodule.js</code>如下：</p>

<pre><code class='language-javascript'>var angular = require('angular');
angular.module('MyModule', []);</code></pre>

<p>此时如果执行<code class='language-javascript'>webpack</code>命令会报如下错误：</p>

<pre><code class='language-javascript'>ERROR in ./mymodule.js
Module not found: Error: Cannot resolve module 'angular' in /xxx/xxx/app
 @ ./mymodule.js 1:14-32</code></pre>

<p>这是因为webpack无法解析angular依赖模块，此时需要在配置文件中对外部依赖进行配置：</p>

<pre><code class='language-javascript'>externals: {
    'angular': true
}</code></pre>

<p>更多信息参考<a href="http://webpack.github.io/docs/configuration.html#externals">configuration#externals</a>。</p>

<h3>7. 输出类型</h3>

<p>现在假如我们希望打包后的文件作为一个单独的库，并且遵循AMD规范可以被被requirejs来使用，可以修改配置文件如下：</p>

<pre><code class='language-javascript'>output: {
    filename: 'app.js',
    library: 'app',
    libraryTarget: 'amd'
}</code></pre>

<p>此时输出的<code class='language-javascript'>app.js</code>结构如下：</p>

<pre><code class='language-javascript'>define("app", ["angular"], function( /* ... */ ) {
    /* ... */
});</code></pre>

<p>通过配置<code class='language-javascript'>output.libraryTarget</code>，可以自定义输出的模块类型，包括AMD，CommonJS，变量等多种输出类型。具体可以参考<a href="http://webpack.github.io/docs/configuration.html#output">configuration#output</a>。</p>

<h3>8. 多文件</h3>

<p>现在假如项目目录结构如下：</p>

<pre><code class='language-javascript'>/app
  |--components.js
  |--index.html
  |--main.js
  |--mymodule.js</code></pre>

<p>其中<code class='language-javascript'>mymodule.js</code>被<code class='language-javascript'>main.js</code>和<code class='language-javascript'>components.js</code>所使用。假如我们希望<code class='language-javascript'>main.js</code>输出为<code class='language-javascript'>app.js</code>，而<code class='language-javascript'>components</code>输出为<code class='language-javascript'>app.components.js</code>，则可以修改配置文件如下：</p>

<pre><code class='language-javascript'>entry: {
    app: './main.js',
    'app.coomponents': './components.js'
},
output: {
    filename: '[name].js'
}</code></pre>

<h3>9. 插件</h3>

插件可以完成更多 loader 不能完成的功能。

插件的使用一般是在 webpack 的配置信息 plugins 选项中指定。

Webpack 本身内置了一些常用的插件，还可以通过 npm 安装第三方插件。

接下来，我们利用一个最简单的 `BannerPlugin` 内置插件来实践插件的配置和运行，这个插件的作用是给输出的文件头部添加注释信息。

修改` webpack.config.js，`添加` plugins`：
<pre>var webpack = require('webpack')

module.exports = {
  entry: './entry.js',
  output: {
    path: __dirname,
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {test: /\.css$/, loader: 'style!css'}
    ]
  },
  plugins: [
    new webpack.BannerPlugin('This file is created by zhaoda')
  ]
}</pre>
然后运行 `webpack`，打开 `bundle.js`，可以看到文件头部出现了我们指定的注释信息：
<pre>/*! This file is created by zhaoda */
/******/ (function(modules) { // webpackBootstrap
/******/  // The module cache
/******/  var installedModules = {};
// 后面代码省略</pre>
<h3>10. 开发环境</h3>

当项目逐渐变大，`webpack `的编译时间会变长，可以通过参数让编译的输出内容带有进度和颜色。

<pre>$ webpack --progress --colors</pre>
如果不想每次修改模块后都重新编译，那么可以启动监听模式。开启监听模式后，没有变化的模块会在编译后缓存到内存中，而不会每次都被重新编译，所以监听模式的整体速度是很快的。

<pre>$ webpack --progress --colors --watch</pre>
当然，使用 webpack-dev-server 开发服务是一个更好的选择。它将在 [localhost:8080](localhost:8080) 启动一个 express 静态资源 web 服务器，并且会以监听模式自动运行 webpack，在浏览器打开 [http://localhost:8080/](http://localhost:8080/) 或 http://[localhost:8080/webpack-dev-server/](localhost:8080/webpack-dev-server/) 可以浏览项目中的页面和编译后的资源输出，并且通过一个 socket.io 服务实时监听它们的变化并自动刷新页面。

<pre># 安装
$ npm install webpack-dev-server -g
# 运行
$ webpack-dev-server --progress --colors</pre>

# 番外笔记

<h4 id="commonjs">CommonJS</h4>
<p>&#x670D;&#x52A1;&#x5668;&#x7AEF;&#x7684; Node.js &#x9075;&#x5FAA; <a href="http://wiki.commonjs.org/wiki/CommonJS" target="_blank">CommonJS&#x89C4;&#x8303;</a>&#xFF0C;&#x8BE5;&#x89C4;&#x8303;&#x7684;&#x6838;&#x5FC3;&#x601D;&#x60F3;&#x662F;&#x5141;&#x8BB8;&#x6A21;&#x5757;&#x901A;&#x8FC7; <code>require</code> &#x65B9;&#x6CD5;&#x6765;&#x540C;&#x6B65;&#x52A0;&#x8F7D;&#x6240;&#x8981;&#x4F9D;&#x8D56;&#x7684;&#x5176;&#x4ED6;&#x6A21;&#x5757;&#xFF0C;&#x7136;&#x540E;&#x901A;&#x8FC7; <code>exports</code> &#x6216; <code>module.exports</code> &#x6765;&#x5BFC;&#x51FA;&#x9700;&#x8981;&#x66B4;&#x9732;&#x7684;&#x63A5;&#x53E3;&#x3002;</p>
<pre><code class="lang-js"><span class="hljs-built_in">require</span>(<span class="hljs-string">&quot;module&quot;</span>);
<span class="hljs-built_in">require</span>(<span class="hljs-string">&quot;../file.js&quot;</span>);
exports.doStuff = <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{};
<span class="hljs-built_in">module</span>.exports = someValue;
</code></pre>
<p>&#x4F18;&#x70B9;&#xFF1A;</p>
<ul>
<li>&#x670D;&#x52A1;&#x5668;&#x7AEF;&#x6A21;&#x5757;&#x4FBF;&#x4E8E;&#x91CD;&#x7528;</li>
<li><a href="https://www.npmjs.com" target="_blank">NPM</a> &#x4E2D;&#x5DF2;&#x7ECF;&#x6709;&#x5C06;&#x8FD1;20&#x4E07;&#x4E2A;&#x53EF;&#x4EE5;&#x4F7F;&#x7528;&#x6A21;&#x5757;&#x5305;</li>
<li>&#x7B80;&#x5355;&#x5E76;&#x5BB9;&#x6613;&#x4F7F;&#x7528;</li>
</ul>
<p>&#x7F3A;&#x70B9;&#xFF1A;</p>
<ul>
<li>&#x540C;&#x6B65;&#x7684;&#x6A21;&#x5757;&#x52A0;&#x8F7D;&#x65B9;&#x5F0F;&#x4E0D;&#x9002;&#x5408;&#x5728;&#x6D4F;&#x89C8;&#x5668;&#x73AF;&#x5883;&#x4E2D;&#xFF0C;&#x540C;&#x6B65;&#x610F;&#x5473;&#x7740;&#x963B;&#x585E;&#x52A0;&#x8F7D;&#xFF0C;&#x6D4F;&#x89C8;&#x5668;&#x8D44;&#x6E90;&#x662F;&#x5F02;&#x6B65;&#x52A0;&#x8F7D;&#x7684;</li>
<li>&#x4E0D;&#x80FD;&#x975E;&#x963B;&#x585E;&#x7684;&#x5E76;&#x884C;&#x52A0;&#x8F7D;&#x591A;&#x4E2A;&#x6A21;&#x5757;</li>
</ul>
<p>&#x5B9E;&#x73B0;&#xFF1A;</p>
<ul>
<li>&#x670D;&#x52A1;&#x5668;&#x7AEF;&#x7684; <a href="http://www.nodejs.org" target="_blank">Node.js</a></li>
<li><a href="http://browserify.org" target="_blank">Browserify</a>&#xFF0C;&#x6D4F;&#x89C8;&#x5668;&#x7AEF;&#x7684; CommonJS &#x5B9E;&#x73B0;&#xFF0C;&#x53EF;&#x4EE5;&#x4F7F;&#x7528; NPM &#x7684;&#x6A21;&#x5757;&#xFF0C;&#x4F46;&#x662F;&#x7F16;&#x8BD1;&#x6253;&#x5305;&#x540E;&#x7684;&#x6587;&#x4EF6;&#x4F53;&#x79EF;&#x53EF;&#x80FD;&#x5F88;&#x5927;</li>
<li><a href="https://github.com/medikoo/modules-webmake" target="_blank">modules-webmake</a>&#xFF0C;&#x7C7B;&#x4F3C;Browserify&#xFF0C;&#x8FD8;&#x4E0D;&#x5982; Browserify &#x7075;&#x6D3B;</li>
<li><a href="https://github.com/substack/wreq" target="_blank">wreq</a>&#xFF0C;Browserify &#x7684;&#x524D;&#x8EAB;</li>
</ul>
<h4 id="amd">AMD</h4>
<p><a href="https://github.com/amdjs/amdjs-api" target="_blank">Asynchronous Module Definition</a> &#x89C4;&#x8303;&#x5176;&#x5B9E;&#x53EA;&#x6709;&#x4E00;&#x4E2A;&#x4E3B;&#x8981;&#x63A5;&#x53E3; <code>define(id?, dependencies?, factory)</code>&#xFF0C;&#x5B83;&#x8981;&#x5728;&#x58F0;&#x660E;&#x6A21;&#x5757;&#x7684;&#x65F6;&#x5019;&#x6307;&#x5B9A;&#x6240;&#x6709;&#x7684;&#x4F9D;&#x8D56; <code>dependencies</code>&#xFF0C;&#x5E76;&#x4E14;&#x8FD8;&#x8981;&#x5F53;&#x505A;&#x5F62;&#x53C2;&#x4F20;&#x5230; <code>factory</code> &#x4E2D;&#xFF0C;&#x5BF9;&#x4E8E;&#x4F9D;&#x8D56;&#x7684;&#x6A21;&#x5757;&#x63D0;&#x524D;&#x6267;&#x884C;&#xFF0C;&#x4F9D;&#x8D56;&#x524D;&#x7F6E;&#x3002;</p>
<pre><code class="lang-js">define(<span class="hljs-string">&quot;module&quot;</span>, [<span class="hljs-string">&quot;dep1&quot;</span>, <span class="hljs-string">&quot;dep2&quot;</span>], <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">d1, d2</span>) </span>{
  <span class="hljs-keyword">return</span> someExportedValue;
});
<span class="hljs-built_in">require</span>([<span class="hljs-string">&quot;module&quot;</span>, <span class="hljs-string">&quot;../file&quot;</span>], <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">module, file</span>) </span>{ <span class="hljs-comment">/* ... */</span> });
</code></pre>
<p>&#x4F18;&#x70B9;&#xFF1A;</p>
<ul>
<li>&#x9002;&#x5408;&#x5728;&#x6D4F;&#x89C8;&#x5668;&#x73AF;&#x5883;&#x4E2D;&#x5F02;&#x6B65;&#x52A0;&#x8F7D;&#x6A21;&#x5757;</li>
<li>&#x53EF;&#x4EE5;&#x5E76;&#x884C;&#x52A0;&#x8F7D;&#x591A;&#x4E2A;&#x6A21;&#x5757;</li>
</ul>
<p>&#x7F3A;&#x70B9;&#xFF1A;</p>
<ul>
<li>&#x63D0;&#x9AD8;&#x4E86;&#x5F00;&#x53D1;&#x6210;&#x672C;&#xFF0C;&#x4EE3;&#x7801;&#x7684;&#x9605;&#x8BFB;&#x548C;&#x4E66;&#x5199;&#x6BD4;&#x8F83;&#x56F0;&#x96BE;&#xFF0C;&#x6A21;&#x5757;&#x5B9A;&#x4E49;&#x65B9;&#x5F0F;&#x7684;&#x8BED;&#x4E49;&#x4E0D;&#x987A;&#x7545;</li>
<li>&#x4E0D;&#x7B26;&#x5408;&#x901A;&#x7528;&#x7684;&#x6A21;&#x5757;&#x5316;&#x601D;&#x7EF4;&#x65B9;&#x5F0F;&#xFF0C;&#x662F;&#x4E00;&#x79CD;&#x59A5;&#x534F;&#x7684;&#x5B9E;&#x73B0;</li>
</ul>
<p>&#x5B9E;&#x73B0;&#xFF1A;</p>
<ul>
<li><a href="http://requirejs.org" target="_blank">RequireJS</a></li>
<li><a href="https://github.com/cujojs/curl" target="_blank">curl</a></li>
</ul>
<h4 id="cmd">CMD</h4>
<p><a href="https://github.com/cmdjs/specification/blob/master/draft/module.md" target="_blank">Common Module Definition</a> &#x89C4;&#x8303;&#x548C; AMD &#x5F88;&#x76F8;&#x4F3C;&#xFF0C;&#x5C3D;&#x91CF;&#x4FDD;&#x6301;&#x7B80;&#x5355;&#xFF0C;&#x5E76;&#x4E0E; CommonJS &#x548C; Node.js &#x7684; Modules &#x89C4;&#x8303;&#x4FDD;&#x6301;&#x4E86;&#x5F88;&#x5927;&#x7684;&#x517C;&#x5BB9;&#x6027;&#x3002;</p>
<pre><code class="lang-js">define(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">require, exports, module</span>) </span>{
  <span class="hljs-keyword">var</span> $ = <span class="hljs-built_in">require</span>(<span class="hljs-string">&apos;jquery&apos;</span>);
  <span class="hljs-keyword">var</span> Spinning = <span class="hljs-built_in">require</span>(<span class="hljs-string">&apos;./spinning&apos;</span>);
  exports.doSomething = ...
  <span class="hljs-built_in">module</span>.exports = ...
})
</code></pre>
<p>&#x4F18;&#x70B9;&#xFF1A;</p>
<ul>
<li>&#x4F9D;&#x8D56;&#x5C31;&#x8FD1;&#xFF0C;&#x5EF6;&#x8FDF;&#x6267;&#x884C;</li>
<li>&#x53EF;&#x4EE5;&#x5F88;&#x5BB9;&#x6613;&#x5728; Node.js &#x4E2D;&#x8FD0;&#x884C;</li>
</ul>
<p>&#x7F3A;&#x70B9;&#xFF1A;</p>
<ul>
<li>&#x4F9D;&#x8D56; SPM &#x6253;&#x5305;&#xFF0C;&#x6A21;&#x5757;&#x7684;&#x52A0;&#x8F7D;&#x903B;&#x8F91;&#x504F;&#x91CD;</li>
</ul>
<p>&#x5B9E;&#x73B0;&#xFF1A;</p>
<ul>
<li><a href="http://seajs.org/" target="_blank">Sea.js</a></li>
<li><a href="https://github.com/cloudcome/coolie" target="_blank">coolie</a></li>
</ul>
<h4 id="umd">UMD</h4>
<p><a href="https://github.com/umdjs/umd" target="_blank">Universal Module Definition</a> &#x89C4;&#x8303;&#x7C7B;&#x4F3C;&#x4E8E;&#x517C;&#x5BB9; CommonJS &#x548C; AMD &#x7684;&#x8BED;&#x6CD5;&#x7CD6;&#xFF0C;&#x662F;&#x6A21;&#x5757;&#x5B9A;&#x4E49;&#x7684;&#x8DE8;&#x5E73;&#x53F0;&#x89E3;&#x51B3;&#x65B9;&#x6848;&#x3002;</p>
<h4 id="es6-&#x6A21;&#x5757;">ES6 &#x6A21;&#x5757;</h4>
<p>EcmaScript6 &#x6807;&#x51C6;&#x589E;&#x52A0;&#x4E86; JavaScript &#x8BED;&#x8A00;&#x5C42;&#x9762;&#x7684;&#x6A21;&#x5757;&#x4F53;&#x7CFB;&#x5B9A;&#x4E49;&#x3002;<a href="http://es6.ruanyifeng.com/#docs/module" target="_blank">ES6 &#x6A21;&#x5757;</a>&#x7684;&#x8BBE;&#x8BA1;&#x601D;&#x60F3;&#xFF0C;&#x662F;&#x5C3D;&#x91CF;&#x7684;&#x9759;&#x6001;&#x5316;&#xFF0C;&#x4F7F;&#x5F97;&#x7F16;&#x8BD1;&#x65F6;&#x5C31;&#x80FD;&#x786E;&#x5B9A;&#x6A21;&#x5757;&#x7684;&#x4F9D;&#x8D56;&#x5173;&#x7CFB;&#xFF0C;&#x4EE5;&#x53CA;&#x8F93;&#x5165;&#x548C;&#x8F93;&#x51FA;&#x7684;&#x53D8;&#x91CF;&#x3002;CommonJS &#x548C; AMD &#x6A21;&#x5757;&#xFF0C;&#x90FD;&#x53EA;&#x80FD;&#x5728;&#x8FD0;&#x884C;&#x65F6;&#x786E;&#x5B9A;&#x8FD9;&#x4E9B;&#x4E1C;&#x897F;&#x3002;</p>
<pre><code class="lang-js"><span class="hljs-keyword">import</span> <span class="hljs-string">&quot;jquery&quot;</span>;
<span class="hljs-keyword">export</span> <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">doStuff</span>(<span class="hljs-params"></span>) </span>{}
<span class="hljs-built_in">module</span> <span class="hljs-string">&quot;localModule&quot;</span> {}
</code></pre>
<p>&#x4F18;&#x70B9;&#xFF1A;</p>
<ul>
<li>&#x5BB9;&#x6613;&#x8FDB;&#x884C;&#x9759;&#x6001;&#x5206;&#x6790;</li>
<li>&#x9762;&#x5411;&#x672A;&#x6765;&#x7684; EcmaScript &#x6807;&#x51C6;</li>
</ul>
<p>&#x7F3A;&#x70B9;&#xFF1A;</p>
<ul>
<li>&#x539F;&#x751F;&#x6D4F;&#x89C8;&#x5668;&#x7AEF;&#x8FD8;&#x6CA1;&#x6709;&#x5B9E;&#x73B0;&#x8BE5;&#x6807;&#x51C6;</li>
<li>&#x5168;&#x65B0;&#x7684;&#x547D;&#x4EE4;&#x5B57;&#xFF0C;&#x65B0;&#x7248;&#x7684; Node.js&#x624D;&#x652F;&#x6301;</li>
</ul>
<p>&#x5B9E;&#x73B0;&#xFF1A;</p>
<ul>
<li><a href="https://babeljs.io/" target="_blank">Babel</a></li>
</ul>

<h4 id="&#x524D;&#x7AEF;&#x6A21;&#x5757;&#x52A0;&#x8F7D;">&#x524D;&#x7AEF;&#x6A21;&#x5757;&#x52A0;&#x8F7D;</h4>
<p>&#x524D;&#x7AEF;&#x6A21;&#x5757;&#x8981;&#x5728;&#x5BA2;&#x6237;&#x7AEF;&#x4E2D;&#x6267;&#x884C;&#xFF0C;&#x6240;&#x4EE5;&#x4ED6;&#x4EEC;&#x9700;&#x8981;&#x589E;&#x91CF;&#x52A0;&#x8F7D;&#x5230;&#x6D4F;&#x89C8;&#x5668;&#x4E2D;&#x3002;</p>
<p>&#x6A21;&#x5757;&#x7684;&#x52A0;&#x8F7D;&#x548C;&#x4F20;&#x8F93;&#xFF0C;&#x6211;&#x4EEC;&#x9996;&#x5148;&#x80FD;&#x60F3;&#x5230;&#x4E24;&#x79CD;&#x6781;&#x7AEF;&#x7684;&#x65B9;&#x5F0F;&#xFF0C;&#x4E00;&#x79CD;&#x662F;&#x6BCF;&#x4E2A;&#x6A21;&#x5757;&#x6587;&#x4EF6;&#x90FD;&#x5355;&#x72EC;&#x8BF7;&#x6C42;&#xFF0C;&#x53E6;&#x4E00;&#x79CD;&#x662F;&#x628A;&#x6240;&#x6709;&#x6A21;&#x5757;&#x6253;&#x5305;&#x6210;&#x4E00;&#x4E2A;&#x6587;&#x4EF6;&#x7136;&#x540E;&#x53EA;&#x8BF7;&#x6C42;&#x4E00;&#x6B21;&#x3002;&#x663E;&#x800C;&#x6613;&#x89C1;&#xFF0C;&#x6BCF;&#x4E2A;&#x6A21;&#x5757;&#x90FD;&#x53D1;&#x8D77;&#x5355;&#x72EC;&#x7684;&#x8BF7;&#x6C42;&#x9020;&#x6210;&#x4E86;&#x8BF7;&#x6C42;&#x6B21;&#x6570;&#x8FC7;&#x591A;&#xFF0C;&#x5BFC;&#x81F4;&#x5E94;&#x7528;&#x542F;&#x52A8;&#x901F;&#x5EA6;&#x6162;&#xFF1B;&#x4E00;&#x6B21;&#x8BF7;&#x6C42;&#x52A0;&#x8F7D;&#x6240;&#x6709;&#x6A21;&#x5757;&#x5BFC;&#x81F4;&#x6D41;&#x91CF;&#x6D6A;&#x8D39;&#x3001;&#x521D;&#x59CB;&#x5316;&#x8FC7;&#x7A0B;&#x6162;&#x3002;&#x8FD9;&#x4E24;&#x79CD;&#x65B9;&#x5F0F;&#x90FD;&#x4E0D;&#x662F;&#x597D;&#x7684;&#x89E3;&#x51B3;&#x65B9;&#x6848;&#xFF0C;&#x5B83;&#x4EEC;&#x8FC7;&#x4E8E;&#x7B80;&#x5355;&#x7C97;&#x66B4;&#x3002;</p>
<p><strong>&#x5206;&#x5757;&#x4F20;&#x8F93;</strong>&#xFF0C;&#x6309;&#x9700;&#x8FDB;&#x884C;&#x61D2;&#x52A0;&#x8F7D;&#xFF0C;&#x5728;&#x5B9E;&#x9645;&#x7528;&#x5230;&#x67D0;&#x4E9B;&#x6A21;&#x5757;&#x7684;&#x65F6;&#x5019;&#x518D;&#x589E;&#x91CF;&#x66F4;&#x65B0;&#xFF0C;&#x624D;&#x662F;&#x8F83;&#x4E3A;&#x5408;&#x7406;&#x7684;&#x6A21;&#x5757;&#x52A0;&#x8F7D;&#x65B9;&#x6848;&#x3002;</p>
<p>&#x8981;&#x5B9E;&#x73B0;&#x6A21;&#x5757;&#x7684;&#x6309;&#x9700;&#x52A0;&#x8F7D;&#xFF0C;&#x5C31;&#x9700;&#x8981;&#x4E00;&#x4E2A;&#x5BF9;&#x6574;&#x4E2A;&#x4EE3;&#x7801;&#x5E93;&#x4E2D;&#x7684;&#x6A21;&#x5757;&#x8FDB;&#x884C;&#x9759;&#x6001;&#x5206;&#x6790;&#x3001;&#x7F16;&#x8BD1;&#x6253;&#x5305;&#x7684;&#x8FC7;&#x7A0B;&#x3002;</p>
<h4 id="&#x6240;&#x6709;&#x8D44;&#x6E90;&#x90FD;&#x662F;&#x6A21;&#x5757;">&#x6240;&#x6709;&#x8D44;&#x6E90;&#x90FD;&#x662F;&#x6A21;&#x5757;</h4>
<p>&#x5728;&#x4E0A;&#x9762;&#x7684;&#x5206;&#x6790;&#x8FC7;&#x7A0B;&#x4E2D;&#xFF0C;&#x6211;&#x4EEC;&#x63D0;&#x5230;&#x7684;&#x6A21;&#x5757;&#x4EC5;&#x4EC5;&#x662F;&#x6307;JavaScript&#x6A21;&#x5757;&#x6587;&#x4EF6;&#x3002;&#x7136;&#x800C;&#xFF0C;&#x5728;&#x524D;&#x7AEF;&#x5F00;&#x53D1;&#x8FC7;&#x7A0B;&#x4E2D;&#x8FD8;&#x6D89;&#x53CA;&#x5230;&#x6837;&#x5F0F;&#x3001;&#x56FE;&#x7247;&#x3001;&#x5B57;&#x4F53;&#x3001;HTML &#x6A21;&#x677F;&#x7B49;&#x7B49;&#x4F17;&#x591A;&#x7684;&#x8D44;&#x6E90;&#x3002;&#x8FD9;&#x4E9B;&#x8D44;&#x6E90;&#x8FD8;&#x4F1A;&#x4EE5;&#x5404;&#x79CD;&#x65B9;&#x8A00;&#x7684;&#x5F62;&#x5F0F;&#x5B58;&#x5728;&#xFF0C;&#x6BD4;&#x5982; coffeescript&#x3001; less&#x3001; sass&#x3001;&#x4F17;&#x591A;&#x7684;&#x6A21;&#x677F;&#x5E93;&#x3001;&#x591A;&#x8BED;&#x8A00;&#x7CFB;&#x7EDF;&#xFF08;i18n&#xFF09;&#x7B49;&#x7B49;&#x3002;</p>
<p>&#x5982;&#x679C;&#x4ED6;&#x4EEC;&#x90FD;&#x53EF;&#x4EE5;&#x89C6;&#x4F5C;&#x6A21;&#x5757;&#xFF0C;&#x5E76;&#x4E14;&#x90FD;&#x53EF;&#x4EE5;&#x901A;&#x8FC7;<code>require</code>&#x7684;&#x65B9;&#x5F0F;&#x6765;&#x52A0;&#x8F7D;&#xFF0C;&#x5C06;&#x5E26;&#x6765;&#x4F18;&#x96C5;&#x7684;&#x5F00;&#x53D1;&#x4F53;&#x9A8C;&#xFF0C;&#x6BD4;&#x5982;&#xFF1A;</p>
<pre><code class="lang-js"><span class="hljs-built_in">require</span>(<span class="hljs-string">&quot;./style.css&quot;</span>);
<span class="hljs-built_in">require</span>(<span class="hljs-string">&quot;./style.less&quot;</span>);
<span class="hljs-built_in">require</span>(<span class="hljs-string">&quot;./template.jade&quot;</span>);
<span class="hljs-built_in">require</span>(<span class="hljs-string">&quot;./image.png&quot;</span>);
</code></pre>
<p>&#x90A3;&#x4E48;&#x5982;&#x4F55;&#x505A;&#x5230;&#x8BA9; <code>require</code> &#x80FD;&#x52A0;&#x8F7D;&#x5404;&#x79CD;&#x8D44;&#x6E90;&#x5462;&#xFF1F;</p>
<h4 id="&#x9759;&#x6001;&#x5206;&#x6790;">&#x9759;&#x6001;&#x5206;&#x6790;</h4>
<p>&#x5728;&#x7F16;&#x8BD1;&#x7684;&#x65F6;&#x5019;&#xFF0C;&#x8981;&#x5BF9;&#x6574;&#x4E2A;&#x4EE3;&#x7801;&#x8FDB;&#x884C;&#x9759;&#x6001;&#x5206;&#x6790;&#xFF0C;&#x5206;&#x6790;&#x51FA;&#x5404;&#x4E2A;&#x6A21;&#x5757;&#x7684;&#x7C7B;&#x578B;&#x548C;&#x5B83;&#x4EEC;&#x4F9D;&#x8D56;&#x5173;&#x7CFB;&#xFF0C;&#x7136;&#x540E;&#x5C06;&#x4E0D;&#x540C;&#x7C7B;&#x578B;&#x7684;&#x6A21;&#x5757;&#x63D0;&#x4EA4;&#x7ED9;&#x9002;&#x914D;&#x7684;&#x52A0;&#x8F7D;&#x5668;&#x6765;&#x5904;&#x7406;&#x3002;&#x6BD4;&#x5982;&#x4E00;&#x4E2A;&#x7528; LESS &#x5199;&#x7684;&#x6837;&#x5F0F;&#x6A21;&#x5757;&#xFF0C;&#x53EF;&#x4EE5;&#x5148;&#x7528; LESS &#x52A0;&#x8F7D;&#x5668;&#x5C06;&#x5B83;&#x8F6C;&#x6210;&#x4E00;&#x4E2A;CSS &#x6A21;&#x5757;&#xFF0C;&#x5728;&#x901A;&#x8FC7; CSS &#x6A21;&#x5757;&#x628A;&#x4ED6;&#x63D2;&#x5165;&#x5230;&#x9875;&#x9762;&#x7684; <code>&lt;style&gt;</code> &#x6807;&#x7B7E;&#x4E2D;&#x6267;&#x884C;&#x3002;Webpack &#x5C31;&#x662F;&#x5728;&#x8FD9;&#x6837;&#x7684;&#x9700;&#x6C42;&#x4E2D;&#x5E94;&#x8FD0;&#x800C;&#x751F;&#x3002;</p>
<p>&#x540C;&#x65F6;&#xFF0C;&#x4E3A;&#x4E86;&#x80FD;&#x5229;&#x7528;&#x5DF2;&#x7ECF;&#x5B58;&#x5728;&#x7684;&#x5404;&#x79CD;&#x6846;&#x67B6;&#x3001;&#x5E93;&#x548C;&#x5DF2;&#x7ECF;&#x5199;&#x597D;&#x7684;&#x6587;&#x4EF6;&#xFF0C;&#x6211;&#x4EEC;&#x8FD8;&#x9700;&#x8981;&#x4E00;&#x4E2A;&#x6A21;&#x5757;&#x52A0;&#x8F7D;&#x7684;&#x517C;&#x5BB9;&#x7B56;&#x7565;&#xFF0C;&#x6765;&#x907F;&#x514D;&#x91CD;&#x5199;&#x6240;&#x6709;&#x7684;&#x6A21;&#x5757;&#x3002;</p>
<h2>参考文档汇总</h2>



<li><a href="http://wiki.commonjs.org/wiki/CommonJS" target="_blank">CommonJS &#x89C4;&#x8303;</a></li>
<li><a href="http://guowenfh.github.io/2016/03/24/vue-webpack-01-base/" target="_blank">Webpack入坑之旅</a></li>
<li><a href="http://webpackdoc.com/index.html" target="_blank">Webpack中文指南</a></li>
<li><a href="https://github.com/amdjs/amdjs-api" target="_blank">Asynchronous Module Definition</a></li>
<li><a href="https://github.com/cmdjs/specification/blob/master/draft/module.md" target="_blank">Common Module Definition</a></li>
<li><a href="https://github.com/seajs/seajs/issues/242" target="_blank">CMD &#x6A21;&#x5757;&#x5B9A;&#x4E49;&#x89C4;&#x8303;</a></li>
<li><a href="https://github.com/umdjs/umd" target="_blank">Universal Module Definition</a></li>
<li><a href="http://es6.ruanyifeng.com/#docs/module" target="_blank">ECMAScript 6 Module</a></li>
<li><a href="http://www.75team.com/archives/882" target="_blank">&#x4EC0;&#x4E48;&#x662F; AMD&#x3001; CommonJS&#x3001; UMD</a></li>
<li><a href="http://my.oschina.net/felumanman/blog/263330" target="_blank">&#x5173;&#x4E8E; CommonJS AMD CMD UMD</a></li>
<li><a href="http://blog.3gcnbeta.com/2014/05/27/%E4%B8%BA%E4%BB%80%E4%B9%88%E6%88%91%E6%8E%A8%E8%8D%90requirejs-%E8%80%8C%E4%B8%8D%E6%98%AFseajs/" target="_blank">&#x4E3A;&#x4EC0;&#x4E48;&#x6211;&#x63A8;&#x8350; requirejs &#x800C;&#x4E0D;&#x662F; seajs</a></li>
<li><a href="http://www.zhihu.com/question/20351507" target="_blank">AMD &#x548C; CMD &#x7684;&#x533A;&#x522B;&#x6709;&#x54EA;&#x4E9B;</a></li>
<li><a href="https://github.com/seajs/seajs/issues/547" target="_blank">&#x524D;&#x7AEF;&#x6A21;&#x5757;&#x5316;&#x5F00;&#x53D1;&#x7684;&#x4EF7;&#x503C;</a></li>
<li><a href="http://www.blueskyonmars.com/2009/01/29/what-server-side-javascript-needs/" target="_blank">What Server Side JavaScript needs</a></li>
<li><a href="http://requirejs.org" target="_blank">RequireJS</a></li>
<li><a href="https://github.com/cujojs/curl" target="_blank">curl</a></li>
<li><a href="http://seajs.org/" target="_blank">Sea.js</a></li>
<li><a href="https://github.com/cloudcome/coolie" target="_blank">coolie</a></li>
<li><a href="http://browserify.org" target="_blank">Browserify</a></li>
<li><a href="https://github.com/medikoo/modules-webmake" target="_blank">modules-webmake</a></li>
<li><a href="https://github.com/substack/wreq" target="_blank">wreq</a></li>
<li><a href="http://webpack.github.io/docs/" target="_blank">Webpack &#x5B98;&#x65B9;&#x6587;&#x6863;</a></li>
<li><a href="https://fakefish.github.io/react-webpack-cookbook/" target="_blank">React Webpack cookbook</a></li>
<li><a href="https://babeljs.io/" target="_blank">Babel</a></li>
<li><a href="http://webpack.github.io/docs/multiple-entry-points.html">Multiple entry points</a></li>
<li><a href="http://webpack.github.io/docs/configuration.html#entry">configuration#entry</a></li>
<li><a href="http://webpack.github.io/docs/code-splitting.html#multiple-entry-chunks">Code Splitting</a></li>
