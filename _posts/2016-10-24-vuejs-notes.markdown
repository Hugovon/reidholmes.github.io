---
published: true
layout:     post
title:      "vuejs框架笔记"
date:       2016-10-24 18:38:22
categories:
---


<h3>安装 </h3>
<p>使用npm安装： </p>
<pre>npm install vue</pre> <p>当然你也可以在github上clone最新的版本并作为单文件引入，或者使用CDN: </p>
<pre class="brush:java; toolbar: true; auto-links: false;">http://cdn.jsdelivr.net/vue/1.0.7/vue.min.js
http://cdnjs.cloudflare.com/ajax/libs/vue/1.0.7/vue.min.js</pre> <h3>HelloWorld </h3>
<p>动手写第一个Vue.js 应用吧。app.html: </p>
<pre class="brush:java; toolbar: true; auto-links: false;">&lt;div id="app"&gt;
    &lt;div&gt;{{message}}&lt;/div&gt;
    &lt;input type="text" v-model="message"&gt;
&lt;/div&gt;</pre> <p>app.js: </p>
<pre class="brush:java; toolbar: true; auto-links: false;">new Vue({
    el:'#app',
    data: {
        message:'hello vue.js.'
    }
});</pre> <h3>创建Vue实例 </h3>
<p>在使用Vue.js之前，我们需要先像这样实例化一个Vue对象： </p>
<pre>new Vue({
   el:'#app'
});</pre> <h3>双向数据绑定 </h3>
<p><img alt="Vue.js notes" src="https://reidholmes.github.io/img/vuejs.png" width="700" height="372" /> </p>
<p>就像HelloWorld展示的那样，app.html是view层，app.js是model层，通过vue.js（使用v-model这个指令）完成中间的底层逻辑，实现绑定的效果。改变其中的任何一层，另外一层都会改变。 </p>
<h2>目录结构</h2><p>src 为开发目录，其中 components 为组件子目录，templates 为模板子目录。</p><p>dist 为构建出的文件目录。</p><p>index.html 为入口文件。</p><p>package.json 为项目描述文件，是刚才 npm init 所建立。 </p><p>webpack.config.js 是 webpack 的构建配置文件</p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/102.png" alt="" title=""></p><h2>Webpack 配置</h2><p>下面是 webpack 的配置文件，如何使用 webpack，请移步 webpack 的官网。</p><pre class="prettyprint"><code class=" hljs coffeescript"><span class="hljs-reserved">var</span> webpack= <span class="hljs-built_in">require</span>(<span class="hljs-string">"webpack"</span>);

