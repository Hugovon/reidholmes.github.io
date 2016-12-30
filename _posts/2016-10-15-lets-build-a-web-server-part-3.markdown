---
published: true
layout:     post
title:      "构建一个 WEB 服务器（三）"
date:       2016-10-15 20:05:02
categories:
---


<blockquote><p>“当我们必须要发明创造的时候学到的东西最多” —— Piaget</p></blockquote>
<p>在 <a class="reference external" href="https://reidholmes.github.io/2016/10/15/lets-build-a-web-server-part-2/">第二部分</a> 你创建了一个能够处理基本的 GET 请求的微型 WSGI 服务器。 同时我向你提了一个问题，“如何才能让你的服务器在同一时刻能够处理多个请求？” 在本篇文章中你可以找到答案。那么，系好安全带，加足马力吧。 你将会有一趟非常快速的旅程。 确保你已经准备好了你的 Linux, Mac OS X （或任一 *nix 系统）以及 Python 。 本文所有的源代码都已经放在 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/">Github</a> 上了。</p>
<p>首先，让我们来回顾一下一个非常基础的 Web 服务器看起来是啥样的， 以及这个服务器需要做些什么才能服务来自客户端的请求。 你在 <a class="reference external" href="https://reidholmes.github.io/2016/10/15/lets-build-a-web-server-part-1/">第一部分</a> 和 <a class="reference external" href="https://reidholmes.github.io/2016/10/15/lets-build-a-web-server-part-2/">第二部分</a> 创建的服务器是个一次只能处理一个客户端请求的循环服务器。 在它处理完正在处理的客户端请求之前，它是无法接受新的连接的。 一些客户端可能会不高兴，因为它们必须得排队等待， 对于那些非常繁忙的服务器，这个等待可能会是个非常漫长的过程。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it1.png" alt="lsbaws_part3_it1.png" /></p>
<p>下面是循环服务器 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3a.py">webserver3a.py</a> 代码:</p>
<div class="highlight">
<pre><span class="c1">#####################################################################</span>
<span class="c1"># Iterative server - webserver3a.py                                 #</span>
<span class="c1">#                                                                   #</span>
<span class="c1"># Tested with Python 2.7.9 &amp; Python 3.4 on Ubuntu 14.04 &amp; Mac OS X  #</span>
<span class="c1">#####################################################################</span>
<span class="kn">import</span> <span class="nn">socket</span>

<span class="n">SERVER_ADDRESS</span> <span class="o">=</span> <span class="p">(</span><span class="n">HOST</span><span class="p">,</span> <span class="n">PORT</span><span class="p">)</span> <span class="o">=</span> <span class="s1">''</span><span class="p">,</span> <span class="mi">8888</span>
<span class="n">REQUEST_QUEUE_SIZE</span> <span class="o">=</span> <span class="mi">5</span>


