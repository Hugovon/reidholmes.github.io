---
published: true
layout:     post
title:      "构建一个 WEB 服务器（一）"
date:       2016-10-15 18:20:35
categories:
---






<div class="entry-content">
				<p>本文译自：<a class="reference external" href="http://ruslanspivak.com/lsbaws-part1/">http://ruslanspivak.com/lsbaws-part1/</a></p>
<p>有一天出去散步的时候，一个女人来到一处工地上，她看到有三个男人在工作。 她问第一个人，“你在做什么？“，第一个男人对这个问题感到非常的厌烦， 他大声的说到”你没看到我正在砌砖吗？“女人对这个答案不大满意， 她又去问第二个男人他在做什么。 第二个男人回答道，”我正在建造一面砖墙。” 然后，他把注意力转向了第一个男人，他说道， “嘿，你只完成墙的尾部。你需要把最后那块砖拿掉。” 仍旧不满意这个答案，她又问第三个男人他在做什么。 这个男人边望着天边对他说， ”我正在建造世界上最大的教堂。“ 当他站在这里仰望着天空的时候，另外两个男人开始争吵应该怎么放砖。 这个男人转向前面的两个男人并说道， “嘿，伙计们，别担心那块砖了。那只是一块内墙， 它将会被涂平没人会看到那块砖。 只需移动到另一层。”</p>
<p>这则故事的寓意是当你了解了整个系统并且理解了不同的部分是如何组合在一起的时候（砖，墙，教堂）， 你就可以更快的识别和解决问题（不正确的砖）。</p>
<p>对于从零开始创建我们自己的 Web server 来说需要做些什么呢？</p>
<p><strong>我相信要成为一个优秀的开发者你必须更好的理解你日常使用的基础软件系统，包括编程语言，编译器，解释器，数据库和操作系统，web server 和 web 框架。同时，为了更深入的理解整个系统，必须从零开始，一块砖一面墙的重新构建它们</strong></p>
<p>子曾经曰过：</p>
<blockquote><p>“我听到，我忘记。“</p></blockquote>
<p><img src="https://mozillazg.com/static/images/lsbaws-part1/LSBAWS_confucius_hear.png" alt="LSBAWS_confucius_hear.png" /></p>
<blockquote><p>”我看到，我记住。“</p></blockquote>
<p><img src="https://mozillazg.com/static/images/lsbaws-part1/LSBAWS_confucius_see.png" alt="LSBAWS_confucius_see.png" /></p>
<blockquote><p>”我做到，我理解。“</p></blockquote>
<p><img src="https://mozillazg.com/static/images/lsbaws-part1/LSBAWS_confucius_do.png" alt="LSBAWS_confucius_do.png" /></p>
<p>此刻，我希望你能深信通过重新构建一个不同的软件系统来学习它们是如何工作的是一个好主意。</p>
<p>在这三篇系列中我将向你展示如何构建你自己的基础的 Web server。让我们开始吧。</p>
<p>先开始最重要的事情，什么是一个 Web server？</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part1/LSBAWS_HTTP_request_response.png" alt="LSBAWS_HTTP_request_response.png" /></p>
<p>一言以蔽之，它是一个物理服务器上的一个网络服务器（哎呀，一个服务器上的服务器）同时等待客户端发送一个请求。 当它接收到一个请求的时候，它将生成一个响应并把它发送回客户端。 客户端和服务器的通信方式是使用的 HTTP 协议。 客户端可以是你的浏览器或其他会说 HTTP 的任何软件。</p>
<p>一个非常简单的 Web server 实现长什么样呢？我放了一个在这里。这个例子使用 Python 写的，但是就算你不懂 Python（它是个非常容易学习的语言，你可以试一下！）你也能够从下面的代码和解释中理解基本的概念：</p>
<div class="highlight">
<pre><span class="kn">import</span> <span class="nn">socket</span>

<span class="n">HOST</span><span class="p">,</span> <span class="n">PORT</span> <span class="o">=</span> <span class="s1">''</span><span class="p">,</span> <span class="mi">8888</span>

<span class="n">listen_socket</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">socket</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">AF_INET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SOCK_STREAM</span><span class="p">)</span>
<span class="n">listen_socket</span><span class="o">.</span><span class="n">setsockopt</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">SOL_SOCKET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SO_REUSEADDR</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
<span class="n">listen_socket</span><span class="o">.</span><span class="n">bind</span><span class="p">((</span><span class="n">HOST</span><span class="p">,</span> <span class="n">PORT</span><span class="p">))</span>
<span class="n">listen_socket</span><span class="o">.</span><span class="n">listen</span><span class="p">(</span><span class="mi">1</span><span class="p">)</span>
<span class="k">print</span> <span class="s1">'Serving HTTP on port </span><span class="si">%s</span><span class="s1"> ...'</span> <span class="o">%</span> <span class="n">PORT</span>
<span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
    <span class="n">client_connection</span><span class="p">,</span> <span class="n">client_address</span> <span class="o">=</span> <span class="n">listen_socket</span><span class="o">.</span><span class="n">accept</span><span class="p">()</span>
    <span class="n">request</span> <span class="o">=</span> <span class="n">client_connection</span><span class="o">.</span><span class="n">recv</span><span class="p">(</span><span class="mi">1024</span><span class="p">)</span>
    <span class="k">print</span> <span class="n">request</span>

    <span class="n">http_response</span> <span class="o">=</span> <span class="s2">"""</span><span class="se">\</span>
<span class="s2">HTTP/1.1 200 OK</span>

<span class="s2">Hello, World!</span>
<span class="s2">"""</span>
    <span class="n">client_connection</span><span class="o">.</span><span class="n">sendall</span><span class="p">(</span><span class="n">http_response</span><span class="p">)</span>
    <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>
