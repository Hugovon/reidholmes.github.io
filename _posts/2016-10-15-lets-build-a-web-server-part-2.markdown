---
published: true
layout:     post
title:      "构建一个 WEB 服务器（二）"
date:       2016-10-15 18:38:22
categories:
---



<p>本文译自：<a class="reference external" href="http://ruslanspivak.com/lsbaws-part2/">http://ruslanspivak.com/lsbaws-part2/</a></p>
<p>还记得吗？在 <a class="reference external" href="https://reidholmes.github.io/2016/10/15/lets-build-a-web-server-part-1/">第一部分</a> 我向你问了一个问题：”如何在你新鲜出炉的 Web 服务器上不做任何修改的就运行 Django 应用，Flask 应用， Pyramid 应用?“ 往下读就可以找到答案。</p>
<p>在过去，当你选择的 Python Web 框架会限制你所能选择的 Web 服务器, 如果那个框架和服务器被设计的可以一起工作的话，那就皆大欢喜了：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_before_wsgi.png" alt="Server Framework Fit" /></p>
<p>但是，当你尝试把一个服务器和一个框架一起使用的时候可能会（可能你已经）面临它们被设计为不兼容的情况：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_after_wsgi.png" alt="Server Framework Clash" /></p>
<p>一般来说你必须使用能够一起工作的组件而不仅仅是你想使用的组件。</p>
<p>那么，你如何确保你能够在你的 Web 服务器上运行多个 Web 框架，并且不需要修改 Web 服务器或 Web 框架的现有的代码呢？ 解决这个问题的答案就是 Python Web Server Gateway Interface (或简称 WSGI , 读作 “wizgy”)</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_wsgi_idea.png" alt="WSGI Interface" /></p>
<p><a class="reference external" href="https://www.python.org/dev/peps/pep-0333/">WSGI</a> 允许开发者自由选择 Web 框架和 Web 服务器。现在你可以任意混搭不同的 Web 服务器和 Web 框架，并选择一个你需要的合适的组合。 比如，你可以 用 <a class="reference external" href="http://gunicorn.org/">Gunicorn</a> or<a class="reference external" href="http://uwsgi-docs.readthedocs.org/">Nginx/uWSGI</a> or <a class="reference external" href="http://waitress.readthedocs.org/">Waitress</a> 运行 <a class="reference external" href="https://www.djangoproject.com/">Django</a>, <a class="reference external" href="http://flask.pocoo.org/">Flask</a>, 或 <a class="reference external" href="http://trypyramid.com/">Pyramid</a> . 真正的随意混搭，感谢那些服务器和框架对 WSGI 的支持：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_wsgi_interop.png" alt="Mix &amp; Match" /></p>
<p>因此，<a class="reference external" href="https://www.python.org/dev/peps/pep-0333/">WSGI</a> 就是我在 <a class="reference external" href="https://reidholmes.github.io/2016/10/15/lets-build-a-web-server-part-1/">第一部分</a> 向你问的并在文章开头重复过的问题的答案。 你的 Web 服务器必须实现 WSGI 接口的服务器端部分， 所有的现代 Python Web 框架都已经实现了 WSGI 接口的框架端部分， 这部分允许你不需要修改你的服务器代码去适应某个特定的框架就可以使用这些框架。</p>
<p>现在你已经知道了被 Web 服务器和 Web 框架所支持的 WSGI 允许你选择适合你的组合， 它同样也对服务器和框架的开发者有益，因为他们可以专注于标准中他们各自的区域，不会出现因为越界而踩到对方的脚趾。 其他语言也有类似的接口：例如，Java 有 <a class="reference external" href="http://en.wikipedia.org/wiki/Java_servlet">Servlet API</a>，Ruby 有 <a class="reference external" href="http://en.wikipedia.org/wiki/Rack_%28web_server_interface%29">Rack</a>.</p>
<p>一切都很棒，但是我猜你会说”Show me the code!“，好吧，一起来看看下面这个非常简约的 WSGI 服务器实现吧：</p>
<div class="highlight">
<pre><span class="c1"># Tested with Python 2.7.9, Linux &amp; Mac OS X</span>
<span class="kn">import</span> <span class="nn">socket</span>
<span class="kn">import</span> <span class="nn">StringIO</span>
<span class="kn">import</span> <span class="nn">sys</span>