<span class="k">def</span> <span class="nf">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">):</span>
    <span class="n">request</span> <span class="o">=</span> <span class="n">client_connection</span><span class="o">.</span><span class="n">recv</span><span class="p">(</span><span class="mi">1024</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">decode</span><span class="p">())</span>
    <span class="n">http_response</span> <span class="o">=</span> <span class="n">b</span><span class="s2">"""</span><span class="se">\</span>
<span class="s2">HTTP/1.1 200 OK</span>

<span class="s2">Hello, World!</span>
<span class="s2">"""</span>
    <span class="n">client_connection</span><span class="o">.</span><span class="n">sendall</span><span class="p">(</span><span class="n">http_response</span><span class="p">)</span>


<span class="k">def</span> <span class="nf">serve_forever</span><span class="p">():</span>
    <span class="n">listen_socket</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">socket</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">AF_INET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SOCK_STREAM</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">setsockopt</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">SOL_SOCKET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SO_REUSEADDR</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">bind</span><span class="p">(</span><span class="n">SERVER_ADDRESS</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">listen</span><span class="p">(</span><span class="n">REQUEST_QUEUE_SIZE</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="s1">'Serving HTTP on port {port} ...'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">port</span><span class="o">=</span><span class="n">PORT</span><span class="p">))</span>

    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="n">client_connection</span><span class="p">,</span> <span class="n">client_address</span> <span class="o">=</span> <span class="n">listen_socket</span><span class="o">.</span><span class="n">accept</span><span class="p">()</span>
        <span class="n">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">)</span>
        <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s1">'__main__'</span><span class="p">:</span>
    <span class="n">serve_forever</span><span class="p">()</span>
</pre>
</div>
<p>为了观察你的服务器一次只能处理一个客户端请求的现象， 服务器代码需要做一点修改，在发送响应信息给客户端后的地方增加了 一个 60 秒的延时。这一行的更改是为了告诉服务器进程需要休息 60 秒。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it2.png" alt="lsbaws_part3_it2.png" /></p>
<p>下面是包含休息代码的服务器代码<a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3b.py">webserver3b.py</a> :</p>
<div class="highlight">
<pre><span class="c1">#########################################################################</span>
<span class="c1"># Iterative server - webserver3b.py                                     #</span>
<span class="c1">#                                                                       #</span>
<span class="c1"># Tested with Python 2.7.9 &amp; Python 3.4 on Ubuntu 14.04 &amp; Mac OS X      #</span>
<span class="c1">#                                                                       #</span>
<span class="c1"># - Server sleeps for 60 seconds after sending a response to a client   #</span>
<span class="c1">#########################################################################</span>
<span class="kn">import</span> <span class="nn">socket</span>
<span class="kn">import</span> <span class="nn">time</span>

<span class="n">SERVER_ADDRESS</span> <span class="o">=</span> <span class="p">(</span><span class="n">HOST</span><span class="p">,</span> <span class="n">PORT</span><span class="p">)</span> <span class="o">=</span> <span class="s1">''</span><span class="p">,</span> <span class="mi">8888</span>
<span class="n">REQUEST_QUEUE_SIZE</span> <span class="o">=</span> <span class="mi">5</span>


<span class="k">def</span> <span class="nf">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">):</span>
    <span class="n">request</span> <span class="o">=</span> <span class="n">client_connection</span><span class="o">.</span><span class="n">recv</span><span class="p">(</span><span class="mi">1024</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">decode</span><span class="p">())</span>
    <span class="n">http_response</span> <span class="o">=</span> <span class="n">b</span><span class="s2">"""</span><span class="se">\</span>
<span class="s2">HTTP/1.1 200 OK</span>

<span class="s2">Hello, World!</span>
<span class="s2">"""</span>
    <span class="n">client_connection</span><span class="o">.</span><span class="n">sendall</span><span class="p">(</span><span class="n">http_response</span><span class="p">)</span>
    <span class="n">time</span><span class="o">.</span><span class="n">sleep</span><span class="p">(</span><span class="mi">60</span><span class="p">)</span>  <span class="c1"># sleep and block the process for 60 seconds</span>


<span class="k">def</span> <span class="nf">serve_forever</span><span class="p">():</span>
    <span class="n">listen_socket</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">socket</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">AF_INET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SOCK_STREAM</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">setsockopt</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">SOL_SOCKET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SO_REUSEADDR</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">bind</span><span class="p">(</span><span class="n">SERVER_ADDRESS</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">listen</span><span class="p">(</span><span class="n">REQUEST_QUEUE_SIZE</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="s1">'Serving HTTP on port {port} ...'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">port</span><span class="o">=</span><span class="n">PORT</span><span class="p">))</span>

    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="n">client_connection</span><span class="p">,</span> <span class="n">client_address</span> <span class="o">=</span> <span class="n">listen_socket</span><span class="o">.</span><span class="n">accept</span><span class="p">()</span>
        <span class="n">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">)</span>
        <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s1">'__main__'</span><span class="p">:</span>
    <span class="n">serve_forever</span><span class="p">()</span>
</pre>
</div>
<p>用以下方式启动服务器：</p>
<pre class="literal-block">$ python webserver3b.py
</pre>
<p>现在打开一个新的终端窗口，然后执行 <tt class="docutils literal">curl</tt>命令。 你应该会看到屏幕上打印了 "Hello, World!" 字符串：</p>
<pre class="literal-block">$ curl http://localhost:8888/hello
Hello, World!
</pre>
<p>然后立即打开第二个终端窗口并执行相同的<tt class="docutils literal">curl</tt> 命令：</p>
<pre class="literal-block">$ curl http://localhost:8888/hello
</pre>
<p>如果你在 60 秒内做完了这些操作的话，第二个 <tt class="docutils literal">curl</tt> 应该不会立马 输出任何的的信息，只是会阻塞在那里。 服务器应该也没有在它的标准输出中答应新的请求 body 信息。 下面是这种现象在我的 Mac 电脑上的样子（右下角用黄色高亮的窗口显示第二个 <tt class="docutils literal">curl</tt> 命令阻塞了， 正在等待连接能够被服务器接受）：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it3.png" alt="lsbaws_part3_it3.png" /></p>
<p>在你等了足够长的时间后（大于 60 秒）你应该会看到第一个 <tt class="docutils literal">curl</tt> 结束 以及第二个 <tt class="docutils literal">curl</tt>在屏幕上打印了 "Hello, World!"，然后阻塞 60 秒，之后再结束：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it4.png" alt="lsbaws_part3_it4.png" /></p>
<p>它的工作方式是，服务器完成对第一个 <tt class="docutils literal">curl</tt>客户端的请求后，只有等它休息完 60 秒之后才会 开始处理第二个请求。这将导致出现顺序，或一步一步的循环，或在这里是一次只能处理一个客户端请求。</p>
<p>让我们抽出一点时间来说一下关于客户端和服务器之间的通信方面的东西。 为了让两个程序能够在网络进行中进行通信，它们需要使用套接字（socket）。 你已经在 <a class="reference external" href="https://reidholmes.github.io/2016/10/15/lets-build-a-web-server-part-1//">第一部分</a>和 <a class="reference external" href="https://reidholmes.github.io/2016/10/15/lets-build-a-web-server-part-2/">第二部分</a> 中见过 socket 了。但是什么是一个 socket 呢？</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it_socket.png" alt="lsbaws_part3_it_socket.png" /></p>
<p>一个 socket 是一个通信端点的抽象概念， 它允许你的程序通过文件描述与另一个程序进行通信。 在这篇文章中，我会特别讲述在 Linux/Mac OS X 上的 TCP/IP socket。 需要理解一个重要的概念，那就是 TCP socket 对。</p>
<blockquote><p>一个 TCP 连接的 socket 对是一个 4 元组，这个元组标识了一个 TCP 连接的两个端点： 本地 IP 地址，本地端口，远程 IP 地址，以及远程端口。 一个套接字对唯一标识了网络上每个 TCP 连接。 一个 IP 地址和一个端口号这个两个值标识了每个端点，通常被叫做一个套接字。</p></blockquote>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it_socket.png" alt="lsbaws_part3_it_socketpair.png" /></p>
<p>因此，元组 <tt class="docutils literal">{10.10.10.2:49152, 12.12.12.3:8888}</tt> 是一个套接字对，它唯一标识了 客户端 上的一个 TCP 连接的两个端点。 元组 <tt class="docutils literal">{12.12.12.3:8888, 10.10.10.2:49152}</tt> 是一个套接字对，它唯一标识了 服务端 上的一个 TCP 连接的两个端点。这个两个值标识了一个 TCP 连接的服务端端点， IP 地址 12.12.12.3 和端口 8888 在这里被归为一个套接字（客户端端点有相同的应用）。</p>
<p>一个服务器通常通过创建一个套接字,然后开始接受来自客户端的请求，它的常规顺序如下：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it_server_socket_sequence.png" alt="lsbaws_part3_it_server_socket_sequence.png" /></p>
<ol class="arabic">
<li>
<p class="first">服务器创建一个 TCP/IP socket。这个用的是下面的 Python 语句来实现的：</p>
<pre class="literal-block">listen_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
</pre>
</li>
<li>
<p class="first">服务器可能会设置一些 socket 选项（这是可选的，但是你能在上面的服务器代码中看到它,只是为了在你决定杀死或重启服务器的时候能够立即就可以一遍又一遍的重复使用相同的地址）。:</p>
<pre class="literal-block">listen_socket.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
</pre>
</li>
<li>
<p class="first">然后，服务器绑定这个地址. <tt class="docutils literal">bind</tt> 函数在 socket 上分配一个本地协议地址。对于 TCP 则是，调用 <tt class="docutils literal">bind</tt> 让你指定一个端口号，一个 IP 地址，这两个都要或都不需要指定。:</p>
<pre class="literal-block">listen_socket.bind(SERVER_ADDRESS)
</pre>
</li>
<li>
<p class="first">然后，服务器把这个 socket 设定为一个监听 socket</p>
<pre class="literal-block">listen_socket.listen(REQUEST_QUEUE_SIZE)
</pre>
</li>
</ol>
<p>这个 <tt class="docutils literal">listen</tt> 方法只需要在服务器端进行调用。 它告诉内核，它应该接受目标为这个 socket 的接入连接请求。</p>
<p>这样做以后，服务器就开始在一个循环内一次接受来自一个客户端的连接。 当有一个可用的连接的时候， <tt class="docutils literal">accept</tt> 调用返回连接的客户端的套接字。 然后，服务器从这个连接的客户端的套接字中读取请求数据， 在它的标准输出上答应这个数据，然后给客户端发送回一条消息。 然后，服务器关闭了这个客户端连接，它准备再次开始接受来的新客户端的连接。</p>
<p>下面是一个客户端与服务器通过 TCP/IP 进行通信需要做的事情：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it_client_socket_sequence.png" alt="lsbaws_part3_it_client_socket_sequence.png" /></p>
<p>下面是客户端连接你的服务器，发送一个请求然后答应响应的一段示例代码：</p>
<div class="highlight">
<pre><span class="kn">import</span> <span class="nn">socket</span>

<span class="c1"># create a socket and connect to a server</span>
<span class="n">sock</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">socket</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">AF_INET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SOCK_STREAM</span><span class="p">)</span>
<span class="n">sock</span><span class="o">.</span><span class="n">connect</span><span class="p">((</span><span class="s1">'localhost'</span><span class="p">,</span> <span class="mi">8888</span><span class="p">))</span>

<span class="c1"># send and receive some data</span>
<span class="n">sock</span><span class="o">.</span><span class="n">sendall</span><span class="p">(</span><span class="n">b</span><span class="s1">'test'</span><span class="p">)</span>
<span class="n">data</span> <span class="o">=</span> <span class="n">sock</span><span class="o">.</span><span class="n">recv</span><span class="p">(</span><span class="mi">1024</span><span class="p">)</span>
<span class="k">print</span><span class="p">(</span><span class="n">data</span><span class="o">.</span><span class="n">decode</span><span class="p">())</span>
</pre>
</div>
<p>创建完套接字后，客户端需要连接服务器. 可以通过调用 <tt class="docutils literal">connect</tt> 实现这个功能：</p>
<pre class="literal-block">sock.connect(('localhost', 8888))
</pre>
<p>客户端只需要提供想要连接的服务器的远程 IP 地址或主机名以及远程端口号就可以了。</p>
<p>你可能已经注意到了，客户端没有调用 <tt class="docutils literal">bind</tt>和 <tt class="docutils literal">accept</tt>。 客户不需要调用 <tt class="docutils literal">bind</tt> 是因为客户端不关系本地 IP 地址和本地端口号。 当客户端调用 <tt class="docutils literal">connect</tt> 时，内核里的 TCP/IP 协议栈会分配本地 IP 地址和本地端口号。 这个本地端口叫做 <strong>临时端口</strong> 或者 短命端口 :)。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it_ephemeral_port.png" alt="lsbaws_part3_it_ephemeral_port.png" /></p>
<p>服务器上的端口标识了一个知名(well-know)服务，客户端连接的端口叫做一个知名端口（比如，HTTP 的 80 端口， SSH 的 22 端口）。 起一个 Python shell 然后发起一个到你本地运行的服务器的客户端连接， 然后查看为你创建的套接字分配了什么样的临时端口（在尝试下面的例子前需要先启动服务器<a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3a.py">webserver3a.py</a> 或 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3b.py">webserver3b.py</a> ）：</p>
<pre class="literal-block">&gt;&gt;&gt; import socket
&gt;&gt;&gt; sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
&gt;&gt;&gt; sock.connect(('localhost', 8888))
&gt;&gt;&gt; host, port = sock.getsockname()[:2]
&gt;&gt;&gt; host, port
('127.0.0.1', 60589)
</pre>
<p>在上面的例子中，内核给那个套接字分配了一个临时端口 60589。</p>
<p>在回答 <a class="reference external" href="https://reidholmes.github.io/2016/10/15/lets-build-a-web-server-part-2/">第二部分</a> 的问题前我还需要快速讲一下其他一些重要的概念。 你很快就会看到为什么它们是重要的。 这两个概念是 <em>进程</em> 和<em>文件描述符</em> 。</p>
<p>什么是进程？一个进程只是一个正在执行的程序的实例。 比如说，当服务器代码被执行的时候，它被加载到内存里，然后这个正在执行的程序的实例就叫做进程。 内核记录了有关这个进程的一大串的信息—— 比如进程的 ID ——为了方便跟踪这个进程。 当你运行你的循环服务器 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3a.py">webserver3a.py</a> 或<a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3b.py">webserver3b.py</a> 的时候，你只是运行了一个进程。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it_server_process.png" alt="lsbaws_part3_it_server_process.png" /></p>
<p>在终端窗口中启动服务器 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3b.py">webserver3b.py</a> :</p>
<pre class="literal-block">$ python webserver3b.py
</pre>
<p>在另一个不同的终端窗口中使用 <tt class="docutils literal">ps</tt> 命令来获取刚才那个进程的一些信息:</p>
<pre class="literal-block">$ ps | grep webserver3b | grep -v grep
7182 ttys003    0:00.04 python webserver3b.py
</pre>
<p><tt class="docutils literal">ps</tt> 命令告诉你你确实只是运行了一个 Python 进程 webserver3b 。 当一个进程被创建的时候，内核给它分配了一个进程 ID, PID。 在 UNIX 中，每个用户进程同时也有一个父进程，这个进程有它自己的进程 ID 叫做父进程 ID 或缩写为 PPID。 一般情况下，我假定你运行了一个 BASH shell ，然后当你启用服务器的时候， 一个新的进程被创建并且有一个 PID，同时它的父 PID 其实就是那个 BASH shel 的 PID。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it_ppid_pid.png" alt="lsbaws_part3_it_ppid_pid.png" /></p>
<p>亲自试一下，然后看看具体是什么情况。 再次启动你的 Python shell ，这将创建一个新的进程， 然后通过调用 <a class="reference external" href="https://docs.python.org/2.7/library/os.html#os.getpid">os.getpid()</a> 和<a class="reference external" href="https://docs.python.org/2.7/library/os.html#os.getppid">os.getppid()</a> 来获取这个 Python shell 进程的 PID 和父 PID（你的 BASH shell 的 PID）。 然后在另一个终端窗口中运行 <tt class="docutils literal">ps</tt> 和 <tt class="docutils literal">grep</tt> 命令来获取 PPID（父进程 ID，在我的这里是 3148）。 在下面的截图中你可以看到一个父子关系的例子， 它展示的是在我的 Mac OS X 机器上子 Python shell 进程与父 BASH shell 进程之间的父子关系：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it_pid_ppid_screenshot.png" alt="lsbaws_part3_it_pid_ppid_screenshot.png" /></p>
<p>另一个非常重要并且需要了解的概念是 <strong>文件描述符</strong> 。 那么，什么是文件描述符呢？ 一个文件描述符是一个正整数， 当一个进程打开一个存在的文件，创建一个新文件或创建一个新的套接字的时候，内核返回一个正整数给进程，这个正整数就是文件描述符。 你可能听说过，在 UNIX 中一切皆文件。 内核通过文件描述符来索引一个进程打开的文件。 当你需要读或写一个文件时，你需要用文件描述符来标记它。 Python 给了你一些更高级别的对象用来处理文件（和套接字）， 你不需要使用文件描述符来标识一个文件。 下面展示了在 Unix 中文件和套接字是如何被标识的：通过它们的整数文件描述符。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it_process_descriptors.png" alt="lsbaws_part3_it_process_descriptors.png" /></p>
<p>默认情况下，UNIX shell 给一个进程的标准输出分配的文件描述符是 0， 标准输入的文件描述符是 1，标准错误的文件描述符是 2。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it_default_descriptors.png" alt="lsbaws_part3_it_default_descriptors.png" /></p>
<p>正如我前面提到的，尽管 Python 给了你一个更高级别的文件或类文件对象用来进行操作， 你依然可以使用对象的 <tt class="docutils literal">fileno()</tt> 方法来获取分配给这个文件的文件描述符。 返回到你的 Python shell 看看你怎样才能做到这样：</p>
<pre class="literal-block">&gt;&gt;&gt; import sys
&gt;&gt;&gt; sys.stdin
&lt;open file '&lt;stdin&gt;', mode 'r' at 0x102beb0c0&gt;
&gt;&gt;&gt; sys.stdin.fileno()
0
&gt;&gt;&gt; sys.stdout.fileno()
1
&gt;&gt;&gt; sys.stderr.fileno()
2
</pre>
<p>当你在 Python 中处理文件和套接字的时候，你通常需要使用一个高级别的 file/socket 对象。 但是在这里你可能需要多次直接使用文件描述符。 下面的例子展示了你可以通过调用一个 <a class="reference external" href="https://docs.python.org/2.7/library/os.html#os.write">write</a> 系统并把文件描述符正数作为一个参数 的方式来写入一个字符串到标准输出：</p>
<pre class="literal-block">&gt;&gt;&gt; import sys
&gt;&gt;&gt; import os
&gt;&gt;&gt; res = os.write(sys.stdout.fileno(), 'hello\n')
hello
</pre>
<p>这里是个非常有意思的地方——这应该不会让你感到特别的惊讶，因为你已经知道在 UNIX 中万物皆文件——你的套接字也有一个分配给它的文件描述符。 在说一遍，当你在 Python 中创建一个套接字的时候， 你得到了一个对象和一个正整数， 你也可以通过直接访问我之前提过的 <tt class="docutils literal">fileno()</tt> 方法的方式得到这个套接字的整数文件描述符。</p>
<pre class="literal-block">&gt;&gt;&gt; import socket
&gt;&gt;&gt; sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
&gt;&gt;&gt; sock.fileno()
3
</pre>
<p>还有一件需要提及的事情是：在第二个例子中的循环服务器 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3b.py">webserver3b.py</a> 中你已经知道了，当服务器进程正在休眠 60 秒的时候，你仍然能够用第二个 <tt class="docutils literal">curl</tt> 命令连接服务器吗？当然可以，只是 <tt class="docutils literal">curl</tt> 将不会立即输出任何信息，它只是会阻塞在那里。 但是，为什么当时服务器并没有在 <tt class="docutils literal">accept</tt> 接受一个连接，而客户端却没有立即被拒绝连接，却依然能够连接服务器？ 答案是套接字对象的<tt class="docutils literal">listen</tt> 方法以及它的 <tt class="docutils literal">BACKLOG</tt> 参数，在代码里面我调用的是 <tt class="docutils literal">REQUEST_QUEUE_SIZE</tt> 。<tt class="docutils literal">BACKLOG</tt> 参数指定了内核中接入连接请求的队列大小。 当服务器 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3b.py">webserver3b.py</a> 正在休眠的时候，你可以用第二个 <tt class="docutils literal">curl</tt> 命令 连接服务器，是因为内核中用于该服务器套接字的接入连接请求队列还有足够的可用空间。</p>
<p>虽然加大 <tt class="docutils literal">BACKLOG</tt> 参数的值并不能把你的服务器变成一个可以一次处理多个客户端请求的服务器，但是，对于非常繁忙的服务器来说，有一个足够大 的 backlog 参数是非常重要的，这样 <tt class="docutils literal">accept</tt> 调用就不用等待有新的连接被建立，可以直接从队列中拿取新的连接，然后开始没有延时的处理这个客户端请求。</p>
<p>哇哦！你已经了解到很多知识了。让我们快速回顾一下你目前说学到的东西（ 如果这些你都非常熟悉的话，那就再加深一下）。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_checkpoint.png" alt="lsbaws_part3_checkpoint.png" /></p>
<ul class="simple">
<li>循环服务器</li>
<li>服务器 socket 创建顺序（socket, bind, listen, accept）</li>
<li>客户端链接创建顺序（socket, connect）</li>
<li>套接字对</li>
<li>套接字</li>
<li>临时端口和知名端口</li>
<li>进程</li>
<li>进程 ID（PID），父进程 ID（PPID），以及父子关系。</li>
<li>文件描述符</li>
<li><tt class="docutils literal">listen</tt> socket 方法的 <tt class="docutils literal">BACKLOG</tt> 参数的含义</li>
</ul>
<p>现在，我已经准备号回答 <a class="reference external" href="https://reidholmes.github.io/2016/10/15/lets-build-a-web-server-part-2/">第二部分</a> 的问题了：“如何让你的服务器在同一时刻 处理多个请求？” 或者换一种方式“如何写一个并发服务器？”</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc2_service_clients.png" alt="lsbaws_part3_conc2_service_clients.png" /></p>
<p>在 Unix 下写一个并发服务器的最简单的方法是使用 <a class="reference external" href="https://docs.python.org/2.7/library/os.html#os.fork">fork()</a> 系统调用。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_fork.png" alt="lsbaws_part3_fork.png" /></p>
<p>下面是一个新的并发服务器 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3c.py">webserver3c.py</a>的代码，这个服务器能够同时处理多个客户端请求（同上一个服务器 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3b.py">webserver3b.py</a> 一样，每个子进程都会休息 60 秒 ）：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_it2.png" alt="lsbaws_part3_it2.png" /></p>
<div class="highlight">
<pre><span class="c1">###########################################################################</span>
<span class="c1"># Concurrent server - webserver3c.py                                      #</span>
<span class="c1">#                                                                         #</span>
<span class="c1"># Tested with Python 2.7.9 &amp; Python 3.4 on Ubuntu 14.04 &amp; Mac OS X        #</span>
<span class="c1">#                                                                         #</span>
<span class="c1"># - Child process sleeps for 60 seconds after handling a client's request #</span>
<span class="c1"># - Parent and child processes close duplicate descriptors                #</span>
<span class="c1">#                                                                         #</span>
<span class="c1">###########################################################################</span>
<span class="kn">import</span> <span class="nn">os</span>
<span class="kn">import</span> <span class="nn">socket</span>
<span class="kn">import</span> <span class="nn">time</span>

<span class="n">SERVER_ADDRESS</span> <span class="o">=</span> <span class="p">(</span><span class="n">HOST</span><span class="p">,</span> <span class="n">PORT</span><span class="p">)</span> <span class="o">=</span> <span class="s1">''</span><span class="p">,</span> <span class="mi">8888</span>
<span class="n">REQUEST_QUEUE_SIZE</span> <span class="o">=</span> <span class="mi">5</span>


<span class="k">def</span> <span class="nf">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">):</span>
    <span class="n">request</span> <span class="o">=</span> <span class="n">client_connection</span><span class="o">.</span><span class="n">recv</span><span class="p">(</span><span class="mi">1024</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span>
        <span class="s1">'Child PID: {pid}. Parent PID {ppid}'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span>
            <span class="n">pid</span><span class="o">=</span><span class="n">os</span><span class="o">.</span><span class="n">getpid</span><span class="p">(),</span>
            <span class="n">ppid</span><span class="o">=</span><span class="n">os</span><span class="o">.</span><span class="n">getppid</span><span class="p">(),</span>
        <span class="p">)</span>
    <span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">decode</span><span class="p">())</span>
    <span class="n">http_response</span> <span class="o">=</span> <span class="n">b</span><span class="s2">"""</span><span class="se">\</span>
<span class="s2">HTTP/1.1 200 OK</span>

<span class="s2">Hello, World!</span>
<span class="s2">"""</span>
    <span class="n">client_connection</span><span class="o">.</span><span class="n">sendall</span><span class="p">(</span><span class="n">http_response</span><span class="p">)</span>
    <span class="n">time</span><span class="o">.</span><span class="n">sleep</span><span class="p">(</span><span class="mi">60</span><span class="p">)</span>


<span class="k">def</span> <span class="nf">serve_forever</span><span class="p">():</span>
    <span class="n">listen_socket</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">socket</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">AF_INET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SOCK_STREAM</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">setsockopt</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">SOL_SOCKET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SO_REUSEADDR</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">bind</span><span class="p">(</span><span class="n">SERVER_ADDRESS</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">listen</span><span class="p">(</span><span class="n">REQUEST_QUEUE_SIZE</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="s1">'Serving HTTP on port {port} ...'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">port</span><span class="o">=</span><span class="n">PORT</span><span class="p">))</span>
    <span class="k">print</span><span class="p">(</span><span class="s1">'Parent PID (PPID): {pid}</span><span class="se">\n</span><span class="s1">'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">pid</span><span class="o">=</span><span class="n">os</span><span class="o">.</span><span class="n">getpid</span><span class="p">()))</span>

    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="n">client_connection</span><span class="p">,</span> <span class="n">client_address</span> <span class="o">=</span> <span class="n">listen_socket</span><span class="o">.</span><span class="n">accept</span><span class="p">()</span>
        <span class="n">pid</span> <span class="o">=</span> <span class="n">os</span><span class="o">.</span><span class="n">fork</span><span class="p">()</span>
        <span class="k">if</span> <span class="n">pid</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>  <span class="c1"># child</span>
            <span class="n">listen_socket</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>  <span class="c1"># close child copy</span>
            <span class="n">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">)</span>
            <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>
            <span class="n">os</span><span class="o">.</span><span class="n">_exit</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>  <span class="c1"># child exits here</span>
        <span class="k">else</span><span class="p">:</span>  <span class="c1"># parent</span>
            <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>  <span class="c1"># close parent copy and loop over</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s1">'__main__'</span><span class="p">:</span>
    <span class="n">serve_forever</span><span class="p">()</span>
</pre>
</div>
<p>在深入讨论 fork 是如何工作前，先试一下吧，看看服务器是否真的能够同时处理多个客户端发送过来的请求， 而不是想它的同胞<a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3a.py">webserver3a.py</a> 和 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3b.py">webserver3b.py</a> 那样无法处理。 在命令行下用以下命令启动服务器：</p>
<pre class="literal-block">$ python webserver3c.py
</pre>
<p>然后尝试你之前试过的那两条同样的命令，现在尽管服务器的子进程在服务了一个客户端请求后会休眠 60 秒，但是仍然不 影响其它的客户端，因为它们是由完全没有依赖的不同进程服务的。你可以看到你的 curl 命令立即输出了 “Hello, World!”， 然后卡住了 60 秒。你可以继续执行 n 条 curl 命令（嗯，差不多是你想执行多少就多少）所有的这些命令都将立即输出服务器的响应“Hello, World”，没有肉眼可见的延迟。试试吧。</p>
<p>为了理解 <a class="reference external" href="https://docs.python.org/2.7/library/os.html#os.fork">fork()</a> 有一点很长的重要那就是，你调用了一次 fork 但是它返回了两次：一次在父进程，一次在子进程。 当你 fork 一个新进程时，返回给子进程的进程 ID 是 0。当 fork 在父进程中返回时，它返回的是子进程的 PID。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc2_how_fork_works.png" alt="lsbaws_part3_conc2_how_fork_works.png" /></p>
<p>我仍然记得当我第一次研究 fork 并尝试它的时候我有多么的着迷。它对我看起来就像魔法一样。</p>
<p>我当时正在阅读一段连续的代码，然后“嘭！”：那段代码克隆了它自己，现在同时运行了两个具有相同代码的实例。 我当时真的认为这就是魔法。</p>
<p>当一个父进程 fork 一个新的子进程时，子进程获得了一份父进程的文件描述符的拷贝：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc2_shared_descriptors.png" alt="lsbaws_part3_conc2_shared_descriptors.png" /></p>
<p>你可能已经看到了，在上面代码中父进程关闭了客户端连接：</p>
<pre class="literal-block">else:  # parent
    client_connection.close()  # close parent copy and loop over
</pre>
<p>如果父进程已经关掉了这个 socket 那么子进程是怎么做仍然可以从客户端 socet 中读取到数据呢？ 答案就在上图总。内核使用描述符引用计数来决定是否需要关闭一个 socket。 只有当某个 socket 的描述符引用计数变成 0 的时候才会关闭这个 socket。 当你的服务器创建一个子进程的时候，子进程获得了父进程的文件描述符拷贝，内核将这些描述符的引用计数也相应的增加了。 在有一个父进程和一个子进程的情况下，关联者客户端 socket 的描述符引用计数就会是 2， 当父进程想在上面的代码中那样关闭了客户端 socket 链接的时候，引用计数就会减少变成 1，但是 仍然还没达到让内核关闭这个 socket 的条件。 子进程也需要关闭来自父进程监听的 socket 拷贝，因为子进程不关心接收新的客户端请求， 它只关心处理来自已建立连接的客户端连接：</p>
<pre class="literal-block">listen_socket.close()  # close child copy
</pre>
<p>我将会在稍后讲述如果你不关闭描述符副本时话会发生什么。</p>
<p>正如你在这个并发服务器源码中说发现的那样，服务器的父进程现在只有一个角色，那就是 接收一个新的客户端连接， fork 一个新的子进程用来处理这个请求，然后循环以便接收另一个客户端的连接， 没有其他多余的事情了。服务器的父进程不会处理客户端请求 —— 它的子进程会去处理。</p>
<p>说点额外的事情。当我们说两个事件是并发执行的时候，具体说的是什么意思呢？</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc2_concurrent_events.png" alt="lsbaws_part3_conc2_concurrent_events.png" /></p>
<p>当我们说两个事件是并发执行的时候，通常我们的意思是，它们是同时发生的。 简短的定义当然非常好，但是你也应该记住复杂的定义：</p>
<blockquote><p>如果你没法通过观察程序来知道哪个是先执行的，那么这两个事件就是并发执行的。</p></blockquote>
<p>又到了概况你目前所学知识的时间了。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_checkpoint.png" alt="lsbaws_part3_checkpoint.png" /></p>
<ul class="simple">
<li>在 Unix 下写并发服务器的最简单的方法是调用系统内的 <a class="reference external" href="https://docs.python.org/2.7/library/os.html#os.fork">fork()</a> 方法</li>
<li>当一个进程 fork 了一个新的进程的时候，它就变成了那个新 fork 的子进程的父进程。</li>
<li>在调用 fork 后父子进程共享相同的文件描述符。</li>
<li>内核使用描述符引用计数来决定是否需要关闭文件/socket</li>
<li>服务器的父进程现在只有一个角色，那就是接收一个新的客户端连接， fork 一个新的子进程用来处理这个请求，然后循环以便接收另一个客户端的连接。</li>
</ul>
<p>让我们来看一下，如果你没有在父子进程中关闭 socket 描述符副本会发生什么。 下面是一个修改版的并发服务器，它没有关闭描述符副本， <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3d.py">webserver3d.py</a> ：</p>
<div class="highlight">
<pre><span class="c1">###########################################################################</span>
<span class="c1"># Concurrent server - webserver3d.py                                      #</span>
<span class="c1">#                                                                         #</span>
<span class="c1"># Tested with Python 2.7.9 &amp; Python 3.4 on Ubuntu 14.04 &amp; Mac OS X        #</span>
<span class="c1">###########################################################################</span>
<span class="kn">import</span> <span class="nn">os</span>
<span class="kn">import</span> <span class="nn">socket</span>

<span class="n">SERVER_ADDRESS</span> <span class="o">=</span> <span class="p">(</span><span class="n">HOST</span><span class="p">,</span> <span class="n">PORT</span><span class="p">)</span> <span class="o">=</span> <span class="s1">''</span><span class="p">,</span> <span class="mi">8888</span>
<span class="n">REQUEST_QUEUE_SIZE</span> <span class="o">=</span> <span class="mi">5</span>


<span class="k">def</span> <span class="nf">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">):</span>
    <span class="n">request</span> <span class="o">=</span> <span class="n">client_connection</span><span class="o">.</span><span class="n">recv</span><span class="p">(</span><span class="mi">1024</span><span class="p">)</span>
    <span class="n">http_response</span> <span class="o">=</span> <span class="n">b</span><span class="s2">"""</span><span class="se">\</span>
<span class="s2">HTTP/1.1 200 OK</span>

<span class="s2">Hello, World!</span>
<span class="s2">"""</span>
    <span class="n">client_connection</span><span class="o">.</span><span class="n">sendall</span><span class="p">(</span><span class="n">http_response</span><span class="p">)</span>


<span class="k">def</span> <span class="nf">serve_forever</span><span class="p">():</span>
    <span class="n">listen_socket</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">socket</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">AF_INET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SOCK_STREAM</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">setsockopt</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">SOL_SOCKET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SO_REUSEADDR</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">bind</span><span class="p">(</span><span class="n">SERVER_ADDRESS</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">listen</span><span class="p">(</span><span class="n">REQUEST_QUEUE_SIZE</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="s1">'Serving HTTP on port {port} ...'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">port</span><span class="o">=</span><span class="n">PORT</span><span class="p">))</span>

    <span class="n">clients</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="n">client_connection</span><span class="p">,</span> <span class="n">client_address</span> <span class="o">=</span> <span class="n">listen_socket</span><span class="o">.</span><span class="n">accept</span><span class="p">()</span>
        <span class="c1"># store the reference otherwise it's garbage collected</span>
        <span class="c1"># on the next loop run</span>
        <span class="n">clients</span><span class="o">.</span><span class="n">append</span><span class="p">(</span><span class="n">client_connection</span><span class="p">)</span>
        <span class="n">pid</span> <span class="o">=</span> <span class="n">os</span><span class="o">.</span><span class="n">fork</span><span class="p">()</span>
        <span class="k">if</span> <span class="n">pid</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>  <span class="c1"># child</span>
            <span class="n">listen_socket</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>  <span class="c1"># close child copy</span>
            <span class="n">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">)</span>
            <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>
            <span class="n">os</span><span class="o">.</span><span class="n">_exit</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>  <span class="c1"># child exits here</span>
        <span class="k">else</span><span class="p">:</span>  <span class="c1"># parent</span>
            <span class="c1"># client_connection.close()</span>
            <span class="k">print</span><span class="p">(</span><span class="nb">len</span><span class="p">(</span><span class="n">clients</span><span class="p">))</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s1">'__main__'</span><span class="p">:</span>
    <span class="n">serve_forever</span><span class="p">()</span>
</pre>
</div>
<p>用以下方式启动服务器：</p>
<pre class="literal-block">$ python webserver3d.py
</pre>
<p>使用 <tt class="docutils literal">curl</tt> 命令来连接服务器：</p>
<pre class="literal-block">$ curl http://localhost:8888/hello
Hello, World!
</pre>
<p>好了，curl 打印了来自并发服务器的响应，但是它并没有立即退出而是卡住那儿了。 发生什么事情了？服务器不再休息 60 了：它的子进程还在处理客户端请求， 关闭客户端连接然后退出，但是客户端 curl 仍然没有退出。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc3_child_is_active.png" alt="lsbaws_part3_conc3_child_is_active.png" /></p>
<p>那么，为什么 curl 没有退出呢？原因就是文件描述符副本。 当子进程关闭客户端连接的时候，内核减少了那个客户端 socket 的引用计数，此时计数变成了 1。 虽然服务器的子进程退出了，但是客户端 socket 并没有被内核关闭，因为此时该 socket 描述符的引用计数还不是 0， 结果终止包（在 TCP/IP 中被叫做 FIN）并没有被发送给客户端，可以说是客户端就会一直在线。 这里还有另外一个问题。如果你的长时间运行的服务器没有关闭文件描述符副本的话， 它最终将用尽所有可用的文件描述符：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc3_out_of_descriptors.png" alt="lsbaws_part3_conc3_out_of_descriptors.png" /></p>
<p>使用 ctrl + c 停止你的服务器 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3d.py">webserver3d.py</a>，然后通过在 shell 中输入内置的 <tt class="docutils literal">ulimit</tt> 命令来查看服务器进程默认可用的资源：</p>
<pre class="literal-block">$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 3842
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 3842
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
</pre>
<p>正如你在上面看到的，服务器进程在我的 Ubuntu 上最大可打开的文件描述符（open files）数目是 1024。</p>
<p>现在我们来看一下如果你的服务器没有关闭描述符副本，它是怎么样用尽可用的文件描述符的。 在一个已有的或新开的终端窗口中，设置最大可打开的文件描述符数目为 256：</p>
<pre class="literal-block">$ ulimit -n 256
</pre>
<p>在你刚执行 <tt class="docutils literal">$ ulimit <span class="pre">-n</span> 256</tt> 命令的那个终端中启动服务器 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3d.py">webserver3d.py</a> :</p>
<pre class="literal-block">$ python webserver3d.py
</pre>
<p>然后使用下面的客户端 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/client3.py">client3.py</a> 测试这个服务器。</p>
<div class="highlight">
<pre><span class="c1">#####################################################################</span>
<span class="c1"># Test client - client3.py                                          #</span>
<span class="c1">#                                                                   #</span>
<span class="c1"># Tested with Python 2.7.9 &amp; Python 3.4 on Ubuntu 14.04 &amp; Mac OS X  #</span>
<span class="c1">#####################################################################</span>
<span class="kn">import</span> <span class="nn">argparse</span>
<span class="kn">import</span> <span class="nn">errno</span>
<span class="kn">import</span> <span class="nn">os</span>
<span class="kn">import</span> <span class="nn">socket</span>


<span class="n">SERVER_ADDRESS</span> <span class="o">=</span> <span class="s1">'localhost'</span><span class="p">,</span> <span class="mi">8888</span>
<span class="n">REQUEST</span> <span class="o">=</span> <span class="n">b</span><span class="s2">"""</span><span class="se">\</span>
<span class="s2">GET /hello HTTP/1.1</span>
<span class="s2">Host: localhost:8888</span>

<span class="s2">"""</span>


<span class="k">def</span> <span class="nf">main</span><span class="p">(</span><span class="n">max_clients</span><span class="p">,</span> <span class="n">max_conns</span><span class="p">):</span>
    <span class="n">socks</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="k">for</span> <span class="n">client_num</span> <span class="ow">in</span> <span class="nb">range</span><span class="p">(</span><span class="n">max_clients</span><span class="p">):</span>
        <span class="n">pid</span> <span class="o">=</span> <span class="n">os</span><span class="o">.</span><span class="n">fork</span><span class="p">()</span>
        <span class="k">if</span> <span class="n">pid</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>
            <span class="k">for</span> <span class="n">connection_num</span> <span class="ow">in</span> <span class="nb">range</span><span class="p">(</span><span class="n">max_conns</span><span class="p">):</span>
                <span class="n">sock</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">socket</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">AF_INET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SOCK_STREAM</span><span class="p">)</span>
                <span class="n">sock</span><span class="o">.</span><span class="n">connect</span><span class="p">(</span><span class="n">SERVER_ADDRESS</span><span class="p">)</span>
                <span class="n">sock</span><span class="o">.</span><span class="n">sendall</span><span class="p">(</span><span class="n">REQUEST</span><span class="p">)</span>
                <span class="n">socks</span><span class="o">.</span><span class="n">append</span><span class="p">(</span><span class="n">sock</span><span class="p">)</span>
                <span class="k">print</span><span class="p">(</span><span class="n">connection_num</span><span class="p">)</span>
                <span class="n">os</span><span class="o">.</span><span class="n">_exit</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>


<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s1">'__main__'</span><span class="p">:</span>
    <span class="n">parser</span> <span class="o">=</span> <span class="n">argparse</span><span class="o">.</span><span class="n">ArgumentParser</span><span class="p">(</span>
        <span class="n">description</span><span class="o">=</span><span class="s1">'Test client for LSBAWS.'</span><span class="p">,</span>
        <span class="n">formatter_class</span><span class="o">=</span><span class="n">argparse</span><span class="o">.</span><span class="n">ArgumentDefaultsHelpFormatter</span><span class="p">,</span>
    <span class="p">)</span>
    <span class="n">parser</span><span class="o">.</span><span class="n">add_argument</span><span class="p">(</span>
        <span class="s1">'--max-conns'</span><span class="p">,</span>
        <span class="nb">type</span><span class="o">=</span><span class="nb">int</span><span class="p">,</span>
        <span class="n">default</span><span class="o">=</span><span class="mi">1024</span><span class="p">,</span>
        <span class="n">help</span><span class="o">=</span><span class="s1">'Maximum number of connections per client.'</span>
    <span class="p">)</span>
    <span class="n">parser</span><span class="o">.</span><span class="n">add_argument</span><span class="p">(</span>
        <span class="s1">'--max-clients'</span><span class="p">,</span>
        <span class="nb">type</span><span class="o">=</span><span class="nb">int</span><span class="p">,</span>
        <span class="n">default</span><span class="o">=</span><span class="mi">1</span><span class="p">,</span>
        <span class="n">help</span><span class="o">=</span><span class="s1">'Maximum number of clients.'</span>
    <span class="p">)</span>
    <span class="n">args</span> <span class="o">=</span> <span class="n">parser</span><span class="o">.</span><span class="n">parse_args</span><span class="p">()</span>
    <span class="n">main</span><span class="p">(</span><span class="n">args</span><span class="o">.</span><span class="n">max_clients</span><span class="p">,</span> <span class="n">args</span><span class="o">.</span><span class="n">max_conns</span><span class="p">)</span>
</pre>
</div>
<p>在一个新的终端窗口中，启动 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/client3.py">client3.py</a> 并告诉它同时创建 300 个连接到服务器的连接：</p>
<pre class="literal-block">$ python client3.py --max-clients=300
</pre>
<p>很快你的服务器就会爆炸。下面是我机子上的异常信息的截图：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc3_too_many_fds_exc.png" alt="lsbaws_part3_conc3_too_many_fds_exc.png" /></p>
<p>教训已经非常清晰了——你的服务器应该关闭描述符副本。但是，就算你关闭了描述符副本， 你仍然还没有跳出丛林，因为你的服务器还有另一个问题，这个问题就是僵尸进程！</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc3_zombies.png" alt="lsbaws_part3_conc3_zombies.png" /></p>
<p>是的，你的服务器代码实际上创建了一些僵尸进程。 让我们来看一下是怎么回事。再次启动你的服务器：</p>
<pre class="literal-block">$ python webserver3d.py
</pre>
<p>在另一个终端窗口中执行下面的 <tt class="docutils literal">curl</tt> 命令：</p>
<pre class="literal-block">$ curl http://localhost:8888/hello
</pre>
<p>现在使用 ps 命令来显示正在运行的 Python 进程。 下面是我的 Ubuntu 上的 <tt class="docutils literal">ps</tt> 命令输出：</p>
<pre class="literal-block">$ ps auxw | grep -i python | grep -v grep
vagrant   9099  0.0  1.2  31804  6256 pts/0    S+   16:33   0:00 python webserver3d.py
vagrant   9102  0.0  0.0      0     0 pts/0    Z+   16:33   0:00 [python] &lt;defunct&gt;
</pre>
<p>你有注意到第二行吗？上面说进程 PID 为 9102 的进程状态是 <strong>Z+</strong> ，进程名称是<strong>&lt;defunct&gt;</strong> 。 这就是我们的僵尸进程了。僵尸进程的问题是你没法杀死它们。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc3_kill_zombie.png" alt="lsbaws_part3_conc3_kill_zombie.png" /></p>
<p>就算你想尝试通过使用 <tt class="docutils literal">$ kill <span class="pre">-9</span></tt> 的方式来杀死僵尸进程也没有用，它们仍然能够活下来。你可以自己试试看。</p>
<p>僵尸进程究竟是什么呢？为什么我们的服务器会创建它们？ 僵尸进程是指一个已经终止了的进程，但是它的父进程并没有等待它，也没有收到它的终止状态。 当一个子进程在它的父进程之前退出时，内核会把这个子进程转换为僵尸进程，同时存储该进程的一下信息方便它的父进程之后来恢复它。 存储的信息通常包括进程 ID,进程终止状态，以及进程的资源使用情况。 好的，因此僵尸进程服务一个特殊的目的，但是如果你的服务器没有照顾好这些僵尸进程的话，你的系统将会变得拥堵不堪。 让我们来看看这是怎么发生的。 首先，停止你正在运行的服务器，然后在一个新的终端窗口总，使用 <tt class="docutils literal">ulimit</tt> 命令设置 max user processes 为 400（确保设置的 open files 足够高，也可以设置为 500）：</p>
<pre class="literal-block">$ ulimit -u 400$ ulimit -n 500
</pre>
<p>在你刚才输入 <tt class="docutils literal">$ ulimit <span class="pre">-u</span> 400</tt> 命令的窗口启动 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3d.py">webserver3d.py</a> 服务器：</p>
<pre class="literal-block">$ python webserver3d.py
</pre>
<p>在另一个新的终端窗口中，启动 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/client3.py">client3.py</a> 并告诉它同时创建 500 个连接：</p>
<pre class="literal-block">$ python client3.py --max-clients=500
</pre>
<p>很快，你的服务器就会崩溃并抛出 <tt class="docutils literal">OSError: Resource temporarily unavailable</tt> 的异常信息， 当它尝试创建一个新的子进程，但是却没法创建成功，因为它已经超出了允许创建的最大子进程数量。 下面是我机子上关于异常信息的截图：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc3_resource_unavailable.png" alt="lsbaws_part3_conc3_resource_unavailable.png" /></p>
<p>如你所见，如果你的长久运行的服务器不好好照看好僵尸进程的话，它们就会导致出现问题。 我将会简短的讨论一下服务器应该如何处理僵尸进程问题。</p>
<p>让我们来回顾一下你目前已经了解到的知识点：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_checkpoint.png" alt="lsbaws_part3_checkpoint.png" /></p>
<ul class="simple">
<li>如果你没有关闭描述符副本，客户端将不会退出，因为客户端连接还没有被关闭。</li>
<li>如果你没有关闭描述符副本，你那长时间运行的服务器最终将耗尽所有可用的文件描述符（<tt class="docutils literal">max open files</tt>）。</li>
<li>当你 fork 一个子进程然后退出，同时父进程没有等待( <tt class="docutils literal">wait</tt> )子进程完成退出操作，父进程就收集不到子进程的退出状态，子进程最终就会变成一个僵尸进程。</li>
<li>僵尸是需要吃东西的。我咱们这里，它们吃内存。如果不管这些僵尸进程的话，你的服务器将最终耗尽所有可用的进程（<tt class="docutils literal">max user processes</tt>）</li>
<li>你无法 <tt class="docutils literal">kill</tt> 一个僵尸进程，你需要等(<tt class="docutils literal">wait</tt> )它完成退出操作。</li>
</ul>
<p>那么，你应该如何处理僵尸进程呢？ 你需要修改你的服务器代码 <tt class="docutils literal">wait</tt> 等待所有的僵尸进程直到得到它们的退出状态。 你可以通过修改你的服务器去调用一个 <a class="reference external" href="https://docs.python.org/2.7/library/os.html#os.wait">wait</a> 系统调用的方式来达到这个目的。 不幸的是，理想跟现实是有差距的，因为如果你调用 <tt class="docutils literal">wait</tt> 然后又没有已经退出的子进程的话，调用 <tt class="docutils literal">wait</tt> 将阻塞你的服务器， 这就阻止你的服务器处理新的客户端连接请求。 难道就没有其他选项了吗？有的，一种解决办法就是联合使用<tt class="docutils literal">signal handler</tt> 和 <tt class="docutils literal">wait</tt> 系统调用。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc4_signaling.png" alt="lsbaws_part3_conc4_signaling.png" /></p>
<p>下面展示了是它如何工作。当一个子进程退出的时候，内核发送了一个 <tt class="docutils literal">SIGCHLD</tt> 信号。 父进程可以设置一个用于异步接收 <tt class="docutils literal">SIGCHLD</tt>事件的信号处理器，并且这个处理可以 wait 子进程以便收集它的终止状态， 这样就可以阻止僵尸进程的发生了。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part_conc4_sigchld_async.png" alt="lsbaws_part_conc4_sigchld_async.png" /></p>
<p>随便说一句，一个异步事件意味着父进程事先并不知道那个事件会发生。</p>
<p>修改你的服务器代码，设置一个 <tt class="docutils literal">SIGCHLD</tt> 时间处理器，在这个时间处理器中 wait 子进程终止。 可用的 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3e.py">webserver3e.py</a> 代码如下：</p>
<div class="highlight">
<pre><span class="c1">###########################################################################</span>
<span class="c1"># Concurrent server - webserver3e.py                                      #</span>
<span class="c1">#                                                                         #</span>
<span class="c1"># Tested with Python 2.7.9 &amp; Python 3.4 on Ubuntu 14.04 &amp; Mac OS X        #</span>
<span class="c1">###########################################################################</span>
<span class="kn">import</span> <span class="nn">os</span>
<span class="kn">import</span> <span class="nn">signal</span>
<span class="kn">import</span> <span class="nn">socket</span>
<span class="kn">import</span> <span class="nn">time</span>

<span class="n">SERVER_ADDRESS</span> <span class="o">=</span> <span class="p">(</span><span class="n">HOST</span><span class="p">,</span> <span class="n">PORT</span><span class="p">)</span> <span class="o">=</span> <span class="s1">''</span><span class="p">,</span> <span class="mi">8888</span>
<span class="n">REQUEST_QUEUE_SIZE</span> <span class="o">=</span> <span class="mi">5</span>


<span class="k">def</span> <span class="nf">grim_reaper</span><span class="p">(</span><span class="n">signum</span><span class="p">,</span> <span class="n">frame</span><span class="p">):</span>
    <span class="n">pid</span><span class="p">,</span> <span class="n">status</span> <span class="o">=</span> <span class="n">os</span><span class="o">.</span><span class="n">wait</span><span class="p">()</span>
    <span class="k">print</span><span class="p">(</span>
        <span class="s1">'Child {pid} terminated with status {status}'</span>
        <span class="s1">'</span><span class="se">\n</span><span class="s1">'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">pid</span><span class="o">=</span><span class="n">pid</span><span class="p">,</span> <span class="n">status</span><span class="o">=</span><span class="n">status</span><span class="p">)</span>
    <span class="p">)</span>


<span class="k">def</span> <span class="nf">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">):</span>
    <span class="n">request</span> <span class="o">=</span> <span class="n">client_connection</span><span class="o">.</span><span class="n">recv</span><span class="p">(</span><span class="mi">1024</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">decode</span><span class="p">())</span>
    <span class="n">http_response</span> <span class="o">=</span> <span class="n">b</span><span class="s2">"""</span><span class="se">\</span>
<span class="s2">HTTP/1.1 200 OK</span>

<span class="s2">Hello, World!</span>
<span class="s2">"""</span>
    <span class="n">client_connection</span><span class="o">.</span><span class="n">sendall</span><span class="p">(</span><span class="n">http_response</span><span class="p">)</span>
    <span class="c1"># sleep to allow the parent to loop over to 'accept' and block there</span>
    <span class="n">time</span><span class="o">.</span><span class="n">sleep</span><span class="p">(</span><span class="mi">3</span><span class="p">)</span>


<span class="k">def</span> <span class="nf">serve_forever</span><span class="p">():</span>
    <span class="n">listen_socket</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">socket</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">AF_INET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SOCK_STREAM</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">setsockopt</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">SOL_SOCKET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SO_REUSEADDR</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">bind</span><span class="p">(</span><span class="n">SERVER_ADDRESS</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">listen</span><span class="p">(</span><span class="n">REQUEST_QUEUE_SIZE</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="s1">'Serving HTTP on port {port} ...'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">port</span><span class="o">=</span><span class="n">PORT</span><span class="p">))</span>

    <span class="n">signal</span><span class="o">.</span><span class="n">signal</span><span class="p">(</span><span class="n">signal</span><span class="o">.</span><span class="n">SIGCHLD</span><span class="p">,</span> <span class="n">grim_reaper</span><span class="p">)</span>

    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="n">client_connection</span><span class="p">,</span> <span class="n">client_address</span> <span class="o">=</span> <span class="n">listen_socket</span><span class="o">.</span><span class="n">accept</span><span class="p">()</span>
        <span class="n">pid</span> <span class="o">=</span> <span class="n">os</span><span class="o">.</span><span class="n">fork</span><span class="p">()</span>
        <span class="k">if</span> <span class="n">pid</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>  <span class="c1"># child</span>
            <span class="n">listen_socket</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>  <span class="c1"># close child copy</span>
            <span class="n">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">)</span>
            <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>
            <span class="n">os</span><span class="o">.</span><span class="n">_exit</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>  <span class="c1"># parent</span>
            <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s1">'__main__'</span><span class="p">:</span>
    <span class="n">serve_forever</span><span class="p">()</span>
</pre>
</div>
<p>启动服务器：:</p>
<pre class="literal-block">$ python webserver3e.py
</pre>
<p>使用你的老朋友 <tt class="docutils literal">curl</tt> 向修改后的并发服务器发送一个请求:</p>
<pre class="literal-block">$ curl http://localhost:8888/hello
</pre>
<p>看一下服务器：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc4_eintr.png" alt="lsbaws_part3_conc4_eintr.png" /></p>
<p>发生了什么？ accept 调用是吧了，并报了个<tt class="docutils literal">EINTR</tt> 错误。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc4_eintr_error.png" alt="lsbaws_part3_conc4_eintr_error.png" /></p>
<p>当子进程退出，触发 <tt class="docutils literal">SIGCHLD</tt> 事件时，激活了时间信号处理器，然后父进程阻塞在了 accept 调用这个地方，</p>
<p>当信号处理器处理完成以后， accept 系统调用也跟着中断了：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc4_eintr_accept.png" alt="lsbaws_part3_conc4_eintr_accept.png" /></p>
<p>别担心，这是个非常简单问题很容易解决。你要做到的就是重新开始 accept 系统调用。</p>
<p>下面是修改版本的服务器 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3f.py">webserver3f.py</a> ，这个版本解决了这个问题：</p>
<div class="highlight">
<pre><span class="c1">###########################################################################</span>
<span class="c1"># Concurrent server - webserver3f.py                                      #</span>
<span class="c1">#                                                                         #</span>
<span class="c1"># Tested with Python 2.7.9 &amp; Python 3.4 on Ubuntu 14.04 &amp; Mac OS X        #</span>
<span class="c1">###########################################################################</span>
<span class="kn">import</span> <span class="nn">errno</span>
<span class="kn">import</span> <span class="nn">os</span>
<span class="kn">import</span> <span class="nn">signal</span>
<span class="kn">import</span> <span class="nn">socket</span>

<span class="n">SERVER_ADDRESS</span> <span class="o">=</span> <span class="p">(</span><span class="n">HOST</span><span class="p">,</span> <span class="n">PORT</span><span class="p">)</span> <span class="o">=</span> <span class="s1">''</span><span class="p">,</span> <span class="mi">8888</span>
<span class="n">REQUEST_QUEUE_SIZE</span> <span class="o">=</span> <span class="mi">1024</span>


<span class="k">def</span> <span class="nf">grim_reaper</span><span class="p">(</span><span class="n">signum</span><span class="p">,</span> <span class="n">frame</span><span class="p">):</span>
    <span class="n">pid</span><span class="p">,</span> <span class="n">status</span> <span class="o">=</span> <span class="n">os</span><span class="o">.</span><span class="n">wait</span><span class="p">()</span>


<span class="k">def</span> <span class="nf">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">):</span>
    <span class="n">request</span> <span class="o">=</span> <span class="n">client_connection</span><span class="o">.</span><span class="n">recv</span><span class="p">(</span><span class="mi">1024</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">decode</span><span class="p">())</span>
    <span class="n">http_response</span> <span class="o">=</span> <span class="n">b</span><span class="s2">"""</span><span class="se">\</span>
<span class="s2">HTTP/1.1 200 OK</span>

<span class="s2">Hello, World!</span>
<span class="s2">"""</span>
    <span class="n">client_connection</span><span class="o">.</span><span class="n">sendall</span><span class="p">(</span><span class="n">http_response</span><span class="p">)</span>


<span class="k">def</span> <span class="nf">serve_forever</span><span class="p">():</span>
    <span class="n">listen_socket</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">socket</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">AF_INET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SOCK_STREAM</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">setsockopt</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">SOL_SOCKET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SO_REUSEADDR</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">bind</span><span class="p">(</span><span class="n">SERVER_ADDRESS</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">listen</span><span class="p">(</span><span class="n">REQUEST_QUEUE_SIZE</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="s1">'Serving HTTP on port {port} ...'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">port</span><span class="o">=</span><span class="n">PORT</span><span class="p">))</span>

    <span class="n">signal</span><span class="o">.</span><span class="n">signal</span><span class="p">(</span><span class="n">signal</span><span class="o">.</span><span class="n">SIGCHLD</span><span class="p">,</span> <span class="n">grim_reaper</span><span class="p">)</span>

    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="n">client_connection</span><span class="p">,</span> <span class="n">client_address</span> <span class="o">=</span> <span class="n">listen_socket</span><span class="o">.</span><span class="n">accept</span><span class="p">()</span>
        <span class="k">except</span> <span class="ne">IOError</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="n">code</span><span class="p">,</span> <span class="n">msg</span> <span class="o">=</span> <span class="n">e</span><span class="o">.</span><span class="n">args</span>
            <span class="c1"># restart 'accept' if it was interrupted</span>
            <span class="k">if</span> <span class="n">code</span> <span class="o">==</span> <span class="n">errno</span><span class="o">.</span><span class="n">EINTR</span><span class="p">:</span>
                <span class="k">continue</span>
            <span class="k">else</span><span class="p">:</span>
                <span class="k">raise</span>

        <span class="n">pid</span> <span class="o">=</span> <span class="n">os</span><span class="o">.</span><span class="n">fork</span><span class="p">()</span>
        <span class="k">if</span> <span class="n">pid</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>  <span class="c1"># child</span>
            <span class="n">listen_socket</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>  <span class="c1"># close child copy</span>
            <span class="n">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">)</span>
            <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>
            <span class="n">os</span><span class="o">.</span><span class="n">_exit</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>  <span class="c1"># parent</span>
            <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>  <span class="c1"># close parent copy and loop over</span>


<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s1">'__main__'</span><span class="p">:</span>
    <span class="n">serve_forever</span><span class="p">()</span>
</pre>
</div>
<p>启动更新后的 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3f.py">webserver3f.py</a> :</p>
<pre class="literal-block">$ python webserver3f.py
</pre>
<p>使用 <tt class="docutils literal">curl</tt> 向修改过的并发服务器发送一个请求：</p>
<blockquote><p>$ curl <a class="reference external" href="http://localhost:8888/hello">http://localhost:8888/hello</a></p></blockquote>
<p>看到了没？ 不再有 EINTR 异常了。现在，验证一下，不再有僵尸了，而且你的 <tt class="docutils literal">SIGCHLD</tt>事件处理器通过 wait 调用来处理子进程的终止事件。为了验证这个，只需要运行 ps 命令，然后你可以看一下应该不再有状态为 <strong>Z+</strong>的 Python 僵尸进程了（不再有 <strong>&lt;default&gt;</strong> 进程）。太棒了！不再有僵尸进程的日子安全感终于有保障了。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_checkpoint.png" alt="lsbaws_part3_checkpoint.png" /></p>
<ul class="simple">
<li>如果你 fork 了一个子进程，但是却没有 wait 它，它就会变成一个僵尸进程。</li>
<li>使用 <tt class="docutils literal">SIGCHLD</tt> 事件处理器来异步 wait 终止的子进程以便收集它的终止状态，</li>
<li>当使用事件处理器的时候，你需要考虑到系统可能会中断，这样的话你就需求为这个场景做些准备。</li>
</ul>
<p>好了，目前来看一起都很棒。没问题，是吧？嗯，确实如此。 再试试 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3f.py">webserver3f.py</a>，不过这次不是使用 <tt class="docutils literal">curl</tt> 制造一个请求，而是使用 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/client3.py">client3.py</a> 创建 128 个同时发生的连接：:</p>
<pre class="literal-block">$ python client3.py --max-clients 128
</pre>
<p>现在再一次执行 <tt class="docutils literal">ps</tt> 命令</p>
<pre class="literal-block">$ ps auxw | grep -i python | grep -v grep
</pre>
<p>看呐，哦，天哪，僵尸进程又回来了！</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc5_zombies_again.png" alt="lsbaws_part3_conc5_zombies_again.png" /></p>
<p>这次又怎么了呢？当你运行了 128 个同步的客户端时，同时就建立了 128 条连接，服务器上处理请求和退出的子进程大多数都在同一时触发大量的 <tt class="docutils literal">SIGCHLD</tt> 信号被发送给父进程。 问题就是这些信号并不是按队列进行处理的，这样的话，你的服务器进程就会错过一些信号，这会遗留一下无人照看的僵尸进程。</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc5_signals_not_queued.png" alt="lsbaws_part3_conc5_signals_not_queued.png" /></p>
<p>解决这个问题的方法是设置一个 <tt class="docutils literal">SIGCHLD</tt> 事件处理器， 不使用 <tt class="docutils literal">wait</tt> 而是调用 <a class="reference external" href="https://docs.python.org/2.7/library/os.html#os.waitpid">waitpid</a> 系统调用并在循环中使用 <tt class="docutils literal">WNOHANG</tt> 选项，确保所有终止的子进程都被照顾到了。 下面是修改后的服务器代码， <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3g.py">webserver3g.py</a> :</p>
<div class="highlight">
<pre><span class="c1">###########################################################################</span>
<span class="c1"># Concurrent server - webserver3g.py                                      #</span>
<span class="c1">#                                                                         #</span>
<span class="c1"># Tested with Python 2.7.9 &amp; Python 3.4 on Ubuntu 14.04 &amp; Mac OS X        #</span>
<span class="c1">###########################################################################</span>
<span class="kn">import</span> <span class="nn">errno</span>
<span class="kn">import</span> <span class="nn">os</span>
<span class="kn">import</span> <span class="nn">signal</span>
<span class="kn">import</span> <span class="nn">socket</span>

<span class="n">SERVER_ADDRESS</span> <span class="o">=</span> <span class="p">(</span><span class="n">HOST</span><span class="p">,</span> <span class="n">PORT</span><span class="p">)</span> <span class="o">=</span> <span class="s1">''</span><span class="p">,</span> <span class="mi">8888</span>
<span class="n">REQUEST_QUEUE_SIZE</span> <span class="o">=</span> <span class="mi">1024</span>


<span class="k">def</span> <span class="nf">grim_reaper</span><span class="p">(</span><span class="n">signum</span><span class="p">,</span> <span class="n">frame</span><span class="p">):</span>
    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="n">pid</span><span class="p">,</span> <span class="n">status</span> <span class="o">=</span> <span class="n">os</span><span class="o">.</span><span class="n">waitpid</span><span class="p">(</span>
                <span class="o">-</span><span class="mi">1</span><span class="p">,</span>          <span class="c1"># Wait for any child process</span>
                 <span class="n">os</span><span class="o">.</span><span class="n">WNOHANG</span>  <span class="c1"># Do not block and return EWOULDBLOCK error</span>
            <span class="p">)</span>
        <span class="k">except</span> <span class="ne">OSError</span><span class="p">:</span>
            <span class="k">return</span>

        <span class="k">if</span> <span class="n">pid</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>  <span class="c1"># no more zombies</span>
            <span class="k">return</span>


<span class="k">def</span> <span class="nf">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">):</span>
    <span class="n">request</span> <span class="o">=</span> <span class="n">client_connection</span><span class="o">.</span><span class="n">recv</span><span class="p">(</span><span class="mi">1024</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="n">request</span><span class="o">.</span><span class="n">decode</span><span class="p">())</span>
    <span class="n">http_response</span> <span class="o">=</span> <span class="n">b</span><span class="s2">"""</span><span class="se">\</span>
<span class="s2">HTTP/1.1 200 OK</span>

<span class="s2">Hello, World!</span>
<span class="s2">"""</span>
    <span class="n">client_connection</span><span class="o">.</span><span class="n">sendall</span><span class="p">(</span><span class="n">http_response</span><span class="p">)</span>


<span class="k">def</span> <span class="nf">serve_forever</span><span class="p">():</span>
    <span class="n">listen_socket</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">socket</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">AF_INET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SOCK_STREAM</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">setsockopt</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">SOL_SOCKET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SO_REUSEADDR</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">bind</span><span class="p">(</span><span class="n">SERVER_ADDRESS</span><span class="p">)</span>
    <span class="n">listen_socket</span><span class="o">.</span><span class="n">listen</span><span class="p">(</span><span class="n">REQUEST_QUEUE_SIZE</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="s1">'Serving HTTP on port {port} ...'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">port</span><span class="o">=</span><span class="n">PORT</span><span class="p">))</span>

    <span class="n">signal</span><span class="o">.</span><span class="n">signal</span><span class="p">(</span><span class="n">signal</span><span class="o">.</span><span class="n">SIGCHLD</span><span class="p">,</span> <span class="n">grim_reaper</span><span class="p">)</span>

    <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="n">client_connection</span><span class="p">,</span> <span class="n">client_address</span> <span class="o">=</span> <span class="n">listen_socket</span><span class="o">.</span><span class="n">accept</span><span class="p">()</span>
        <span class="k">except</span> <span class="ne">IOError</span> <span class="k">as</span> <span class="n">e</span><span class="p">:</span>
            <span class="n">code</span><span class="p">,</span> <span class="n">msg</span> <span class="o">=</span> <span class="n">e</span><span class="o">.</span><span class="n">args</span>
            <span class="c1"># restart 'accept' if it was interrupted</span>
            <span class="k">if</span> <span class="n">code</span> <span class="o">==</span> <span class="n">errno</span><span class="o">.</span><span class="n">EINTR</span><span class="p">:</span>
                <span class="k">continue</span>
            <span class="k">else</span><span class="p">:</span>
                <span class="k">raise</span>

        <span class="n">pid</span> <span class="o">=</span> <span class="n">os</span><span class="o">.</span><span class="n">fork</span><span class="p">()</span>
        <span class="k">if</span> <span class="n">pid</span> <span class="o">==</span> <span class="mi">0</span><span class="p">:</span>  <span class="c1"># child</span>
            <span class="n">listen_socket</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>  <span class="c1"># close child copy</span>
            <span class="n">handle_request</span><span class="p">(</span><span class="n">client_connection</span><span class="p">)</span>
            <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>
            <span class="n">os</span><span class="o">.</span><span class="n">_exit</span><span class="p">(</span><span class="mi">0</span><span class="p">)</span>
        <span class="k">else</span><span class="p">:</span>  <span class="c1"># parent</span>
            <span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>  <span class="c1"># close parent copy and loop over</span>

<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s1">'__main__'</span><span class="p">:</span>
    <span class="n">serve_forever</span><span class="p">()</span>
</pre>
</div>
<p>启动服务器:</p>
<pre class="literal-block">$ python webserver3g.py
</pre>
<p>使用测试客户端 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/client3.py">client3.py</a>:</p>
<pre class="literal-block">$ python client3.py --max-clients 128
</pre>
<p>现在验证一下已经不再有僵尸进程了。耶！没有僵尸进程的生活是如此的美好 🙂</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part3/lsbaws_part3_conc5_no_zombies.png" alt="lsbaws_part3_conc5_no_zombies.png" /></p>
<p>恭喜！这可真是个漫长的旅程，但是我希望你能喜欢它。 现在你已经有了你自己的简单的并发服务器，这些代码可以作为你将来开发产品级 Web 服务器的基础。</p>
<p>我要给你留一个练习，那就是将的 <a class="reference external" href="https://reidholmes.github.io/2016/10/15/lets-build-a-web-server-part-2/">第二部分</a>的 WSGI 服务器更新为并发服务器。 你可以从 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part3/webserver3h.py">这里</a> 得到最终的修改版。但是，只在你完成你的版本后才能查看我的代码。 你已经具备了完成这项工作的所必需的所有信息了。所以，放手去做吧：）</p>
<p>下一步是什么？正如 Josh Billings 说过的，</p>
<blockquote><p>“像一张邮票一样 —— 坚持一件事情直到你到达终点。”</p></blockquote>
<p>开始征服基础知识。质疑你已经知道的。同时总是深入挖掘。</p>
<blockquote><p>“如果你只学习方法，你将束缚于你的方法。但是，如果你学会了原理，你就可以发明你自己的方法。” —— Ralph Waldo Emerson</p></blockquote>
<p>&nbsp;</p>
<ul class="simple">
<li><a class="reference external" href="http://book.douban.com/subject/4859464/">UNIX网络编程 卷1：套接字联网API（第3版）</a></li>
<li><a class="reference external" href="http://book.douban.com/subject/25900403/">UNIX环境高级编程（第3版）</a></li>
<li><a class="reference external" href="http://book.douban.com/subject/25809330/">Linux/UNIX系统编程手册</a></li>
<li><a class="reference external" href="http://book.douban.com/subject/3571433/">TCP/IP详解 卷1：协议（第2版）</a></li>
<li><a class="reference external" href="http://book.douban.com/subject/3666232/">The Little Book of SEMAPHORES (2nd Edition): The Ins and Outs of Concurrency Control and Common Mistakes</a></li>
</ul>
</div>