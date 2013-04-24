JavaScript跟踪
=====
文档 > JavaScript跟踪

* 跟踪代码在哪里找？
* JavaScript跟踪的功能
	* 自定义Page Name
	* 点击或特定事件触发Page View
	* 手动触发触发转化目标
	* 跟踪订单、购物车更新、产品、分类的Page View
	* 跟踪内部搜索关键字、分类和没有结果的搜索关键字
	* 自定义变量
	* 可为子域单独配置Cookie
	* 在离开链接（Outlinks）跟踪中忽略特定域或者子域
	* 禁用下载、离开链接跟踪
	* 禁用特定CSS class的下载、离开链接跟踪
	* 禁用特定链接的下载或离开链接跟踪
	* 强制将一个链接的点击记录成下载
	* 强制将一个链接的点击记录成离开
	* _变更暂停计时_
	* _将文件扩展列表改为下载跟踪_
* 多项目跟踪
* 跟踪API可用方法列表
 	* 单元测试覆盖piwik.js
 	* 压缩piwik.js
* 相关文档

Piwik装备有强大的JavaScript跟踪API。高级用户可以使用Piwik跟踪代码来自定义一些网站分析的数据。插件开发者也可以发送自定义JSON数据到Piwik的跟踪脚本并给插件使用。

###跟踪代码在哪里找？

所有的功能都会在本页介绍，你需要使用最新版本的跟踪代码。要找到你的跟踪代码，请按照下列步骤寻找：

1. 用admin或者super user账号登陆到Piwik
2. 点击设置（Setting）访问管理页面
3. 点击网站（Websites）查看正在跟踪的网站列表
4. 在相关网站后点击查看跟踪代码（View Tracking Code）
5. 复制粘贴跟踪代码到你的页面，紧跟在</body>标签之前

