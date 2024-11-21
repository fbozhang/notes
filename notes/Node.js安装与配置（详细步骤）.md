<div id="content_views" class="markdown_views prism-atom-one-dark">
                    <svg xmlns="http://www.w3.org/2000/svg" style="display: none;">
                        <path stroke-linecap="round" d="M5,0 0,2.5 5,5z" id="raphael-marker-block" style="-webkit-tap-highlight-color: rgba(0, 0, 0, 0);"></path>
                    </svg>
                    <p><font size="5"><strong>前言</strong></font></p> 
<p>本篇博文记录了Node.js安装与<a href="https://so.csdn.net/so/search?q=%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E9%85%8D%E7%BD%AE&amp;spm=1001.2101.3001.7020" target="_blank" class="hl hl-1" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.7020&quot;,&quot;dest&quot;:&quot;https://so.csdn.net/so/search?q=%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F%E9%85%8D%E7%BD%AE&amp;spm=1001.2101.3001.7020&quot;,&quot;extra&quot;:&quot;{\&quot;searchword\&quot;:\&quot;环境变量配置\&quot;}&quot;}" data-tit="环境变量配置" data-pretit="环境变量配置">环境变量配置</a>的详细步骤，旨在为将来再次配置Node.js时提供指导方法。</p> 
<p>另外：Node.js版本请根据自身系统选择，安装位置、全局模块存放位置和环境变量应根据自身实际情况进行更改。<br> </p>
<div class="toc">
 <h3><a name="t0"></a>Node.js安装与配置</h3>
 <ul><li><a href="#Nodejs_6" target="_self">一、安装Node.js</a></li><li><ul><li><a href="#1_7" target="_self">1.下载</a></li><li><a href="#2_11" target="_self">2.安装</a></li><li><a href="#3_16" target="_self">3.添加环境变量</a></li></ul>
  </li><li><a href="#_21" target="_self">二、验证是否安装成功</a></li><li><a href="#_35" target="_self">三、修改模块下载位置</a></li><li><ul><li><a href="#1npm_38" target="_self">1.查看npm默认存放位置</a></li><li><a href="#2_nodejs__node_global__node_cache__50" target="_self">2.在 nodejs 安装目录下，创建 “node_global” 和 “node_cache” 两个文件夹</a></li><li><a href="#3_54" target="_self">3.修改默认文件夹</a></li><li><a href="#4_70" target="_self">4.测试默认位置是否更改成功</a></li></ul>
  </li><li><a href="#_95" target="_self">四、设置淘宝镜像</a></li><li><ul><li><a href="#1npmregistryregistry_96" target="_self">1.将npm默认的registry修改为淘宝registry</a></li><li><a href="#2cnpm_117" target="_self">2.全局安装基于淘宝源的cnpm</a></li></ul>
  </li><li><a href="#_134" target="_self">五、总结大家的问题</a></li><li><ul><li><a href="#1_139" target="_self">1.勾选文件夹权限后，下载模块时仍然报错。</a></li><li><a href="#2npm_install_express_globalexpress_145" target="_self">2.使用`npm install express --global`安装express时提示：</a></li><li><a href="#3cnpm_vcnpm_153" target="_self">3.`cnpm -v`只能在cnpm目录下才可以正常显示版本号。</a></li><li><a href="#4cnpm_cnpm_v_cnpm__156" target="_self">4.安装完cnpm时 运行`cnpm -v `出现'cnpm' 不是内部或外部命令，也不是可运行的程序。</a></li></ul>
 </li></ul>