</pre>
</div>
<p>将上面的代码保存为 <em>webserver1.py</em> 或者直接从 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part1/webserver1.py">GitHub</a> 上下载下来，然后在命令行下像这样运行:</p>
<pre class="literal-block">$ python webserver1.py
Serving HTTP on port 8888 …
</pre>
<p>现在在你浏览器的地址栏中输入如下链接<a class="reference external" href="http://localhost:8888/hello">http://localhost:8888/hello</a> ，按回车键然后就可以看到魔法效果了。 你应该会看到在你的浏览器中显示 "Hello, World!” ，就像下面这样：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part1/browser_hello_world.png" alt="browser_hello_world.png" /></p>
<p>放手去做吧，说真的。当你在测试的时候我会等你的。</p>
<p>做完了？非常好。现在让我们讨论一下它实际上是如何工作的。</p>
<p>首先让我们从你输入的 Web 地址开始。它被叫做 <a class="reference external" href="http://en.wikipedia.org/wiki/Uniform_resource_locator">URL</a> ，下面是它的基本结构：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part1/LSBAWS_URL_Web_address.png" alt="LSBAWS_URL_Web_address.png" /></p>
<p>这就是你如何告诉你的浏览器它需要用来查找和连接的 Web server 地址以及需要显示给你的位于服务器上的页面（路径）。 在你的浏览器发送一个 HTTP 请求前，它首先需要与 Web server 建立一条 TCP 连接。 然后再通过这个 TCP 连接发送一个 HTTP 请求到服务器，然后等待服务器发送回一个 HTTP 响应。 当你的浏览器接收到这个响应的时候，它就会显示它。 在这里它将显示 "Hello, World!"</p>
<p>让我们更详细的探索一下客户端和服务器在发送 HTTP 请求和响应之前是如何建立一条 TCP 连接的。 为了达到这个目的，它们都使用了所谓的 sockets 。 为了代替浏览器直连，你可以通过在命令行上使用 telnet 命令的方式来手动模拟浏览器的行为。</p>
<p>在你运行 Web server 的电脑上打开一个 telnet 会话，可以通过在命令行上输入 telent 并指定连接到 localhost 这个主机和 8888 这个端口，然后按下回车键：</p>
<pre class="literal-block">$ telnet localhost 8888
Trying 127.0.0.1 …
Connected to localhost.
</pre>
<p>此刻，你已经与运行在你的本地机器上的准备发送和接收 HTTP 消息的服务器建立了一条 TCP 连接。 在下面的图片中你将看到一套标准的程序，服务器必须遵守这套程序以便能够接受新的 TCP 连接。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part1/LSBAWS_socket.png" alt="LSBAWS_socket.png" /></p>
<p>在相同的 telnet 会话中输入 <tt class="docutils literal">GET /hello HTTP/1.1</tt> 然后按下回车键：</p>
<pre class="literal-block">$ telnet localhost 8888
Trying 127.0.0.1 …
Connected to localhost.
GET /hello HTTP/1.1

HTTP/1.1 200 OK
Hello, World!
</pre>
<p>你刚刚手动模拟了你的浏览器！你发送了一个 HTTP 请求并收到了一个 HTTP 响应。 下面是一个基本的 HTTP 请求的结构：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part1/LSBAWS_HTTP_request_anatomy.png" alt="LSBAWS_HTTP_request_anatomy.png" /></p>
<p>HTTP 请求包含了一个表示 HTTP 方法的行（GET, 因为我们要求我们的服务器返回我们一下东西）， 路径 /hello 表示了服务器上一个我们需要的”页面“，以及协议版本。</p>
<p>为了简单起见，我们的 Web server 在这里完全忽略了上面提到的请求行。 你可以用任何垃圾数据代替 ”GET /hello HTTP/1.1“，你依然可以得到一个内容为 ”Hello, World!“ 的响应。</p>
<p>一旦你输入完请求行并按下回车键，客户端就会把请求发送到服务器，服务器读取请求行，打印出来，并返回合适的 HTTP 响应。</p>
<p>下面是 server 发送回你的客户端(在这里是 telnet)的 HTTP 响应：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part1/LSBAWS_HTTP_response_anatomy.png" alt="LSBAWS_HTTP_response_anatomy.png" /></p>
<p>让我们来分析一下。响应包括一个状态行<tt class="docutils literal">HTTP/1.1  200 OK</tt>, 接下来是一个空行，然后是 HTTP 响应的 body 。</p>
<p>response 状态行 <tt class="docutils literal">HTTP/1.1 200 OK</tt> 包括了 HTTP 版本，HTTP 状态码 以及 HTTP 状态码原因词组 OK。 当浏览器获取到响应时，它将显示响应的 body 部分，这就是为什么你能在你的浏览器中看到 “Hello, World!” 的原因。</p>
<p>这就是一个 Web server 如何工作的基本模型了。总结一下： Web server 创建一个 socket 监听并开始在一个循环里接受新的连接。客户端启动一个 TCP 连接，成功建立连接之后客户端发送一个 HTTP 请求到 server ，然后 server 响应一个展示给用户的 HTTP response 。客户端和服务器都使用 socket 来建立 TCP 连接。</p>
<p>现在你已经有一个非常基础的 Web server 了，你可以用你的浏览器或其他的 HTTP 客户端来测试它。 正如你见过的，如果想尝试的话你也可以通过使用 telent 手动输入 HTTP 请求的方式成为一个人肉 HTTP 客户端。</p>
<p>有个问题要问你：“如何在你这个新鲜出炉的 Web server 上运行一个 Django 应用， Flask 应用，以及 Pyramid 应用，并且不需要做任何的改动就可以适应这些不同的 Web 框架？”</p>