<span class="hljs-built_in">module</span>.<span class="hljs-built_in">exports</span>={
    <span class="hljs-attribute">entry</span>:{
        <span class="hljs-attribute">bundle</span>:[ <span class="hljs-string">"./src/app.js"</span>]
    },
    <span class="hljs-attribute">output</span>:{
        <span class="hljs-attribute">path</span>:__dirname,
        <span class="hljs-attribute">publicPath</span>:<span class="hljs-string">"/"</span>,
        <span class="hljs-attribute">filename</span>:<span class="hljs-string">"dist/[name].js"</span>
    },
    <span class="hljs-attribute">module</span>:{
        <span class="hljs-attribute">loaders</span>:[
            {<span class="hljs-attribute">test</span>: <span class="hljs-regexp">/\.html$/</span>, <span class="hljs-attribute">loaders</span>: [<span class="hljs-string">&#039;html&#039;</span>]},
            {<span class="hljs-attribute">test</span>: <span class="hljs-regexp">/(\.js)$/</span>, <span class="hljs-attribute">loader</span>:[<span class="hljs-string">"babel"</span>] ,<span class="hljs-attribute">exclude</span>:<span class="hljs-regexp">/node_modules/</span>, 
             <span class="hljs-attribute">query</span>:{
                    <span class="hljs-attribute">presets</span>:[<span class="hljs-string">"es2015"</span>]
             }
            }
        ]
    },
    <span class="hljs-attribute">resolve</span>:{
    },
    <span class="hljs-attribute">plugins</span>:[
         /* 
         <span class="hljs-keyword">new</span> webpack.optimize.UglifyJsPlugin({
            <span class="hljs-attribute">compress</span>: {
                <span class="hljs-attribute">warnings</span>: <span class="hljs-literal">false</span>
            }
        })
               */
    ]
}</code></pre><h2>入口文件</h2><p><strong>index.html</strong></p><pre class="prettyprint"><code class=" hljs xml">
    <span class="hljs-tag">&lt;<span class="hljs-title">meta</span> <span class="hljs-attribute">charset</span>=<span class="hljs-value">"UTF-8"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-title">title</span>&gt;</span>Vue Router Demo<span class="hljs-tag">&lt;/<span class="hljs-title">title</span>&gt;</span>

   <span class="hljs-tag">&lt;<span class="hljs-title">div</span> <span class="hljs-attribute">id</span>=<span class="hljs-value">"app"</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-title">router-view</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-title">router-view</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-title">div</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-title">script</span> <span class="hljs-attribute">src</span>=<span class="hljs-value">"dist/bundle.js"</span>&gt;</span><span class="javascript"></span><span class="hljs-tag">&lt;/<span class="hljs-title">script</span>&gt;</span>
</code></pre><p>其中 id 为 app 的 div 是页面容器，其中的 router-view 会由 vue-router 去渲染组件，讲结果挂载到这个 div 上。 </p><p><strong>app.js</strong></p><pre class="prettyprint"><code class=" hljs php"><span class="hljs-keyword">var</span> Vue = <span class="hljs-keyword">require</span>(<span class="hljs-string">&#039;vue&#039;</span>);
<span class="hljs-keyword">var</span> VueRouter = <span class="hljs-keyword">require</span>(<span class="hljs-string">&#039;vue-router&#039;</span>);

Vue.<span class="hljs-keyword">use</span>(VueRouter);
Vue.config.debug = <span class="hljs-keyword">true</span>;
Vue.config.delimiters = [<span class="hljs-string">&#039;${&#039;</span>, <span class="hljs-string">&#039;}&#039;</span>]; <span class="hljs-comment">// 把默认的{{ }} 改成ES6的模板字符串 ${ }</span>
Vue.config.devtools = <span class="hljs-keyword">true</span>;

<span class="hljs-keyword">var</span> App = Vue.extend({});
<span class="hljs-keyword">var</span> router = <span class="hljs-keyword">new</span> VueRouter({});

router.map(<span class="hljs-keyword">require</span>(<span class="hljs-string">&#039;./routes&#039;</span>));
router.start(App, <span class="hljs-string">&#039;#app&#039;</span>);
router.go({<span class="hljs-string">"path"</span>:<span class="hljs-string">"/"</span>});</code></pre><p>这是 vue 路由的配置。 其中由于习惯问题，我把 vue 默认的{{ }} 改成了的 ${ }  ,总感觉这样看模板，才顺眼一些。 </p><p><strong>routes.js</strong></p><pre class="prettyprint"><code class=" hljs coffeescript"><span class="hljs-built_in">module</span>.<span class="hljs-built_in">exports</span> = {
  <span class="hljs-string">&#039;/&#039;</span>: {
    <span class="hljs-attribute">component</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/index&#039;</span>)
  },
   <span class="hljs-string">&#039;/list&#039;</span>: {
    <span class="hljs-attribute">component</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/list&#039;</span>)
  },
  <span class="hljs-string">&#039;*&#039;</span>: {
    <span class="hljs-attribute">component</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/notFound&#039;</span>)
  }
}</code></pre><h2>第一个组件</h2><p><strong>components/index.js</strong></p><pre class="prettyprint"><code class=" hljs lua"><span class="hljs-built_in">module</span>.exports = {
  template: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;../templates/index.html&#039;</span>),

  ready: <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">()</span></span> {
  }
};</code></pre><p><strong>templates/index.html</strong></p><pre class="prettyprint"><code class=" hljs markdown">
<span class="hljs-header"># Index</span>

<span class="hljs-bullet">* </span><span class="hljs-bullet">* *</span>

Hello World Index!
</code></pre><p><strong>执行 webpack 构建命令</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/103.png" alt="" title=""></p><p><strong>浏览器中访问：</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/104.png" alt="" title=""></p><p><strong>查看 bundle 源码：</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/105.png" alt="" title=""></p><p>发现 template 模板文件，已经被 webpack 打成字符串了。这其中，其实是 webpack 的 html-loader 起的作用</p><h2>组件之间跳转</h2><p>修改刚才的 index 组件，增加一个跳转链接，不用 href 了，要用 vue 的指令 v-link。</p><pre class="prettyprint"><code class=" hljs livecodeserver">
<span class="hljs-comment"># Index</span>

* * *

Hello World Index!

&lt;<span class="hljs-operator">a</span> v-link=<span class="hljs-string">"{path:&#039;/list&#039;}"</span>&gt;List Page&lt;/<span class="hljs-operator">a</span>&gt;
</code></pre><p>添加 list 组件</p><p><strong>components/list.js</strong></p><pre class="prettyprint"><code class=" hljs lua"><span class="hljs-built_in">module</span>.exports = {
  template: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;../templates/list.html&#039;</span>),

  data:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span></span>{
    <span class="hljs-keyword">return</span> {items:[{<span class="hljs-string">"id"</span>:<span class="hljs-number">1</span>,<span class="hljs-string">"name"</span>:<span class="hljs-string">"hello11"</span>},{<span class="hljs-string">"id"</span>:<span class="hljs-number">2</span>,<span class="hljs-string">"name"</span>:<span class="hljs-string">"hello22"</span>}]};
  },
  ready: <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">()</span></span> {
  }
};</code></pre><p><strong>templates/list.html</strong></p><pre class="prettyprint"><code class=" hljs applescript">
<span class="hljs-comment"># List</span>

* * *

Hello List Page!

*   ${<span class="hljs-property">item</span>.<span class="hljs-property">id</span>} : ${<span class="hljs-property">item</span>.<span class="hljs-property">name</span>}
</code></pre><p>v-for 也是 vue 的默认指令，是用来循环数据列表的。</p><p>现在开始执行 webpack –watch 命令进行监听，这样就不用每次敲 webpack 命令了。只要开发者每次修改 js 点了保存，webpack 都会自动构建最新的 bundle 文件。</p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/106.png" alt="" title=""></p><p>浏览器里试试看：</p><p><strong>index 页</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/107.png" alt="" title=""></p><p><strong>点击 List Page 跳转到 list 页</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/108.png" alt="" title=""></p><p>Bingo！ 单页面两个组件之间跳转切换成功！</p><h2> 组件生命周期</h2><p>修改 <strong>componets/list.js </strong></p><pre class="prettyprint"><code class=" hljs javascript">module.exports = {
  template: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;../templates/list.html&#039;</span>),

  data:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
    <span class="hljs-keyword">return</span> {items:[{<span class="hljs-string">"id"</span>:<span class="hljs-number">1</span>,<span class="hljs-string">"name"</span>:<span class="hljs-string">"hello11"</span>},{<span class="hljs-string">"id"</span>:<span class="hljs-number">2</span>,<span class="hljs-string">"name"</span>:<span class="hljs-string">"hello22"</span>}]};
  },

  <span class="hljs-comment">//在实例开始初始化时同步调用。此时数据观测、事件和 watcher 都尚未初始化</span>
  init:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
    console.log(<span class="hljs-string">"init.."</span>);
  },

  <span class="hljs-comment">//在实例创建之后同步调用。此时实例已经结束解析选项，这意味着已建立：数据绑定，计算属性，方法，watcher/事件回调。但是还没有开始 DOM 编译，$el 还不存在。</span>
  created:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
    console.log(<span class="hljs-string">"created.."</span>);
  },

  <span class="hljs-comment">//在编译开始前调用。</span>
  beforeCompile:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
     console.log(<span class="hljs-string">"beforeCompile.."</span>);
  },

  <span class="hljs-comment">//在编译结束后调用。此时所有的指令已生效，因而数据的变化将触发 DOM 更新。但是不担保 $el 已插入文档。</span>
  compiled:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
     console.log(<span class="hljs-string">"compiled.."</span>);
  },

   <span class="hljs-comment">//在编译结束和 $el 第一次插入文档之后调用，如在第一次 attached 钩子之后调用。注意必须是由 Vue 插入（如 vm.$appendTo() 等方法或指令更新）才触发 ready 钩子。</span>
  ready: <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">()</span> {</span>
    console.log(<span class="hljs-string">"ready.."</span>);

  },

  <span class="hljs-comment">//在 vm.$el 插入 DOM 时调用。必须是由指令或实例方法（如 $appendTo()）插入，直接操作 vm.$el 不会 触发这个钩子。</span>
  attached:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
     console.log(<span class="hljs-string">"attached.."</span>);
  },

  <span class="hljs-comment">//在 vm.$el 从 DOM 中删除时调用。必须是由指令或实例方法删除，直接操作 vm.$el 不会 触发这个钩子。</span>
  detached:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
     console.log(<span class="hljs-string">"detached.."</span>);
  },

  <span class="hljs-comment">//在开始销毁实例时调用。此时实例仍然有功能。</span>
  beforeDestroy:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
     console.log(<span class="hljs-string">"beforeDestroy.."</span>);
  },

  <span class="hljs-comment">//在实例被销毁之后调用。此时所有的绑定和实例的指令已经解绑，所有的子实例也已经被销毁。如果有离开过渡，destroyed 钩子在过渡完成之后调用。</span>
  destroyed:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
     console.log(<span class="hljs-string">"destroyed.."</span>);
  }

};</code></pre><p>在浏览器里执行了看看：</p><p><strong>首次进入 List 页面的执行顺序如下：</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/109.png" alt="" title=""></p><p><strong>此时点一下浏览器的后退，List Component 会被销毁，执行顺序如下：</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/110.png" alt="" title=""></p><p><strong>这是官方的生命周期的图：</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/111.png" alt="" title=""></p><h2> 父组件与子组件</h2><p>在很多情况下，组件是有父子关系的，比如 list 列表组件有个子组件 item</p><p><strong>components/item.js</strong></p><pre class="prettyprint"><code class=" hljs lua"><span class="hljs-built_in">module</span>.exports = {
  template: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;../templates/item.html&#039;</span>),

  props:[<span class="hljs-string">"id"</span>,<span class="hljs-string">"name"</span>],

  ready: <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">()</span></span> {

  }
};</code></pre><p><strong>templates/item.html</strong></p><p>我是subitem： <span class="MathJax_Preview"></span><script type="math/tex" id="MathJax-Element-1">{id} - </script>{name}</p><p>修改 list 组件，添加 item 的引用</p><p><strong>components/list.js</strong></p><pre class="prettyprint"><code class=" hljs javascript"><span class="hljs-comment">//引用item组件</span>
import item from <span class="hljs-string">"./item"</span>;