跟踪代码看起来和下面的代码类似

	<!-- Piwik --> <script type="text/javascript"> 
	var _paq = _paq || []; 
	(function(){ var u=(("https:" == document.location.protocol) ? "https://{$PIWIK_URL}/" : "http://{$PIWIK_URL}/"); 
		_paq.push(['setSiteId', {$IDSITE}]); 
		_paq.push(['setTrackerUrl', u+'piwik.php']); 
		_paq.push(['trackPageView']); 
		_paq.push(['enableLinkTracking']); 
		var d=document, g=d.createElement('script'), s=d.getElementsByTagName('script')[0]; 		g.type='text/javascript'				g.defer=true; g.async=true; 		g.src=u+'piwik.'; 
		s.parentNode.insertBefore(g,s); }();
 	</scrpt> 
	<!-- End Piwik Code -->
	
在你的跟踪代码中，{$PIWIK_URL}会被你的Piwik地址替换，{$IDSITE}会被你的网站id所替换。

与你熟悉的javascript代码相比，这个代码可能看起来有一点奇怪，但这是因为他是异步执行的。换句话说，浏览器不会因为等待piwik.js文件下载而不显示你的页面。

为了异步跟踪，跟踪配置被储存在全局的_paq数组中等待piwik.js下载完成后执行。格式是：

	_paq.push([ 'API_method_name', parameter_list ]);
	
你也可以push一个function来执行，比如：

	var visitor_id;
	_paq.push([ function() { visitor_id = this.getVisitorId(); }]);
	
或者例如，用这个异步代码获取一个自定义变量（name, value）：

	_paq.push(['setCustomVariable','1','VisitorType','Member']); 
	_paq.push([ function() { var customVariable = this.getCustomVariable(1); }]);
	
你甚至可以在piwik.js加载完成并执行后再push到_paq数组。

##JavaScript跟踪的功能
####自定义Page Name

默认情况下，Piwik使用当前页面的URL未页面title。如果你的URL比较复杂，或者你想要自定义你的页面，你可以指定页面名称。

一个通用的方法是用document title设置HTML的Title：

	[...] 
	_paq.push(['setDocumentTitle', document.title]);
	_paq.push(['trackPageView']);
	[...]
	
高级用户也可以自动生成一个页面名称，比如在php里面：

	[...] 
	_paq.push(['setDocumentTitle', "<?php echo $myPageTitle ?>"]);
	_paq.push(['trackPageView']);
	[...]
	
###点击或特定事件触发Page View

默认情况下，Piwik是根据js跟踪代码在各个页面的加载时执行来跟踪Page View的。然而，一个现代的网站或者web app、user interactions不需要总是加载一个新页面。比如，当用户点击一个js链接或者当他们点击一个tab（页面中用js控制的）或者当他们和页面产生交互的时候，你也可以进行跟踪。

要跟踪用户交互或者点击，你可以手动地调用Javascript函数`trackPageView()`。比如，如果你想要跟踪一个菜单是否被点击，你可以：

	[...] <a href="#" onclick="javascript:_paq.push(['trackPageView', 'Menu/Freedom']);">Freedom page</a>

###手动触发触发转化目标

默认情况下，Piwik的目标是以匹配一部分URL（开始于，包含或者正则表达式）来定义的。你也可以以page view、下载或者离开链接的点击来跟踪目标。

在一些情境中，你可能想要在其他类型的行为中记录转化，比如：

* 当一个用户提交表单
* 当一个用户在页面停留超过了一定的时间
* 当一个用户在flash中产生了一些交互
* 当一个用户提交了他的购物车并且完成了支付：你可以将Piwik跟踪代码交给支付网站，它会记录自定义的转化收入到你的piwik数据库中。

要使用JavaScript跟踪代码触发目标，你可以简单地：

	[...] _paq.push(['trackGoal', 1]); // 给目标1记录一次转化
	[...]

你也可以在记录一个转化的同时记录自定义的收入。比如，你可以生成一个trackGoal的调用来自动设置转化的价值

	[...]
	// logs a conversion for goal 1 with the custom revenue set
	_paq.push(['trackGoal', 1, <?php echo $cart->getCartValue(); ?>]);
	[...]
	
更多关于目标跟踪的的信息在[跟踪目标](http://piwik.org/docs/tracking-goals-web-analytics/ "跟踪目标")的文档中

###跟踪订单、购物车更新、产品、分类的Page View

Piwik允许为高级、强大的商业跟踪。查看[商业分析](http://piwik.org/docs/ecommerce-analytics/ "商业分析")文档可以了解到商业报告以及如何设置商业跟踪。

###跟踪内部搜索关键字、分类和没有结果的搜索关键字

Piwik有高级的[网站搜索分析](http://piwik.org/docs/site-search/ "网站搜索分析")功能，让你跟踪用户是如何使用内部搜索引擎的。默认情况下，Piwik可以读取包含搜索关键字的URL参数。而且，你也可以使用JavaScript函数`trackSiteSearch`手动记录搜索关键字。

在你的网站中，在标准的页面里，你需要通过调用标准的`piwikiTracker.trackPageView()`来记录Page View。在搜索结果页面，你可以用`piwikTracker.trackSiteSearch(keyword, category, searchCount)`函数来记录内部搜索请求。注意：“`keyword`”参数是必须的，但是`category`和`searchCount`是可选的。

	[...]
	_paq.push(['trackSiteSearch', 
	"Banana", // 搜索关键字
	"Organic Food", // 在搜索引擎在选择的分类，如果不需要，设置为false  
	0 // 搜索结果页面中的结果数量。0表示“该关键词没有结果”。如果你不知道，设置为false。
	]);
	// 我们建议不要在搜索结果页面调用 trackPageView() 
	// _paq.push(['trackPageView']); 
	[...]
	
我们当然非常建议在Piwikk报告“没有结果的关键字”时设置`searchCount`参数。比如，搜索的关键没有返回结果，通常会对用户为什么会去搜没有结果的关键字产生帮助。详见[网站搜索分析](http://piwik.org/docs/site-search/ "网站搜索分析")

###自定义变量

自定义变量是一个强大的功能，它能让你为不同的访问和（或）page view设置自定义的变量。请参考[跟踪自定义变量](http://piwik.org/docs/custom-variables/ "跟踪自定义变量")。

你可以给每个访问和（或）page view设置最多5个自定义变量（name、value）。如果你设置一个自定义变量给一个访问者，当他在1个小时或者两天后回来的时候，他会成为一个新访问者，并且自定义变量会被清空。

这里有两个“scopes”可供你设置自定义变量。这个“scope”是`function setCustomVariable()`的第四个参数。

* 当`scope = "visit"`时，自定义变量名和值会在数据库中被储存到访问visit。你可以因此为每个scope为visite的访问储存最多5个自定义变量。
* 当`scope = "page"`时，自定义变量名和值会被储存在被跟踪的page view中。你可以因此为每个scope为page的page view储存最多5个自定义变量。

自定义变量统计报告在访客（VIsitors）> 自定义变量（Custom Variables）中。scope为visit还是page都综合在这个报告中。

####给访问设置自定义变量

	setCustomVariable (index, name, value, scope = "visit")

这个函数用来创建或更新一个自定义变量名和值。比如，想象你要储存每一个用户的性别。你需要这样储存一个自定义变量`name = "gender"`，`value = "male" or "female"`。

**重要：**一个给定的自定义变量名必须永远储存在相同的索引中。比如，如果你选择要储存一个变量`name = "Gender"`在`index=1`中，并且你要记录另外一个自定义变量在`index=1`，然后这个"Gender"变量会被删除并且被新的储存在`index=1`的自定义变量所替换。

	[...]
	_paq.push(['setCustomVariable',  
	1, // 索引（1-5），自定义变量储存的位置。
	"Gender", // 变量名， 自定义变量的名称，如：Gender, VisitorType 
	"Male", // 值，比如： "Male", "Female" 或者 "new", "engaged", "customer" 
	"visit" // 自定义变量的Scope，"visit" 表示这个自定义变量是一个访问的。
	]); 
	_paq.push(['trackPageView']); 
	[...]
	
你只需要设置变量scope一次，然后它会在整个访问中所被记录。

#####给page view设置一个自定义变量
	setCustomVariable (index, name, value, scope = "page")

和跟踪访问自定义变量一样，有时候跟踪独立的page view自定义变量也非常有用。比如，一个新闻网站或者博客，一个给定的文章可以会属于一个或多个分类。在这种情况下，如果一篇文章同时属于Sports和Europe分类，你可以设置两个个自定义变量：`name="category"`，一个`value="Sports"`另一个`value="Europe"`。这个自定义变量报告会统计出每一格分类有多少访问和page view。这个信息在标准报告中很难获得，因为那些不带有分类信息的记录只能统计出“最好的页面URL”和“最好的页面标题”。

	[...]
	// 在不同的位置跟踪两个同名的自定义变量 
	// 紧接着你可以在 '访问者（Visitors） > 自定义变量（custom variables）' 中产看报告。
	_paq.push(['setCustomVariable', 1, 'Category', 'Sports', 'page']); 
	_paq.push(['setCustomVariable', 2, 'Category', 'Europe', 'page']); // 跟踪同样的变量名，但索引不同
	// 你可以在索引（3、4、5）中跟踪其他page view的自定义变量
	_paq.push(['trackPageView']); 
	[...]

**重要：**你可以在visit scope的索引1中储存一个自定义变量，也可以在page scope的索引1中储存另一个自定义变量。因此，理论上你可以在**每个页面最多跟踪10个自定义变量名和值**（其中5个在page scope，5个在visit scope）。

	[...]
	_paq.push(['setCustomVariable',
	1, // 索引（1-5），自定义变量储存的位置。
	"category", // 变量名， 自定义变量的名称，如：Category, Sub-category, UserType 
	"Sports", // 值，比如："Sports", "News", "World", "Business", 等等 
	"page" // 自定义变量的Scope， "page"表示这个自定义变量是一个page view的。
	]); 
	_paq.push(['trackPageView']); 
	[...]
	
####删除一个自定义变量

	deleteCustomVariable (index, scope )

如果你创建了一个自定义变量，又决定要从访问或page view中删除这个变量，你可以使用`deleteCustomVariable`。

要将这个变更到服务器，你必须在调用`trackPageView()`之前调用这个函数。

	[...]
	_paq.push(['deleteCustomVariable', 1, "visit"]); // 删除当前访问中索引1的自定义变量。
	_paq.push(['trackPageView']);
	[...]

####取得一个自定义变量

	getCustomVariable (index, scope )

这个函数主要用于`scope = "visit"`的时候。

在这种情况下，自定义变量在第一部分的cookie中记录了持续的访问（上一次操作30分钟内）。你可以用`piwikTracker.getCustomVariable.`取得自定义变量名和值。如果请求的索引中没有自定义变量，会返回false。

	[...]
	_paq.push([ function() {
	var customVariable = this.getCustomVariable( 1, "visit" ); // 返回自定义变量: [ "gender", "male" ] 
	// do something with customVariable...
	}]);
	_paq.push(['trackPageView']);
	[...]

###可为子域单独配置Cookie
###在离开链接（Outlinks）跟踪中忽略特定域或者子域
###禁用下载、离开链接跟踪
###禁用特定CSS class的下载、离开链接跟踪
###禁用特定链接的下载或离开链接跟踪
###强制将一个链接的点击记录成下载
###强制将一个链接的点击记录成离开
###_变更暂停计时_
###_将文件扩展列表改为下载跟踪_
##多项目跟踪
##跟踪API可用方法列表
###单元测试覆盖piwik.js
###压缩piwik.js
##相关文档