<span class="k">class</span> <span class="nc">WSGIServer</span><span class="p">(</span><span class="nb">object</span><span class="p">):</span>

    <span class="n">address_family</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">AF_INET</span>
    <span class="n">socket_type</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">SOCK_STREAM</span>
    <span class="n">request_queue_size</span> <span class="o">=</span> <span class="mi">1</span>

    <span class="k">def</span> <span class="nf">__init__</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">server_address</span><span class="p">):</span>
        <span class="c1"># Create a listening socket</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">listen_socket</span> <span class="o">=</span> <span class="n">listen_socket</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">socket</span><span class="p">(</span>
            <span class="bp">self</span><span class="o">.</span><span class="n">address_family</span><span class="p">,</span>
            <span class="bp">self</span><span class="o">.</span><span class="n">socket_type</span>
        <span class="p">)</span>
        <span class="c1"># Allow to reuse the same address</span>
        <span class="n">listen_socket</span><span class="o">.</span><span class="n">setsockopt</span><span class="p">(</span><span class="n">socket</span><span class="o">.</span><span class="n">SOL_SOCKET</span><span class="p">,</span> <span class="n">socket</span><span class="o">.</span><span class="n">SO_REUSEADDR</span><span class="p">,</span> <span class="mi">1</span><span class="p">)</span>
        <span class="c1"># Bind</span>
        <span class="n">listen_socket</span><span class="o">.</span><span class="n">bind</span><span class="p">(</span><span class="n">server_address</span><span class="p">)</span>
        <span class="c1"># Activate</span>
        <span class="n">listen_socket</span><span class="o">.</span><span class="n">listen</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">request_queue_size</span><span class="p">)</span>
        <span class="c1"># Get server host name and port</span>
        <span class="n">host</span><span class="p">,</span> <span class="n">port</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">listen_socket</span><span class="o">.</span><span class="n">getsockname</span><span class="p">()[:</span><span class="mi">2</span><span class="p">]</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">server_name</span> <span class="o">=</span> <span class="n">socket</span><span class="o">.</span><span class="n">getfqdn</span><span class="p">(</span><span class="n">host</span><span class="p">)</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">server_port</span> <span class="o">=</span> <span class="n">port</span>
        <span class="c1"># Return headers set by Web framework/Web application</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">headers_set</span> <span class="o">=</span> <span class="p">[]</span>

    <span class="k">def</span> <span class="nf">set_app</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">application</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">application</span> <span class="o">=</span> <span class="n">application</span>

    <span class="k">def</span> <span class="nf">serve_forever</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">listen_socket</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">listen_socket</span>
        <span class="k">while</span> <span class="bp">True</span><span class="p">:</span>
            <span class="c1"># New client connection</span>
            <span class="bp">self</span><span class="o">.</span><span class="n">client_connection</span><span class="p">,</span> <span class="n">client_address</span> <span class="o">=</span> <span class="n">listen_socket</span><span class="o">.</span><span class="n">accept</span><span class="p">()</span>
            <span class="c1"># Handle one request and close the client connection. Then</span>
            <span class="c1"># loop over to wait for another client connection</span>
            <span class="bp">self</span><span class="o">.</span><span class="n">handle_one_request</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">handle_one_request</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">request_data</span> <span class="o">=</span> <span class="n">request_data</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">client_connection</span><span class="o">.</span><span class="n">recv</span><span class="p">(</span><span class="mi">1024</span><span class="p">)</span>
        <span class="c1"># Print formatted request data a la 'curl -v'</span>
        <span class="k">print</span><span class="p">(</span><span class="s1">''</span><span class="o">.</span><span class="n">join</span><span class="p">(</span>
            <span class="s1">'&lt; {line}</span><span class="se">\n</span><span class="s1">'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">line</span><span class="o">=</span><span class="n">line</span><span class="p">)</span>
            <span class="k">for</span> <span class="n">line</span> <span class="ow">in</span> <span class="n">request_data</span><span class="o">.</span><span class="n">splitlines</span><span class="p">()</span>
        <span class="p">))</span>

        <span class="bp">self</span><span class="o">.</span><span class="n">parse_request</span><span class="p">(</span><span class="n">request_data</span><span class="p">)</span>

        <span class="c1"># Construct environment dictionary using request data</span>
        <span class="n">env</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">get_environ</span><span class="p">()</span>

        <span class="c1"># It's time to call our application callable and get</span>
        <span class="c1"># back a result that will become HTTP response body</span>
        <span class="n">result</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">application</span><span class="p">(</span><span class="n">env</span><span class="p">,</span> <span class="bp">self</span><span class="o">.</span><span class="n">start_response</span><span class="p">)</span>

        <span class="c1"># Construct a response and send it back to the client</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">finish_response</span><span class="p">(</span><span class="n">result</span><span class="p">)</span>

    <span class="k">def</span> <span class="nf">parse_request</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">text</span><span class="p">):</span>
        <span class="n">request_line</span> <span class="o">=</span> <span class="n">text</span><span class="o">.</span><span class="n">splitlines</span><span class="p">()[</span><span class="mi">0</span><span class="p">]</span>
        <span class="n">request_line</span> <span class="o">=</span> <span class="n">request_line</span><span class="o">.</span><span class="n">rstrip</span><span class="p">(</span><span class="s1">'</span><span class="se">\r\n</span><span class="s1">'</span><span class="p">)</span>
        <span class="c1"># Break down the request line into components</span>
        <span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">request_method</span><span class="p">,</span>  <span class="c1"># GET</span>
         <span class="bp">self</span><span class="o">.</span><span class="n">path</span><span class="p">,</span>            <span class="c1"># /hello</span>
         <span class="bp">self</span><span class="o">.</span><span class="n">request_version</span>  <span class="c1"># HTTP/1.1</span>
         <span class="p">)</span> <span class="o">=</span> <span class="n">request_line</span><span class="o">.</span><span class="n">split</span><span class="p">()</span>

    <span class="k">def</span> <span class="nf">get_environ</span><span class="p">(</span><span class="bp">self</span><span class="p">):</span>
        <span class="n">env</span> <span class="o">=</span> <span class="p">{}</span>
        <span class="c1"># The following code snippet does not follow PEP8 conventions</span>
        <span class="c1"># but it's formatted the way it is for demonstration purposes</span>
        <span class="c1"># to emphasize the required variables and their values</span>
        <span class="c1">#</span>
        <span class="c1"># Required WSGI variables</span>
        <span class="n">env</span><span class="p">[</span><span class="s1">'wsgi.version'</span><span class="p">]</span>      <span class="o">=</span> <span class="p">(</span><span class="mi">1</span><span class="p">,</span> <span class="mi">0</span><span class="p">)</span>
        <span class="n">env</span><span class="p">[</span><span class="s1">'wsgi.url_scheme'</span><span class="p">]</span>   <span class="o">=</span> <span class="s1">'http'</span>
        <span class="n">env</span><span class="p">[</span><span class="s1">'wsgi.input'</span><span class="p">]</span>        <span class="o">=</span> <span class="n">StringIO</span><span class="o">.</span><span class="n">StringIO</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">request_data</span><span class="p">)</span>
        <span class="n">env</span><span class="p">[</span><span class="s1">'wsgi.errors'</span><span class="p">]</span>       <span class="o">=</span> <span class="n">sys</span><span class="o">.</span><span class="n">stderr</span>
        <span class="n">env</span><span class="p">[</span><span class="s1">'wsgi.multithread'</span><span class="p">]</span>  <span class="o">=</span> <span class="bp">False</span>
        <span class="n">env</span><span class="p">[</span><span class="s1">'wsgi.multiprocess'</span><span class="p">]</span> <span class="o">=</span> <span class="bp">False</span>
        <span class="n">env</span><span class="p">[</span><span class="s1">'wsgi.run_once'</span><span class="p">]</span>     <span class="o">=</span> <span class="bp">False</span>
        <span class="c1"># Required CGI variables</span>
        <span class="n">env</span><span class="p">[</span><span class="s1">'REQUEST_METHOD'</span><span class="p">]</span>    <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">request_method</span>    <span class="c1"># GET</span>
        <span class="n">env</span><span class="p">[</span><span class="s1">'PATH_INFO'</span><span class="p">]</span>         <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">path</span>              <span class="c1"># /hello</span>
        <span class="n">env</span><span class="p">[</span><span class="s1">'SERVER_NAME'</span><span class="p">]</span>       <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">server_name</span>       <span class="c1"># localhost</span>
        <span class="n">env</span><span class="p">[</span><span class="s1">'SERVER_PORT'</span><span class="p">]</span>       <span class="o">=</span> <span class="nb">str</span><span class="p">(</span><span class="bp">self</span><span class="o">.</span><span class="n">server_port</span><span class="p">)</span>  <span class="c1"># 8888</span>
        <span class="k">return</span> <span class="n">env</span>

    <span class="k">def</span> <span class="nf">start_response</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">status</span><span class="p">,</span> <span class="n">response_headers</span><span class="p">,</span> <span class="n">exc_info</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="c1"># Add necessary server headers</span>
        <span class="n">server_headers</span> <span class="o">=</span> <span class="p">[</span>
            <span class="p">(</span><span class="s1">'Date'</span><span class="p">,</span> <span class="s1">'Tue, 31 Mar 2015 12:54:48 GMT'</span><span class="p">),</span>
            <span class="p">(</span><span class="s1">'Server'</span><span class="p">,</span> <span class="s1">'WSGIServer 0.2'</span><span class="p">),</span>
        <span class="p">]</span>
        <span class="bp">self</span><span class="o">.</span><span class="n">headers_set</span> <span class="o">=</span> <span class="p">[</span><span class="n">status</span><span class="p">,</span> <span class="n">response_headers</span> <span class="o">+</span> <span class="n">server_headers</span><span class="p">]</span>
        <span class="c1"># To adhere to WSGI specification the start_response must return</span>
        <span class="c1"># a 'write' callable. We simplicity's sake we'll ignore that detail</span>
        <span class="c1"># for now.</span>
        <span class="c1"># return self.finish_response</span>

    <span class="k">def</span> <span class="nf">finish_response</span><span class="p">(</span><span class="bp">self</span><span class="p">,</span> <span class="n">result</span><span class="p">):</span>
        <span class="k">try</span><span class="p">:</span>
            <span class="n">status</span><span class="p">,</span> <span class="n">response_headers</span> <span class="o">=</span> <span class="bp">self</span><span class="o">.</span><span class="n">headers_set</span>
            <span class="n">response</span> <span class="o">=</span> <span class="s1">'HTTP/1.1 {status}</span><span class="se">\r\n</span><span class="s1">'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">status</span><span class="o">=</span><span class="n">status</span><span class="p">)</span>
            <span class="k">for</span> <span class="n">header</span> <span class="ow">in</span> <span class="n">response_headers</span><span class="p">:</span>
                <span class="n">response</span> <span class="o">+=</span> <span class="s1">'{0}: {1}</span><span class="se">\r\n</span><span class="s1">'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="o">*</span><span class="n">header</span><span class="p">)</span>
            <span class="n">response</span> <span class="o">+=</span> <span class="s1">'</span><span class="se">\r\n</span><span class="s1">'</span>
            <span class="k">for</span> <span class="n">data</span> <span class="ow">in</span> <span class="n">result</span><span class="p">:</span>
                <span class="n">response</span> <span class="o">+=</span> <span class="n">data</span>
            <span class="c1"># Print formatted response data a la 'curl -v'</span>
            <span class="k">print</span><span class="p">(</span><span class="s1">''</span><span class="o">.</span><span class="n">join</span><span class="p">(</span>
                <span class="s1">'&gt; {line}</span><span class="se">\n</span><span class="s1">'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">line</span><span class="o">=</span><span class="n">line</span><span class="p">)</span>
                <span class="k">for</span> <span class="n">line</span> <span class="ow">in</span> <span class="n">response</span><span class="o">.</span><span class="n">splitlines</span><span class="p">()</span>
            <span class="p">))</span>
            <span class="bp">self</span><span class="o">.</span><span class="n">client_connection</span><span class="o">.</span><span class="n">sendall</span><span class="p">(</span><span class="n">response</span><span class="p">)</span>
        <span class="k">finally</span><span class="p">:</span>
            <span class="bp">self</span><span class="o">.</span><span class="n">client_connection</span><span class="o">.</span><span class="n">close</span><span class="p">()</span>