module.exports = {
  template: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;../templates/list.html&#039;</span>),

  data:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
    <span class="hljs-keyword">return</span> {items:[{<span class="hljs-string">"id"</span>:<span class="hljs-number">1</span>,<span class="hljs-string">"name"</span>:<span class="hljs-string">"hello11"</span>},{<span class="hljs-string">"id"</span>:<span class="hljs-number">2</span>,<span class="hljs-string">"name"</span>:<span class="hljs-string">"hello22"</span>}]};
  },

 <span class="hljs-comment">//定义item组件为子组件</span>
  components:{
     <span class="hljs-string">"item"</span>:item
  },

  ready: <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">()</span> {</span>
  }

};</code></pre><p><strong>templates/list.html</strong></p><pre class="prettyprint"><code class=" hljs xml">
# List

* * *

Hello List Page!

*   <span class="hljs-comment">&lt;!--使用item子组件，同时把id,name使用props传值给item子组件--&gt;</span><span class="hljs-tag">&lt;<span class="hljs-title">item</span> <span class="hljs-attribute">v-bind:id</span>=<span class="hljs-value">"item.id"</span> <span class="hljs-attribute">v-bind:name</span>=<span class="hljs-value">"item.name"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-title">item</span>&gt;</span>
</code></pre><p><strong>浏览器里试试看：</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/112.png" alt="" title=""></p><p>子组件成功被调用了</p><h2>组件跳转传参</h2><p>组件之间的跳转传参，也是一种非常常见的情况。下面为列表页，增加跳转到详情页的跳转，并传参 id 给详情页</p><p><strong>修改路由 routes.js</strong></p><pre class="prettyprint"><code class=" hljs coffeescript"><span class="hljs-built_in">module</span>.<span class="hljs-built_in">exports</span> = {

  <span class="hljs-string">&#039;/&#039;</span>: {
    <span class="hljs-attribute">component</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/index&#039;</span>)
  },
   <span class="hljs-string">&#039;/list&#039;</span>: {
    <span class="hljs-attribute">component</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/list&#039;</span>)
  },

  <span class="hljs-regexp">//</span>增加详情页的跳转路由，并在路径上加上id传参，具名为name：show
   <span class="hljs-string">&#039;/show/:id&#039;</span>: {
      <span class="hljs-attribute">name</span>:<span class="hljs-string">"show"</span>,
      <span class="hljs-attribute">component</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/show&#039;</span>)
    },
  <span class="hljs-string">&#039;*&#039;</span>: {
    <span class="hljs-attribute">component</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/notFound&#039;</span>)
  }
}</code></pre><p>添加组件 show</p><p><strong>components/show.js</strong></p><pre class="prettyprint"><code class=" hljs javascript">module.exports = {
  template: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;../templates/show.html&#039;</span>),

  data:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
    <span class="hljs-keyword">return</span> {};
  },

  created:<span class="hljs-function"><span class="hljs-keyword">function</span><span class="hljs-params">()</span>{</span>
     <span class="hljs-comment">//获取params的参数ID</span>
     <span class="hljs-keyword">var</span> id=<span class="hljs-keyword">this</span>.$route.params.id;

     <span class="hljs-comment">//根据获取的参数ID，返回不同的data对象（真实业务中，这里应该是Ajax获取数据）</span>
     <span class="hljs-keyword">if</span> (id==<span class="hljs-number">1</span>){
         <span class="hljs-keyword">this</span>.$data={<span class="hljs-string">"id"</span>:id,<span class="hljs-string">"name"</span>:<span class="hljs-string">"hello111"</span>,<span class="hljs-string">"age"</span>:<span class="hljs-number">24</span>};
     }<span class="hljs-keyword">else</span>{
        <span class="hljs-keyword">this</span>.$data={<span class="hljs-string">"id"</span>:id,<span class="hljs-string">"name"</span>:<span class="hljs-string">"hello222"</span>,<span class="hljs-string">"age"</span>:<span class="hljs-number">28</span>};
     }
  },

  ready: <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-params">()</span> {</span>
    console.log(<span class="hljs-keyword">this</span>.$data);
  }

};</code></pre><p><strong>templates/show.html</strong></p><pre class="prettyprint"><code class=" hljs applescript">
<span class="hljs-comment"># Show</span>

