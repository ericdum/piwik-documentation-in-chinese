JavaScript跟踪
=====
文档 > JavaScript跟踪

原文：[http://piwik.org/docs/javascript-tracking/](http://piwik.org/docs/javascript-tracking/)

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
	* 更改跳转等待时间
	* 更改要跟踪为“download”的文件扩展列表
* 多piwik跟踪器
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

在这种情况下，自定义变量在第一方的cookie中记录了持续的访问（上一次操作30分钟内）。你可以用`piwikTracker.getCustomVariable.`取得自定义变量名和值。如果请求的索引中没有自定义变量，会返回false。

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
###更改跳转等待时间

当一个用户点击下载一个文件或者点击站外链接的时候，Piwik要记录它。在流程中，它在用户跳转到请求的文件或者是链接之前加入一个短暂的延时。默认值是500ms，但是你可以设置一个更多的时间。你要注意，不管怎样，这个结果是有风险的。因为这个时间可能不足以让数据被Piwik记录到。

	[...] _paq.push(['setLinkTrackingTimer', 250]); // 250 milliseconds 
	_paq.push(['trackPageView']);
	[...]

###更改要跟踪为“download”的文件扩展列表

默认情况下，任何文件以这些扩展名编码的文件都会在Piwik后台被记录成“download”：

	7z|aac|arc|arj|asf|asx|avi|bin|bz|bz2|csv|deb|dmg|doc|
	exe|flv|gif|gz|gzip|hqx|jar|jpg|jpeg|js|mp2|mp3|mp4|mpg|
	mpeg|mov|movie|msi|msp|odb|odf|odg|odp|ods|odt|ogg|ogv|
	pdf|phps|png|ppt|qt|qtm|ra|ram|rar|rpm|sea|sit|tar|
	tbz|tbz2|tgz|torrent|txt|wav|wma|wmv|wpd||xls|xml|z|zip

要替换你想要做为“download”来跟踪的扩展名列表，你可以用`setDownloadExtensions( string )`：

	[...] _paq.push(['setDownloadExtensions', "jpg|png|gif"]); // 我们现在只跟踪图片的点击
	_paq.push(['trackPageView']);
	[...]
	
如果你想要跟踪一个新的问价那类型，你可以用`addDownloadExtensions( filetype )`只将他加入到列表中：

	[...] // 点击MP5或者MP6之后会被记录成下载
	_paq.push(['addDownloadExtensions', "mp5|mp6"]); 
	_paq.push(['trackPageView']);
	[...]	


##多piwik跟踪器

可以用多个Piwik跟踪器来指向同一个或者不同的piwik服务器。为了提高页面的加载时间，你可以只加载piwik.js一次。每一个`Piwik.getTracker()`的调用都会返回一个唯一可供你配置的Piwik Tracker对象（实例）。

当使用多跟踪器的时候，你必须用异步js tracker对象：

	<script type="text/javascript"> 
	try { 
		var piwikTracker = Piwik.getTracker("http://URL_1/piwik.php", 1); 
		piwikTracker.trackPageView(); 
		var piwik2 = Piwik.getTracker("http://URL_2/piwik.php", 4); 
		piwik2.trackPageView(); 
	} catch( err ) {} 
	</script>
	
请注意你也可以手动设置网站ID和Piwik跟踪器URL，而不用在`getTracker`的调用中设置。

	// 替换 Piwik.getTracker("http://example.com/piwik/", 12)
	var piwikTracker = Piwik.getTracker();
	piwikTracker.setSiteId( 12 );
	piwikTracker.setTrackerUrl( "http://example.com/piwik/" );
	piwikTracker.trackPageView();

##跟踪API可用方法列表

_从Piwik类中请求Tracker实例。_

* Piwik.getTracker( trackerUrl, siteId ) - 取得新的实例。
	* [Google Analytics equivalent] _getTracker(account)
	* [Yahoo! Analytics equivalent] getTracker(account)
* Piwik.getAsyncTracker() - 取的tracker用来异步跟踪的内部实例。get the internal instance of the Tracker used for asynchronous tracking

_使用Tracker对象_

* enableLinkTracking( enable ) - 在所有合适的链接中安装链接跟踪。将enable参数设置为true来使用伪点击来跟踪鼠标中键不能触发点击事件的浏览器（如Firefox）。 默认只有设置为true才会跟踪。
* addListener( element ) - 给特定的链接添加click侦听器。当它被点击的时候，Piwik会记录这个自动点击。
* setRequestMethod( method ) - 设置一个"GET"或"POST"请求方法（默认为"GET"）。要使用"POST"方法，Piwik的域名必须和要跟踪的一名一致。
* trackGoal( idGoal, [customRevenue]); - 为目标idGoal手动记录一个转化，如果需要可以传入一个自定义的收入。
* trackLink( url, linkType ) - 用js手动记录一个点击。url是要被记录的完整的URL。linkType只能是离开链接"link"或者是下载链接"download"。
* trackPageView([customTitle]) - 记录页面的访问
	* [相当于Google Analytics] _trackPageview(opt_pageURL)
	* [相当于Yahoo! Analytics] submit()
* trackSiteSearch(keyword, [category], [resultsCount]) - 记录内部搜索的特定关键字, 分类、结果数量都是可选的。

_配置Tracker对象_

* setDocumentTitle( string )- 重写`document.title`
	* [相当于Yahoo! Analytics] YWATracker.setDocumentName("xxx")
* setDomains( array ) - 设置主机名或者是域名。有子域名可用通配符：
		
		setDomains('.example.com');
或者

		setDomains('*.example.com');
		
	* [相当于Google Analytics] _setDomainName(".example.com")
	* [相当于Yahoo! Analytics] setDomains("*.abc.net")
* setCustomUrl( string ) - 重写页面URL
* setReferrerUrl( string ) - 重写Http Referer
* setSiteId( integer ) - 设置网站ID。多余的：可以在getTracker()构造函数中指定
* setTrackerUrl( string ) -指定Piwik服务器URL。多余的：可以在getTracker()构造函数中指定
* setDownloadClasses( string | array ) - 设置要当成下载记录的class（作为piwik_download的补充）
* setDownloadExtensions( string ) - 设置需要记录为下载的文件扩展名列表，如：'doc|pdf|txt'
* addDownloadExtensions( string ) - 添加额外需要记录为下载的文件扩展名列表，如：'doc|pdf|txt'
* setIgnoreClasses( string | array ) - 设置class给要忽略的链接（作为piwik_ignore的补充）
* setLinkClasses( string | array ) - 设置class给离开链接（作为piwik_link的补充）
* setLinkTrackingTimer( integer ) - 设置以毫秒记的链接跟踪延时.
* discardHashTag( bool ) - 设置为true以不记录URL的hash tag (anchor)
* setGenerationTimeMs(generationTime) - Piwik默认会用浏览器DOM的时间API来精确得测量页面的加载时间。你可以用一个毫秒数值来重写这个记录。
* appendToTrackingUrl(appendToUrl) - 添加自定义字符串到HTTP请求`piwik.php?`的最后
* setDoNotTrack( bool ) - 设置为true以不跟踪在浏览器中设置了不跟踪的用户。
* disableCookies() - 禁用所有第一方cookie。当前站点现有的Piwik cookies将在下一个page view被删除。
* killFrame() - 防止页面被包含在frame或者iframe中，如果那样，会强制将上层页面跳转到当前页。
* redirectFile( url ) - 如果当前页是被人保存下来打开的，就会强制跳转到这个url（如另存为网页到桌面）。
* setHeartBeatTimer( minimumVisitLength, heartBeatDelay ) - 如果超过minimumVisitLength（秒）记录访问者的停留时间；heartBeatDelay指定更新这个数据的频率；
* getVisitorId() - 返回16个字符的访问者ID
* getVisitorInfo() - 返回访问者的cookie数组。
* getAttributionInfo() - 返回访问者的属性数组(Referer 和/或 Campaign name & keyword).

	属性信息是Piwik信任的正确来源 ([first or last referrer](http://piwik.org/faq/general/#faq_106))到目标转化.
	
	你还可以使用下面这些函数来获得特定的属性数据：
	* piwikTracker.getAttributionCampaignName()
	* piwikTracker.getAttributionCampaignKeyword()
	* piwikTracker.getAttributionReferrerTimestamp()
	* piwikTracker.getAttributionReferrerUrl()
* setCustomVariable (index, name, value, scope) - 设置自定义变流量
* deleteCustomVariable (index, scope ) - 删除自定义变量
* getCustomVariable (index, scope ) - 获取自定义变量
* setCampaignNameKey(name) - 自定义 Campaign 参数的key
* setCampaignKeywordKey(keyword) - 自定义 Keyword 参数的key
* setConversionAttributionFirstReferrer( bool ) - 设置true以标记转化到第一个来源地址。默认情况下，转化会标记到最近的来源地址。

_跟踪cookie的配置_

从Piwik 1.2开始，第一方cookie被使用。必须要要考虑的是保留时间、避免和其他的cookie、跟踪系统、应用发生冲突。

* setCookieNamePrefix( prefix ) - 默认前缀为 '_pk_'.
* setCookieDomain( domain )- 默认为`document.domain`。如果你的网站可以同时通过 www.example.com和example.com访问，你需要使用：

		tracker.setCookieDomain('.example.com');
or

		tracker.setCookieDomain('*.example.com');
	
* setCookiePath( path ) - 默认为 '/'.
* setVisitorCookieTimeout( seconds ) - 默认为 2 years
* setSessionCookieTimeout( seconds ) - 默认为 30 minutes
* setReferralCookieTimeout( seconds ) - 默认为 6 months


###单元测试覆盖piwik.js

Piwik JavaScript跟踪API是使用一个JavaScript单元测试来尽可能保证它的质量的，我们从未让功能坏过。测试使用的是Qunit。要入刑这个测试，直接查看[Piwik Git仓库](http://piwik.org/participate/contributing-with-git/ "Piwik Git repository")，然后到/path/to/piwik/tests/javascript/。在浏览器中测试。

Piwik JavaScript API在很多浏览器中进行过测试。为了最大化覆盖率，我们使用了像[crossbrowsertesting.com](http://crossbrowsertesting.com/)和[browsershots.org](http://browsershots.org/)这样的服务。


###压缩piwik.js

你的访问者必须下载的这个piwik.js是为了减小他大小被压缩过的。YUI压缩器可以用来压缩Javascript的（[详情](https://github.com/piwik/piwik/blob/master/js/README#L1 "more information")）。你可以在/js/piwik.js中找到没有被压缩过的原始版本。

有问题？

如果你有任何关于使用Piwik JavaScript跟踪的问题，[请在本站搜索](http://piwik.org/search/ "please search the website")或者[在论坛中提问](http://forum.piwik.org/ "ask in the forum")。Enjoy！

##相关文档

* [Piwik能不用js跟踪吗？](http://piwik.org/faq/new-to-piwik/#faq_63 "Does Piwik track visitors without JavaScript?")
* [Piwik怎么跟踪下载？](http://piwik.org/faq/new-to-piwik/#faq_47 "How does Piwik track downloads?")
* [怎么跟踪错误页面（404页面）？让我知道404个URL和引入的页面？](http://piwik.org/faq/how-to/#faq_60 "How to track error pages (404 pages) in Piwik? Which URLs are 404 and which referers lead to these pages?")
* [怎么设置一组自定义页面（结构）让他们的page view以分类进行总和？](http://piwik.org/faq/how-to/#faq_62 "How can I set custom groups of pages (structure) so that page view are aggregated by categories?")
* [PIwik的JavaScript跟踪代码和XHTML1.0兼容吗？](http://piwik.org/faq/general/#faq_66 "Is the Piwik JavaScript Tracking Code XHTML 1.0 compatible?")
* [Piwik在所有报告中显示“无数据”。](http://piwik.org/faq/troubleshooting/#faq_58 "Piwik shows "No Data" in all reports.")
* [从搜索引擎或者广告来的访问没有被记录，Piwik显示“没有可用数据”。](http://piwik.org/faq/troubleshooting/#faq_51 "Visits from search engines or campaigns are not recorded, Piwik shows "No data available".")
* [为什么piwik的统计和其他分析工具的统计不一致（日志分析或者基于js的统计）？](http://piwik.org/faq/troubleshooting/#faq_50 "Statistics from Piwik and my other web analytics tool (log analyzer or javaScript based) are different, why?")
* [Piwik的js代码在IE上显示一个红叉或者一个坏掉的图片图标而不是一个1*1像素的透明图标？](http://piwik.org/faq/troubleshooting/#faq_57 "The Piwik JavaScript code shows a red cross (on IE) or a broken image icon instead of the 1*1 transparent Pixel, what  is the issue?")
* [怎么在js中禁用所有的跟踪cookie](http://piwik.org/faq/general/#faq_157 "How do I disable all tracking cookies used by Piwik in the javascript code?")