<span class="n">SERVER_ADDRESS</span> <span class="o">=</span> <span class="p">(</span><span class="n">HOST</span><span class="p">,</span> <span class="n">PORT</span><span class="p">)</span> <span class="o">=</span> <span class="s1">''</span><span class="p">,</span> <span class="mi">8888</span>


<span class="k">def</span> <span class="nf">make_server</span><span class="p">(</span><span class="n">server_address</span><span class="p">,</span> <span class="n">application</span><span class="p">):</span>
    <span class="n">server</span> <span class="o">=</span> <span class="n">WSGIServer</span><span class="p">(</span><span class="n">server_address</span><span class="p">)</span>
    <span class="n">server</span><span class="o">.</span><span class="n">set_app</span><span class="p">(</span><span class="n">application</span><span class="p">)</span>
    <span class="k">return</span> <span class="n">server</span>


<span class="k">if</span> <span class="n">__name__</span> <span class="o">==</span> <span class="s1">'__main__'</span><span class="p">:</span>
    <span class="k">if</span> <span class="nb">len</span><span class="p">(</span><span class="n">sys</span><span class="o">.</span><span class="n">argv</span><span class="p">)</span> <span class="o">&lt;</span> <span class="mi">2</span><span class="p">:</span>
        <span class="n">sys</span><span class="o">.</span><span class="n">exit</span><span class="p">(</span><span class="s1">'Provide a WSGI application object as module:callable'</span><span class="p">)</span>
    <span class="n">app_path</span> <span class="o">=</span> <span class="n">sys</span><span class="o">.</span><span class="n">argv</span><span class="p">[</span><span class="mi">1</span><span class="p">]</span>
    <span class="n">module</span><span class="p">,</span> <span class="n">application</span> <span class="o">=</span> <span class="n">app_path</span><span class="o">.</span><span class="n">split</span><span class="p">(</span><span class="s1">':'</span><span class="p">)</span>
    <span class="n">module</span> <span class="o">=</span> <span class="nb">__import__</span><span class="p">(</span><span class="n">module</span><span class="p">)</span>
    <span class="n">application</span> <span class="o">=</span> <span class="nb">getattr</span><span class="p">(</span><span class="n">module</span><span class="p">,</span> <span class="n">application</span><span class="p">)</span>
    <span class="n">httpd</span> <span class="o">=</span> <span class="n">make_server</span><span class="p">(</span><span class="n">SERVER_ADDRESS</span><span class="p">,</span> <span class="n">application</span><span class="p">)</span>
    <span class="k">print</span><span class="p">(</span><span class="s1">'WSGIServer: Serving HTTP on port {port} ...</span><span class="se">\n</span><span class="s1">'</span><span class="o">.</span><span class="n">format</span><span class="p">(</span><span class="n">port</span><span class="o">=</span><span class="n">PORT</span><span class="p">))</span>
    <span class="n">httpd</span><span class="o">.</span><span class="n">serve_forever</span><span class="p">()</span>