* * *

Hello show page!

<span class="hljs-property">id</span>:${<span class="hljs-property">id</span>}

<span class="hljs-property">name</span>:${<span class="hljs-property">name</span>}

age:${age}
</code></pre><p><strong>修改 templates/item.html</strong></p><p>我是subitem： <a> <span class="MathJax_Preview"></span><script type="math/tex" id="MathJax-Element-2">{id} : </script>{name}</a></p><p>这里 name:’show’ 表示具名路由路径，params 就是传参。</p><p><strong>继续浏览器里点到详情页试试：</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/113.png" alt="" title=""></p><p><strong>点击“hello11”，跳转到详情页：</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/114.png" alt="" title=""></p><p>传参逻辑成功。</p><h2>嵌套路由</h2><p>仅有路由跳转是远远不够的，很多情况下，我们还有同一个页面上，多标签页的切换，在 vue 中，用嵌套路由，也可以非常方便的实现。</p><p>添加两个小组件</p><p><strong>components/tab1.js</strong></p><pre class="prettyprint"><code class=" hljs d"><span class="hljs-keyword">module</span>.exports = {
  <span class="hljs-keyword">template</span>: <span class="hljs-string">"

Tab1 content

"</span>
};</code></pre><p><strong>components/tab2.js</strong></p><pre class="prettyprint"><code class=" hljs d"><span class="hljs-keyword">module</span>.exports = {
  <span class="hljs-keyword">template</span>: <span class="hljs-string">"

Tab2 content

"</span>
};</code></pre><p><strong>修改 components/index.js 组件，挂载这两个子组件</strong></p><pre class="prettyprint"><code class=" hljs coffeescript"><span class="hljs-reserved">import</span> tab1 from <span class="hljs-string">"./tab1"</span>;
<span class="hljs-reserved">import</span> tab2 from <span class="hljs-string">"./tab2"</span>;

