Piwik的服务器日志分析
====
Piwik的服务器日志分析

你知道吗？所有的web服务器都会创建access log文件，它们包含服务器接收到的所有请求。你现在可以将这些请求导入到Piwik进行分析了！我们提供一个日志脚本来让你解析、分析服务器访问日志，包括IP地址、URL、User Agent、Referrer URL以及搜索关键字、广告系列（计划、Campaign）信息等。

本页说明了如何使用免费的Piwik网站分析软件来导入你的日志文件到你的Piwik服务器。默认的Piwik跟踪，请考虑使用Javascript tracker.

* 如何用Piwik日志分析来解析日志
* 商业使用案例
* 日志分析FAQ
* 日志分析的功能和Piwik的工具集
* Piwik解析日志文件的Demo
* Feedback：你的功能需求

如何用Piwik日志分析来解析日志
====
使用日志分析，你需要最新版本的Piwik、Python和服务器访问日志来导入到Piwik。

文档在用户手册中：[如何运行日志分析工具](http://piwik.org/log-analytics/how-to/ "如何运行日志分析工具")

商业使用案例
====
想象你是销售网站主机的公司，并且有上百个用户。许多主机都想要购买先进、漂亮、强大、免费、自由的网站分析给他们的客户。你可以使用Piwik，从[日志分析商业使用案例](http://piwik.org/log-analytics/web-hosting-use-case/ "日志分析商业使用案例")中学习“如何整合Piwik日志分析到现有的商业模式中”。

日志分析FAQ
====
查看[日志分析FAQ](http://piwik.org/log-analytics/faq/ "日志分析FAQ")获得大多数让你焦虑的答案

日志分析的功能和Piwik的工具集
====
* 转换千兆服务器访问日志到漂亮的分析面板。
* 支持全部[50+ Piwik功能](http://piwik.org/features/ "50+ Piwik功能")：基于URL的目标跟踪、[匿名IP](http://piwik.org/privacy/ "匿名IP")、实时[访问日志]（http://piwik.org/docs/real-time/ "访问日志"）、搜索关键词、页面跟踪等。
* 在你自己的数据库中持续管理你的分析数据。
* 创建自定义插件或者重用大量的APIs。
* 使用匿名IP保护[用户隐私](http://piwik.org/privacy/ "用户隐私")
* 删除老日志的功能让数据库大小[尽在掌控](http://piwik.org/docs/managing-your-databases-size/ "尽在掌控")
* “不跟踪”功能。
* Sarbanes-Oxley和适用PCI
* 适用EU数据隐私的法律
* 识别多种数据库日志格式（Apache、Nginx、IIS等）。如果你的日志格式不能被识别请联系我们，我们将添加支持。
* 通过导入历史日志简单地从AWStats和Urchin整合到Piwik。为了未来的投资导入你的日志到Piwik
* 从报告中自动过滤机器人（搜索引擎、垃圾信息机器人等）。有一个配置选项来跟踪机器人并且给他们一个区别于真实访问者的自定义变量。
* 你获得了[免费自由软件](http://piwik.org/free-software/ "免费自由软件")

Piwik解析日志文件的Demo
====
我们在案例（Showcases）中设置了一个独立的以日志导入模式运行的Piwik演示，见[demo-log-analytics.piwik.org](http://demo-log-analytics.piwik.org/ "demo-log-analytics.piwik.org")。

这个演示展示了forum.piwik.org的Nginx访问日志生成的Piwik报告。我们在Piwik服务器上建立了两个网站来展示：1、默认日志导入模式；2、使用参数：–enable-http-errors –enable-http-redirects –enable-static –enable-bots 的完全模式

Feedback：你的功能需求
====
请向直接[通过工单](http://dev.piwik.org/trac/ticket/3163 "通过工单")向我们提交你的功能需求。如果你是一个python开发者，查看[源代码](https://github.com/piwik/piwik/blob/master/misc/log-analytics/import_logs.py "源代码")考虑[提交pull请求](http://piwik.org/participate/contributing-with-git/ "提交pull请求")

我们非常感谢如果你能给出改进的建议或者报告bugs。

我们希望你能喜欢用Piwik来跟踪你的服务器日志