</pre>
</div>
<p>上面的代码比 <a class="reference external" href="http://mozillazg.com/2015/06/let-us-build-a-web-server-part-1-zh-cn.html">第一部分</a> 的服务器代码更长，但是，为了让你能够理解而不至于陷入细节的泥潭中，它已经足够小了（只有不到 150 行）。 上面的服务器代码同样也能做更多的工作——它能运行用你上面所见的 Web 框架（<a class="reference external" href="http://trypyramid.com/">Pyramid</a>, <a class="reference external" href="http://flask.pocoo.org/">Flask</a>, <a class="reference external" href="https://www.djangoproject.com/">Django</a>, 或其他的 Python WSGI 框架）所写的基础 Web 应用，</p>
<p>不信？动手试一下吧。把上面的代码保存为<tt class="docutils literal">webserver2.py</tt> 或者直接从 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part2/webserver2.py">GitHub</a> 上下载下来。如果你不带任何参数就运行这个程序的话，它会向你抱怨，然后退出。</p>
<pre class="literal-block">$ python webserver2.py
Provide a WSGI application object as module:callable
</pre>
<p>它真的非常想要服务你的 Web 应用，这是个非常有趣的开始。 为了能够运行这个服务器你只需要安装 Python 就可以了。 但是，为了运行用 <a class="reference external" href="http://trypyramid.com/">Pyramid</a>, <a class="reference external" href="http://flask.pocoo.org/">Flask</a>, 或 <a class="reference external" href="https://www.djangoproject.com/">Django</a> 开发的应用，你需要首先安装这些框架。 让我们来安装这三个框架吧。 我喜欢使用 <a class="reference external" href="https://virtualenv.pypa.io/">virtualenv</a>. 只需按照下面的步骤去创建并激活一个虚拟环境，然后就可以安装这三个框架了。</p>
<pre class="literal-block">$ [sudo] pip install virtualenv
$ mkdir ~/envs
$ virtualenv ~/envs/lsbaws/
$ cd ~/envs/lsbaws/
$ ls
bin  include  lib
$ source bin/activate
(lsbaws) $ pip install pyramid
(lsbaws) $ pip install flask
(lsbaws) $ pip install django
</pre>
<p>到这一步的时候你需要创建一个 Web 应用。让我们先用 <a class="reference external" href="http://trypyramid.com/">Pyramid</a> 开始吧。把下面的代码保存为 <tt class="docutils literal">pyramidapp.py</tt> 并放到你之前所保存的<tt class="docutils literal">webserver2.py</tt> 文件或直接从 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part2/pyramidapp.py">GitHub</a> 所下载的文件所在目录（即：把 <tt class="docutils literal">pyramidapp.py</tt> 放在<tt class="docutils literal">webserver2.py</tt> 所在目录）：</p>
<div class="highlight">
<pre><span class="kn">from</span> <span class="nn">pyramid.config</span> <span class="kn">import</span> <span class="n">Configurator</span>
<span class="kn">from</span> <span class="nn">pyramid.response</span> <span class="kn">import</span> <span class="n">Response</span>