<span class="hljs-built_in">module</span>.<span class="hljs-built_in">exports</span> = {
  <span class="hljs-attribute">template</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;../templates/index.html&#039;</span>),

  <span class="hljs-attribute">components</span>:{
     <span class="hljs-string">"tab1"</span>:tab1,
     <span class="hljs-string">"tab2"</span>:tab2
  },

  <span class="hljs-attribute">ready</span>: <span class="hljs-reserved">function</span> () {

  }
};</code></pre><p><strong>在路由里加上子路由</strong></p><pre class="prettyprint"><code class=" hljs coffeescript"><span class="hljs-built_in">module</span>.<span class="hljs-built_in">exports</span> = {

  <span class="hljs-string">&#039;/&#039;</span>: {
    <span class="hljs-attribute">component</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/index&#039;</span>),

    <span class="hljs-regexp">//</span>子路由
    <span class="hljs-attribute">subRoutes</span>:{
      <span class="hljs-string">"/tab1"</span>:{
          <span class="hljs-attribute">component</span>:<span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/tab1&#039;</span>)
      },
      <span class="hljs-string">"/tab2"</span>:{
          <span class="hljs-attribute">component</span>:<span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/tab2&#039;</span>)
      }
    }
  },

   <span class="hljs-string">&#039;/list&#039;</span>: {
    <span class="hljs-attribute">component</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/list&#039;</span>)
  },

   <span class="hljs-string">&#039;/show/:id&#039;</span>: {
      <span class="hljs-attribute">name</span>:<span class="hljs-string">"show"</span>,
      <span class="hljs-attribute">component</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/show&#039;</span>)
    },

  <span class="hljs-string">&#039;*&#039;</span>: {
    <span class="hljs-attribute">component</span>: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/notFound&#039;</span>)
  }

}</code></pre><p>好了，在浏览器里试一下：</p><p><strong>初始状态：</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/115.png" alt="" title=""></p><p><strong>点了 tab1，tab2：</strong></p><p><img src="http://oa5504rxk.bkt.clouddn.com/week11_vue/116.png" alt="" title=""></p><p>Tab 切换没问题，可是，初始状态显示是空的，能不能默认显示 Tab1 Content 呢？很简单，调整下路由就可以了：</p><pre>module.exports = {

    component: <span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/index&#039;</span>),

    <span class="hljs-comment">//子路由</span>
    subRoutes:{
     <span class="hljs-comment">//默认显示Tab1</span>
      <span class="hljs-string">"/"</span>:{
          component:<span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/tab1&#039;</span>)
      },
      <span class="hljs-string">"/tab1"</span>:{
          component:<span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/tab1&#039;</span>)
      },
      <span class="hljs-string">"/tab2"</span>:{
          component:<span class="hljs-built_in">require</span>(<span class="hljs-string">&#039;./components/tab2&#039;</span>)
      }
    }</pre>

----------



<h2>参考文档汇总</h2>




- [http://vuejs.org/](http://vuejs.org/ )


- [https://github.com/vuejs](https://github.com/vuejs )


- [http://vegibit.com/vue-js-tutorial/](http://vegibit.com/vue-js-tutorial/ )


- [http://www.zhihu.com/people/evanyou](http://www.zhihu.com/people/evanyou )


- [http://www.html-js.com/article/column/99](http://www.html-js.com/article/column/99 )


- [https://github.com/vuejs](https://github.com/vuejs )