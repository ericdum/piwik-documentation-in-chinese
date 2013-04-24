如何使用日志分析工具
=====
Piwik服务器日志分析 > 如何使用日志分析工具

本页解释了如何用Piwik日志分析工具将服务器日志导入到Piwik

* 系统要求
* 如何：使用默认参数运行日志文件分析脚本
* 如何：导入其他数据包括机器人、静态文件、HTTP错误报告
* 如何：排除特定的记录


系统要求
----
* 安装或升级Piwik。需要花大概5分钟以内。
* 要执行脚本，你需要通过SSH或者其他能执行脚本的途径访问你的服务器
* 需要Python 2.6。注意：载入和解析日志文件的脚本是用Python写的，但是Piwik的API是用PHP5写的。
* 你还需要一个或多个日志文件来用Piwik解析、分析。（日志文件内的每一行记录都必须是按时间排序的）
* 注意：我们建议你使用包含User Agent, Referrer URL和完整URLs的扩展日志格式。如果这些字段没有在日志中，分析数据可能不精确。
* 设置GEO Location来精确探测国家和城市。Piwik推断访问者的国家是基于访问者的浏览器语言的，但是这些信息访问日志（access log）都不支持，所以Geo Location是必要的。
* 最低版本：Piwik 1.7.2，但是我们始终建议更新到最新的版本。

**注意：数据精确性**

使用服务器日志导入（相对于js跟踪）会有少部分用户数据点丢失：屏幕分辨率、浏览器插件和页面（title）都不支持 （report Actions > Page Titles大部分会是空）。跟踪cookies不能被用于计算“一些丢失的数据点”，见“FAQ”。

如何：使用默认参数运行日志文件分析脚本
----
一旦Piwik开始运行，你将会找到这个脚本：misc/log-analytics/import_logs.py

	$ python /path/to/piwik/misc/log-analytics/import_logs.py
	
这个会显示在帮助信息中。唯一必要的参数是：

	--url=http://analytics.example.com

用来指定Piwik的根URL。然后，你可以指定一个或多个日志文件用于导入。

查看输出帮助和README获得更多可用参数的信息和说明。

如：如果你想要跟踪所有请求（静态文件、机器人请求、HTTP错误、HTTP 跳转），可以使用下面的命令

	python /path/to/piwik/misc/log-analytics/import_logs.py --url=http://analytics.example.com access.log
	--idsite=1234 --recorders=4 --enable-http-errors --enable-http-redirects --enable-static
	--enable-bots


如何：导入其他数据包括机器人、静态文件、HTTP错误报告
----
脚本默认不会跟踪静态文件（JS、CSS、图片等）和所有机器人流量。

你可以用下面的命令来启用这些流量：

*		--enable-bots

用于跟踪搜索、垃圾信息机器人，并给它们指定一个自定义变量名：Bot。启用后，日志文件会需要更长的处理时间知道所有机器人的page view都被传到Piwik中。

*		--enable-static
	
用于指定跟踪静态文件（图片、JS、CSS）。这会延长一些日志的处理时间。

*		--enable-http-errors
	
用于指定跟踪page view的HTTP错误（4xx、5xx状态），并将自定义变量HTTP-code设置成400、500等。如果有的话，页面title会显示成URL referrer（用于帮助找到是哪一个页面链接到了不错在的页面等）。

*		--enable-reverse-dns
	
用于启用反向DNS（用于生成访客>服务提供商报告），可用于探测诸如DNS太慢引起的一些性能问题。

*		--recorders=N
	
指定线程的数量：我们建议设置成系统的CPU的核心数量。（或者视你的服务器配置而定）

*		--recorder-max-payload-size=N

这个入口用于批量跟踪来达到更好的性能。默认300PV（日志记录）会被一次性发送到Piwik。你可以实验这个数字来获得更好的性能，但是需要有个上限。


如何：排除特定的记录
----
有几个方式来排除特定被跟踪的日志记录或者访客。

* 你可以排除特定IP地址或者IP段。通过用管理员登陆Piwik，在“设置 > 网站”中配置excluded IPs
* 脚本提供一个选项通过HTTP头的User Agent排除来访问者：

		--useragent-exclude	

* 脚本提供一个选来来强制使用URL域名白名单，不再白名单中的域名会被过滤：

		--hostname
		
* 它可以用于通过URL路径排除一些特定的日志记录：

		--exclude-path