<span class="k">def</span> <span class="nf">hello_world</span><span class="p">(</span><span class="n">request</span><span class="p">):</span>
    <span class="k">return</span> <span class="n">Response</span><span class="p">(</span>
        <span class="s1">'Hello world from Pyramid!</span><span class="se">\n</span><span class="s1">'</span><span class="p">,</span>
        <span class="n">content_type</span><span class="o">=</span><span class="s1">'text/plain'</span><span class="p">,</span>
    <span class="p">)</span>

<span class="n">config</span> <span class="o">=</span> <span class="n">Configurator</span><span class="p">()</span>
<span class="n">config</span><span class="o">.</span><span class="n">add_route</span><span class="p">(</span><span class="s1">'hello'</span><span class="p">,</span> <span class="s1">'/hello'</span><span class="p">)</span>
<span class="n">config</span><span class="o">.</span><span class="n">add_view</span><span class="p">(</span><span class="n">hello_world</span><span class="p">,</span> <span class="n">route_name</span><span class="o">=</span><span class="s1">'hello'</span><span class="p">)</span>
<span class="n">app</span> <span class="o">=</span> <span class="n">config</span><span class="o">.</span><span class="n">make_wsgi_app</span><span class="p">()</span>
</pre>
</div>
<p>现在，你可以准备用你自己的 Web 服务器来服务你的 Pyramid 应用了：</p>
<pre class="literal-block">(lsbaws) $ python webserver2.py pyramidapp:app
WSGIServer: Serving HTTP on port 8888 ...
</pre>
<p>你只需告诉你的服务器从 python 模块<tt class="docutils literal">pyramidapp</tt> 中载入一个可调用的 <tt class="docutils literal">app</tt> 对象，你的服务器现在已经准备好 接收请求并把它们转发给你的 Pyramid 应用了。 这个应用目前只处理了一个路由：<tt class="docutils literal">/hello</tt> 路由。 在你的浏览器中输入 <a class="reference external" href="http://localhost:8888/hello">http://localhost:8888/hello</a> 地址，然后按下回车键，注意返回的结果：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_browser_pyramid.png" alt="Pyramid" /></p>
<p>你也可以在命令行中使用 <tt class="docutils literal">curl</tt> 命令来测试这个服务器：</p>
<pre class="literal-block">$ curl -v http://localhost:8888/hello
...
</pre>
<p>检查服务器以及 <tt class="docutils literal">curl</tt> 打印到标准输出的内容。</p>
<p>现在轮到 <a class="reference external" href="http://flask.pocoo.org/">Flask</a> 了。让我们按照相同的步骤来操作。</p>
<div class="highlight">
<pre><span class="kn">from</span> <span class="nn">flask</span> <span class="kn">import</span> <span class="n">Flask</span>
<span class="kn">from</span> <span class="nn">flask</span> <span class="kn">import</span> <span class="n">Response</span>
<span class="n">flask_app</span> <span class="o">=</span> <span class="n">Flask</span><span class="p">(</span><span class="s1">'flaskapp'</span><span class="p">)</span>