</div>
<p></p> 
<h1><a name="t1"></a><a id="Nodejs_6"></a>一、安装Node.js</h1> 
<h2><a name="t2"></a><a id="1_7"></a>1.下载</h2> 
<p><strong><a href="http://nodejs.cn/download/">Node.js官网下载</a></strong><br> 根据自身系统下载对应的安装包（我这里为Windows11 64位，故选择下载第一个安装包）<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/ebe850304a33fa7f66cd51f192e7b78f.png" alt="在这里插入图片描述"></p> 
<h2><a name="t3"></a><a id="2_11"></a>2.安装</h2> 
<p>双击安装包，点击Next，勾选使用许可协议，点击Next，选择安装位置（可根据个人情况更换路径，我这里选择安装在E:\devTools\<a href="https://so.csdn.net/so/search?q=nodejs&amp;spm=1001.2101.3001.7020" target="_blank" class="hl hl-1" data-report-view="{&quot;spm&quot;:&quot;1001.2101.3001.7020&quot;,&quot;dest&quot;:&quot;https://so.csdn.net/so/search?q=nodejs&amp;spm=1001.2101.3001.7020&quot;,&quot;extra&quot;:&quot;{\&quot;searchword\&quot;:\&quot;nodejs\&quot;}&quot;}" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.7020&quot;,&quot;dest&quot;:&quot;https://so.csdn.net/so/search?q=nodejs&amp;spm=1001.2101.3001.7020&quot;,&quot;extra&quot;:&quot;{\&quot;searchword\&quot;:\&quot;nodejs\&quot;}&quot;}" data-tit="nodejs" data-pretit="nodejs">nodejs</a>）<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/634880837f4d91d4cac5dd7c574129c4.png" alt="在这里插入图片描述"><br> 继续点击Next，点击Next，点击Install，点击Finish完成安装。</p> 
<h2><a name="t4"></a><a id="3_16"></a>3.添加环境变量</h2> 
<p>3.1 进入环境变量，编辑【系统变量】下的变量【Path】<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/8cf84f89b51330f2e6fd9b3f14e5a7df.png" alt="选择Path变量">3.2 添加Node.js的安装路径（此处为E:\devTools\nodejs\）<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/6227ff688372bcdafcb8ddcfc95de9ec.png" alt="写入Node.js安装路径"></p> 
<h1><a name="t5"></a><a id="_21"></a>二、验证是否安装成功</h1> 
<p>进入<a href="https://so.csdn.net/so/search?q=cmd%E5%91%BD%E4%BB%A4&amp;spm=1001.2101.3001.7020" target="_blank" class="hl hl-1" data-report-view="{&quot;spm&quot;:&quot;1001.2101.3001.7020&quot;,&quot;dest&quot;:&quot;https://so.csdn.net/so/search?q=cmd%E5%91%BD%E4%BB%A4&amp;spm=1001.2101.3001.7020&quot;,&quot;extra&quot;:&quot;{\&quot;searchword\&quot;:\&quot;cmd命令\&quot;}&quot;}" data-report-click="{&quot;spm&quot;:&quot;1001.2101.3001.7020&quot;,&quot;dest&quot;:&quot;https://so.csdn.net/so/search?q=cmd%E5%91%BD%E4%BB%A4&amp;spm=1001.2101.3001.7020&quot;,&quot;extra&quot;:&quot;{\&quot;searchword\&quot;:\&quot;cmd命令\&quot;}&quot;}" data-tit="cmd命令" data-pretit="cmd命令">cmd命令</a>行窗口，输入node -v查看nodejs版本</p> 
<pre data-index="0" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">node <span class="token operator">-</span>v
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p>输入npm -v查看npm版本</p> 
<pre data-index="1" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm <span class="token operator">-</span>v
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p>如下图所示，即为安装成功：<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/181c4eb156ee98d4d42ab6a0806305ae.png" alt="验证是否安装成功"></p> 
<h1><a name="t6"></a><a id="_35"></a>三、修改模块下载位置</h1> 
<p><mark><em><strong>此步骤修改以后npm全局下载模块的保存位置，可根据自身情况选择是否更改。</strong></em></mark></p> 
<h2><a name="t7"></a><a id="1npm_38"></a>1.查看npm默认存放位置</h2> 
<p>使用npm get prefix查看npm全局模块的存放路径</p> 
<pre data-index="2" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm get prefix
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p>使用npm get cache查看npm缓存默认存放路径</p> 
<pre data-index="3" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm get cache
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p><img src="https://i-blog.csdnimg.cn/blog_migrate/23cb8b564293d53a01ec8d752f397f04.png" alt="在这里插入图片描述"><br> 如上图所示，npm 全局模块存放位置以及cache的存放位置，默认是在 C 盘 “C:\Users\用户\AppData” 下。</p> 
<h2><a name="t8"></a><a id="2_nodejs__node_global__node_cache__50"></a>2.在 nodejs 安装目录下，创建 “node_global” 和 “node_cache” 两个文件夹</h2> 
<p><img src="https://i-blog.csdnimg.cn/blog_migrate/c5818023a165441670c047b9369e3dd3.png" alt="在这里插入图片描述"></p> 
<h2><a name="t9"></a><a id="3_54"></a>3.修改默认文件夹</h2> 
<p>设置全局模块的安装路径到 “node_global” 文件夹，</p> 
<pre data-index="4" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm config <span class="token function">set</span> prefix <span class="token string">"E:\devTools\nodejs\node_global"</span>
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p>设置缓存到 “node_cache” 文件夹</p> 
<pre data-index="5" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm config <span class="token function">set</span> cache <span class="token string">"E:\devTools\nodejs\node_cache"</span>
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p>如下图所示：<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/ed63a97f8e655a41c70508dc0d8368aa.png" alt="在这里插入图片描述"></p> 
<p><font color="red" size="4"><strong>注意：</strong></font>由于 node 全局模块大多数都是可以通过命令行访问的，还要把【node_global】的路径“E:\devTools\nodejs\node_global”加入到【系统变量 】下的【PATH】 变量中，方便直接使用命令行运行，如下图所示：<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/3336ada33d85fcf84e0180a5d30b524b.png" alt="在这里插入图片描述"></p> 
<h2><a name="t10"></a><a id="4_70"></a>4.测试默认位置是否更改成功</h2> 
<p>经过上面的步骤，nodejs下载的模块就会自动下载到我们自定义的目录，接下来我们测试一下是否更改成功。输入下面的命令：</p> 
<pre data-index="6" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm install express <span class="token operator">-</span>g
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p>或者</p> 
<pre data-index="7" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm install express <span class="token operator">--</span>global
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p><font color="red" size="4"><strong>注意：</strong></font>“-g”等同于“–global”，“-g” 是全局安装，不加“-g”就是默认下载到当前目录。“-g” 表示安装到之前设置的【node_global】目录下，同时nodejs会自动地在node_global文件夹下创建【node_modules】子文件夹， 即自动下载到“E:\devTools\nodejs\node_global\node_modules” 路径下。<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/a2a247262bd5f80301fb23ff1dcdf675.png" alt="在这里插入图片描述">如上图所示，下载express模块成功，然后在文件管理器中查看是否保存到上面自定义的路径下。<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/7e803d68addc4173d9c36f8ed42a9f48.png" alt="在这里插入图片描述">可以看到，express模块已经成功地下载到【E:\devTools\nodejs\node_global\node_modules】下。</p> 
<p><font color="red" size="5"><strong>注意：</strong></font>若执行命令npm install express -g出现如下报错：<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/f190d0f825082c11d73e0add0fb7fb44.png" alt="在这里插入图片描述"><br> 是由于对文件夹操作的权限不够，右击Nodejs文件夹-&gt;属性-&gt;安全，点击编辑，将所有权限都✔即可。<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/70c1d87b4bbeab6ab0cfc297bdea993f.png" alt="在这里插入图片描述"><br> <font color="red" size="4"><strong>※</strong></font>执行npm install express -g仍然出错的话继续将nodejs下【node_cache】、【node_global】、【node_modules】这三个文件夹的所有权限勾选，再次执行：</p> 
<pre data-index="8" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm install express <span class="token operator">-</span>g
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p>即可下载成功。</p> 
<h1><a name="t11"></a><a id="_95"></a>四、设置淘宝镜像</h1> 
<h2><a name="t12"></a><a id="1npmregistryregistry_96"></a>1.将npm默认的registry修改为淘宝registry</h2> 
<p>说明：npm 默认的 registry ,也就是下载 npm 包时会从国外的服务器下载，国内下载会很慢，一般更换为淘宝镜像：https://registry.npm.taobao.org。</p> 
<p>1.1 查看当前使用的镜像路径</p> 
<pre data-index="9" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm config get registry
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p><img src="https://i-blog.csdnimg.cn/blog_migrate/04228e7e6fa4f982bc52890afa5d9009.png" alt="在这里插入图片描述"><br> 1.2 更换npm为淘宝镜像</p> 
<pre data-index="10" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm config <span class="token function">set</span> registry https:<span class="token operator">/</span><span class="token operator">/</span>registry<span class="token punctuation">.</span>npm<span class="token punctuation">.</span>taobao<span class="token punctuation">.</span>org/
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p><img src="https://i-blog.csdnimg.cn/blog_migrate/1b5a2d9f147e2c554c237e94826578f1.png" alt="在这里插入图片描述"><br> 1.3 检查镜像是否配置成功<br> 再次执行npm config get registry，检查当前的镜像路径：</p> 
<pre data-index="11" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm config get registry
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p><img src="https://i-blog.csdnimg.cn/blog_migrate/fc8af7bbaecc8a785bc7f233e2e7ad49.png" alt="在这里插入图片描述"><br> 如上图所示，npm默认的registry已修改为淘宝registry。</p> 
<h2><a name="t13"></a><a id="2cnpm_117"></a>2.全局安装基于淘宝源的cnpm</h2> 
<p>说明：由于npm的服务器在海外，所以访问速度比较慢，访问不稳定 ，cnpm的服务器是由淘宝团队提供，服务器在国内，cnpm是npm镜像，一般会同步更新，相差在10分钟，所以cnpm在安装一些软件时候会比较有优势。但是cnpm一般只用于模块安装，在项目创建与卸载等相关操作时仍使用npm。</p> 
<p>2.1 全局安装基于淘宝源的cnpm</p> 
<pre data-index="12" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm install <span class="token operator">-</span>g cnpm <span class="token operator">--</span>registry=https:<span class="token operator">/</span><span class="token operator">/</span>registry<span class="token punctuation">.</span>npm<span class="token punctuation">.</span>taobao<span class="token punctuation">.</span>org
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p><img src="https://i-blog.csdnimg.cn/blog_migrate/3b9fb97d2a37d274211ae147969bb78a.png" alt="在这里插入图片描述">2.2 本地查看cnpm模块<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/a7ee9a3d6ae54636348b69b5eaa73bba.png" alt="在这里插入图片描述"><br> 2.3 执行命令查看cnpm是否安装成功</p> 
<pre data-index="13" class="prettyprint"><code class="prism language-powershell has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">cnpm <span class="token operator">-</span>v
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p>如下图所示，即代表cnpm配置成功。<br> <img src="https://i-blog.csdnimg.cn/blog_migrate/901c71fa4935edb41c4e88c119375dde.png" alt="在这里插入图片描述"></p> 
<h1><a name="t14"></a><a id="_134"></a>五、总结大家的问题</h1> 
<p>--------------------------- 分割线 2022.10.20 总结大家提的问题 ---------------------------<br> 非常感谢大家踊跃评论，可能部分评论的问题没有及时作出回答，在此道歉。<br> 下面是大家提出的一些问题并总结回答如下：</p> 
<h2><a name="t15"></a><a id="1_139"></a>1.勾选文件夹权限后，下载模块时仍然报错。</h2> 
<p>回答：<br> 将node.js的安装路径和下面的【node_cache】、【node_global】、【node_modules】几个子文件夹的权限都勾选上。</p> 
<p>如果执行npm install命令安装模块仍然报错，可以再根据报错信息中的<code>path</code>将文件夹的权限都勾选上。<img src="https://i-blog.csdnimg.cn/blog_migrate/f190d0f825082c11d73e0add0fb7fb44.png" alt="在这里插入图片描述"></p> 
<h2><a name="t16"></a><a id="2npm_install_express_globalexpress_145"></a>2.使用<code>npm install express --global</code>安装express时提示：</h2> 
<pre data-index="14" class="prettyprint"><code class="prism language-c has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm WARN config global `<span class="token operator">--</span>global`<span class="token punctuation">,</span> `<span class="token operator">--</span>local` are deprecated<span class="token punctuation">.</span> Use `<span class="token operator">--</span>location<span class="token operator">=</span>global` instead<span class="token punctuation">.</span>
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p>回答：<br> <strong>这仅是一条警告，不是报错</strong>，仍可以正常下载模块，可以忽略，意思是npm不建议使用–global（等价于-g）或者–local，应该使用<code>--location=global</code>替代。<br> 如果仍使用-g且不想让npm报warn，请自行百度查找解决办法。</p> 
<h2><a name="t17"></a><a id="3cnpm_vcnpm_153"></a>3.<code>cnpm -v</code>只能在cnpm目录下才可以正常显示版本号。</h2> 
<p>回答：<code>cnpm -v</code>只能在cnpm安装的目录才能运行，应是未使用-g或–global进行全局安装导致。</p> 
<h2><a name="t18"></a><a id="4cnpm_cnpm_v_cnpm__156"></a>4.安装完cnpm时 运行<code>cnpm -v </code>出现’cnpm’ 不是内部或外部命令，也不是可运行的程序。</h2> 
<p>回答：<br> ①首先确认是否使用的-g全局安装</p> 
<pre data-index="15" class="prettyprint"><code class="prism language-objectivec has-numbering" onclick="mdcp.copyCode(event)" style="position: unset;">npm install <span class="token operator">-</span>g cnpm <span class="token operator">--</span>registry<span class="token operator">=</span>https<span class="token punctuation">:</span><span class="token comment">//registry.npm.taobao.org</span>
<div class="hljs-button {2}" data-title="复制"></div></code><ul class="pre-numbering" style=""><li style="color: rgb(153, 153, 153);">1</li></ul></pre> 
<p>②如果是，检查npm模块本地存放位置是否有cnpm文件夹<br> ③新建一个管理员身份的cmd命令行窗口再次执行<code>cnpm -v</code></p> 
<p>ps：如果将npm默认的registry修改为淘宝registry后，使用npm下载时就会使用国内的淘宝镜像，如果大家安装cnpm遇到报错或者安装后仍然有问题，就可以不用再安装cnpm了，npm淘宝镜像和cnpm两种方案选择其一即可。当然，小朋友才做选择，作为成年人，大家也可以都要（不是</p> 
<p><font size="5"><strong>写在最后</strong></font></p> 
<p>安装Node.js后，可通过安装Vue.js运行vue项目，Vue.js安装与创建默认项目可见下一篇博文：<a href="https://blog.csdn.net/qq_42006801/article/details/124852760">https://blog.csdn.net/qq_42006801/article/details/124852760</a></p>
                </div>