<span class="nd">@flask_app.route</span><span class="p">(</span><span class="s1">'/hello'</span><span class="p">)</span>
<span class="k">def</span> <span class="nf">hello_world</span><span class="p">():</span>
    <span class="k">return</span> <span class="n">Response</span><span class="p">(</span>
        <span class="s1">'Hello world from Flask!</span><span class="se">\n</span><span class="s1">'</span><span class="p">,</span>
        <span class="n">mimetype</span><span class="o">=</span><span class="s1">'text/plain'</span>
    <span class="p">)</span>

<span class="n">app</span> <span class="o">=</span> <span class="n">flask_app</span><span class="o">.</span><span class="n">wsgi_app</span>
</pre>
</div>
<p>把上面的代码保存为 <tt class="docutils literal">flaskapp.py</tt> 或从<a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part2/flaskapp.py">GitHub</a> 上下载，然后用以下方式运行服务器:</p>
<pre class="literal-block">(lsbaws) $ python webserver2.py flaskapp:app
WSGIServer: Serving HTTP on port 8888 ...
</pre>
<p>现在在你的浏览器中输入<a class="reference external" href="http://localhost:8888/hello">http://localhost:8888/hello</a> 然后按下回车键：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_browser_flask.png" alt="Flask" /></p>
<p>再一次，尝试 <tt class="docutils literal">curl</tt> 命令，然后看一下服务器返回的由这个 Flask 应用所生成的信息：</p>
<pre class="literal-block">$ curl -v http://localhost:8888/hello
...
</pre>
<p>这个服务器能处理 <a class="reference external" href="https://www.djangoproject.com/">Django</a> 应用吗啊？试一下就知道了！ 这次涉及的东西有点复杂，我建议你克隆这个 <a class="reference external" href="https://github.com/rspivak/lsbaws/">仓库</a> 然后使用 GitHub 仓库 中的 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part2/djangoapp.py">djangoapp.py</a> 文件。 下面的源码主要是添加 Django <tt class="docutils literal">helloworld</tt> 项目（预先使用 Django 的 <tt class="docutils literal"><span class="pre">django-admin.py</span> startproject</tt> 命令）到当前 Python 路径 然后导入项目中的 WSGI 应用。</p>
<div class="highlight">
<pre><span class="kn">import</span> <span class="nn">sys</span>
<span class="n">sys</span><span class="o">.</span><span class="n">path</span><span class="o">.</span><span class="n">insert</span><span class="p">(</span><span class="mi">0</span><span class="p">,</span> <span class="s1">'./helloworld'</span><span class="p">)</span>
<span class="kn">from</span> <span class="nn">helloworld</span> <span class="kn">import</span> <span class="n">wsgi</span>


<span class="n">app</span> <span class="o">=</span> <span class="n">wsgi</span><span class="o">.</span><span class="n">application</span>
</pre>
</div>
<p>把上面的代码保存为 <tt class="docutils literal">djangoapp.py</tt> 然后用你的 Web 服务器运行这个 Django 应用：</p>
<pre class="literal-block">(lsbaws) $ python webserver2.py djangoapp:app
WSGIServer: Serving HTTP on port 8888 ...
</pre>
<p>输入如下地址并回车：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_browser_django.png" alt="Django" /></p>
<p>正如你之前做过的那几次一样，你也可以在命令行中进行测试。 确认这个 Django 应用处理了你这一次的请求：</p>
<pre class="literal-block">$ curl -v http://localhost:8888/hello
...
</pre>
<p>你试过了吗？你有确认过这个服务器可以与这三个框架一起工作吗？ 如果还没有的话，一定要试一下。 阅读很重要，但是这个系列讲的是关于重新构建，这意味着你需要手动进行这些尝试。 快去试试吧。别担心，我会等你的。 我是认真的，你必须去尝试，最好能够亲自一个字一个字的敲下所有的字符， 并确保它能达到预期的效果。</p>
<p>好了，你已经熟悉 WSGI 的威力了：它允许你混搭你的 Web 服务器和 Web 框架。 WSGI 规定了 Python Web 服务器和 Python Web 框架之间的一些接口。 它非常的简单，不管是在服务器还是框架端都非常容易实现。 下面的片段展示了服务器和框架端的接口：</p>
<div class="highlight">
<pre><span class="k">def</span> <span class="nf">run_application</span><span class="p">(</span><span class="n">application</span><span class="p">):</span>
    <span class="sd">"""Server code."""</span>
    <span class="c1"># This is where an application/framework stores</span>
    <span class="c1"># an HTTP status and HTTP response headers for the server</span>
    <span class="c1"># to transmit to the client</span>
    <span class="n">headers_set</span> <span class="o">=</span> <span class="p">[]</span>
    <span class="c1"># Environment dictionary with WSGI/CGI variables</span>
    <span class="n">environ</span> <span class="o">=</span> <span class="p">{}</span>

    <span class="k">def</span> <span class="nf">start_response</span><span class="p">(</span><span class="n">status</span><span class="p">,</span> <span class="n">response_headers</span><span class="p">,</span> <span class="n">exc_info</span><span class="o">=</span><span class="bp">None</span><span class="p">):</span>
        <span class="n">headers_set</span><span class="p">[:]</span> <span class="o">=</span> <span class="p">[</span><span class="n">status</span><span class="p">,</span> <span class="n">response_headers</span><span class="p">]</span>

    <span class="c1"># Server invokes the ‘application' callable and gets back the</span>
    <span class="c1"># response body</span>
    <span class="n">result</span> <span class="o">=</span> <span class="n">application</span><span class="p">(</span><span class="n">environ</span><span class="p">,</span> <span class="n">start_response</span><span class="p">)</span>
    <span class="c1"># Server builds an HTTP response and transmits it to the client</span>
    <span class="err">…</span>

<span class="k">def</span> <span class="nf">app</span><span class="p">(</span><span class="n">environ</span><span class="p">,</span> <span class="n">start_response</span><span class="p">):</span>
    <span class="sd">"""A barebones WSGI app."""</span>
    <span class="n">start_response</span><span class="p">(</span><span class="s1">'200 OK'</span><span class="p">,</span> <span class="p">[(</span><span class="s1">'Content-Type'</span><span class="p">,</span> <span class="s1">'text/plain'</span><span class="p">)])</span>
    <span class="k">return</span> <span class="p">[</span><span class="s1">'Hello world!'</span><span class="p">]</span>

<span class="n">run_application</span><span class="p">(</span><span class="n">app</span><span class="p">)</span>
</pre>
</div>
<p>它的工作原理是这样的：</p>
<ol class="arabic simple">
<li>框架提供了一个 <tt class="docutils literal">application</tt> 可调用对象（WSGI 规范没有规定它应该如何被实现）</li>
<li>每当收到来自 HTTP 客户端的请求的时候，服务器就调用这个 <tt class="docutils literal">application</tt> 可调用对象。 它把一个包含 WSGI/CGI 变量的字典 <tt class="docutils literal">environ</tt> 和一个<tt class="docutils literal">start_response</tt> 可调用对象作为参数传递给了 <tt class="docutils literal">application</tt> 可调用对象。</li>
<li>框架/应用生成一个 HTTP 状态信息和 HTTP 响应头信息，并把它们传递给了<tt class="docutils literal">start_response</tt> 可调用对象， 让服务器把它们存起来。框架/应用也返回了一个响应 body 信息。</li>
<li>服务器把状态信息，响应头信息以及响应 body 信息合并为一个 HTTP 响应，然后把它传输给客户端（这一步不是规范的一部分， 但是它是流程中的下一个逻辑步骤，为了清晰可见我把它列在了这里）</li>
</ol>
<p>下面是这个接口的可视化图表：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_wsgi_interface.png" alt="WSGI Interface Visual" /></p>
<p>到目前位置，你已经见过了 <a class="reference external" href="http://trypyramid.com/">Pyramid</a>, <a class="reference external" href="http://flask.pocoo.org/">Flask</a>以及 <a class="reference external" href="https://www.djangoproject.com/">Django</a> Web 应用，你也见过了实现 WSGI 规范的服务器端代码。 你也见过不用任何框架所实现的极简 WSGI 应用的代码片段。</p>
<p>事实是，当你用这些框架中某个开发一个 Web 应用的时候，你是在高层面进行工作， 并没有直接与 WSGI 打交到，但是我知道非常好奇框架端的 WSGI 接口实现，也是因为你正在阅读这篇文章。 那么，让我们来创建一个不使用 <a class="reference external" href="http://trypyramid.com/">Pyramid</a>, <a class="reference external" href="http://flask.pocoo.org/">Flask</a>, <a class="reference external" href="https://www.djangoproject.com/">Django</a> 的微型 WSGI Web 应用/Web 框架， 并用你的服务器来运行它：</p>
<div class="highlight">
<pre><span class="k">def</span> <span class="nf">app</span><span class="p">(</span><span class="n">environ</span><span class="p">,</span> <span class="n">start_response</span><span class="p">):</span>
    <span class="sd">"""A barebones WSGI application.</span>

<span class="sd">    This is a starting point for your own Web framework :)</span>
<span class="sd">    """</span>
    <span class="n">status</span> <span class="o">=</span> <span class="s1">'200 OK'</span>
    <span class="n">response_headers</span> <span class="o">=</span> <span class="p">[(</span><span class="s1">'Content-Type'</span><span class="p">,</span> <span class="s1">'text/plain'</span><span class="p">)]</span>
    <span class="n">start_response</span><span class="p">(</span><span class="n">status</span><span class="p">,</span> <span class="n">response_headers</span><span class="p">)</span>
    <span class="k">return</span> <span class="p">[</span><span class="s1">'Hello world from a simple WSGI application!</span><span class="se">\n</span><span class="s1">'</span><span class="p">]</span>
</pre>
</div>
<p>再一次的，把上面的代码保存为 <tt class="docutils literal">wsgiapp.py</tt>或直接从 <a class="reference external" href="https://github.com/rspivak/lsbaws/blob/master/part2/wsgiapp.py">GitHub</a> 上下载它，然后用你的 Web 服务器像下面这样运行这个应用：</p>
<pre class="literal-block">(lsbaws) $ python webserver2.py wsgiapp:app
WSGIServer: Serving HTTP on port 8888 ...
</pre>
<p>输入如下地址并按下回车键。你应该会看到这样的结果：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_browser_simple_wsgi_app.png" alt="Simple WSGI Application" /></p>
<p>在学习如何创建一个 Web 服务器的同时，你刚刚又写了一个你自己的微型 WSGI WEB 框架！ 真是意外之喜！</p>
<p>现在，让我们回到服务器都给客户端传输了什么东西。 下面是当你使用 HTTP 客户端调用你的 Pyramind 应用时，服务器生成的 HTTP 响应：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_http_response.png" alt="HTTP Response Part 1" /></p>
<p>这个响应有一些你在 <a class="reference external" href="http://mozillazg.com/2015/06/let-us-build-a-web-server-part-1-zh-cn.html">第一部分</a> 看到过的东西，但是它也有一些新东西。比如说，它有四个你之前还没见过的 <a class="reference external" href="http://en.wikipedia.org/wiki/List_of_HTTP_header_fields">HTTP headers</a>：<tt class="docutils literal"><span class="pre">Content-Type</span></tt> , <tt class="docutils literal"><span class="pre">Content-Length</span></tt> , <tt class="docutils literal">Date</tt> 以及<tt class="docutils literal">Server</tt> . 这些包含在响应里的头信息是一个 Web 服务器应该要生成的信息。 虽然它们中没有一个是严格要求必须提供的。 这些头信息的目的是传输关于 HTTP 请求/响应的附加信息。</p>
<p>现在你已经了解了关于 WSGI 接口的更详细的信息了，下面是同一个 HTTP 响应部分是如何产生的更详细的信息：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_http_response_explanation.png" alt="HTTP Response Part 2" /></p>
<p>我还没有说过任何有关 <tt class="docutils literal">environ</tt> 字典相关的信息，但是，基本上就是它是一个 Python 字典，它必须包含某些由 WSGI 规范所规定的 WSGI 和 CGI 变量。 解析完请求信息后，服务器从 HTTP 请求中得到这个字典所需的一些值。 这个字典看起来像下面这样：</p>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_environ.png" alt="Environ Python Dictionary" /></p>
<p>Web 框架使用来自这个字典里的信息来决定那个 view 可以被用来服务，基于获得的路由，请求方法等信息, 决定可以从哪里读取请求的 body 信息以及哪里可以用来写入错误信息，如果有的话。</p>
<p>到目前为止，你已经创建了你自己的 WSGI Web 服务器，你也用不同的 Web 框架编写过 Web 应用了。同时，你也顺便创建过极其简陋的 Web 应用/Web 框架。 真是一个操蛋的旅程。让我们来重述一下为了服务一个针对 WSGI 应用的请求信息，你的 WSGI Web 框架需要做的事情：</p>
<ol class="arabic simple">
<li>首先，服务器启动并载入一个由你的 Web 框架/应用所定义的 <tt class="docutils literal">application</tt> 可调用对象</li>
<li>然后，服务器读取一个请求</li>
<li>然后，服务器解析这个请求</li>
<li>然后，服务器用这个请求数据构建了一个 <tt class="docutils literal">environ</tt> 字典</li>
<li>然后，服务器以 <tt class="docutils literal">environ</tt> 字典和一个<tt class="docutils literal">start_response</tt> 可调用对象作为参数来调用 <tt class="docutils literal">application</tt> 对象，并获得一个返回的响应 body 。</li>
<li>然后，服务器用通过调用 <tt class="docutils literal">application</tt>对象获得的 body 数据以及通过<tt class="docutils literal">start_reponse</tt> 可调用对象设置的状态信息和响应头信息一起构建了一个 HTTP 响应。</li>
<li>最后，服务器把 HTTP 响应传输回客户端</li>
</ol>
<p><img src="https://mozillazg.com/static/images/lsbaws-part2/lsbaws_part2_server_summary.png" alt="Server Summary" /></p>
<p>就这些了。你现在有了一个可以工作的 WSGI 服务器，它能够服务那些用 WSGI 兼容的 Web 框架（比如：<a class="reference external" href="https://www.djangoproject.com/">Django</a>, <a class="reference external" href="http://flask.pocoo.org/">Flask</a>, <a class="reference external" href="http://trypyramid.com/">Pyramid</a>或者是你自己开发的 WSGI 框架) 开发的基础的 Web 应用。最棒的是不需要修改任何的服务器代码就可以与多个 Web 框架一起使用。目前看起来还不赖嘛。</p>
<p>在你离开前，这里有另一个问题需要你思考，”如何让你的服务器能够在同一时刻处理多个请求？“</p>