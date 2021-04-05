---
title: CLI转为http服务小探
subtitle: tech
date: 2020-12-02 01:29:22
updated:
tags: [CLI, http, system]
categories: [专业]
---

# 背景

需要将现成的CLI转换为web服务的HTTP接口.

这个我记得之前看到哪篇文章推荐过, 但是一时想不起来了

想了想关键词

* CLI
* command line
* http
* server
* web 
  * 终于把web这个带上之后, 用CLI to web server终于搜到一个相近的
  * w2c 这个python 程序
    * [dongyx/c2w: convert CLI programs to Web services](https://github.com/dongyx/c2w)
  * 然后用这个里的描述, google终于找到了相似语义的, 我少了program, 导致CLI应该被理解成了终端输入参数的web服务, 也就是常见的web服务...
  * converts a CLI program to a Web service
    * [The Return of the CLI: CLIs Being Used by API\-Related Companies \| Nordic APIs \|](https://nordicapis.com/the-return-of-the-cli-clis-being-used-by-api-related-companies/)
      * 初看没看懂
    * [How can I convert my console application to a web service using visual studio 2013 \- CodeProject](https://www.codeproject.com/Questions/1129689/How-can-I-convert-my-console-application-to-a-web)
      * hhh, 一个咨询解决方案的, 问题一致.
      *  只不过我实现了一套, 现在想看看有没有更好的实现方案
   *  [adamkewley/jobson: A platform for transforming command\-line applications into a job service\.](https://github.com/adamkewley/jobson)
      *  这个东西似乎是转换成任务, 结果不知道有没有返回的地方
   *  [\(1\) \[FOSS\] Convert any command line tool/service into a asynchronous REST API\. \[Python\] : linux](https://www.reddit.com/r/linux/comments/i8p2fe/foss_convert_any_command_line_toolservice_into_a/)
      *  又拿到一个关键词, shell2http
      *  [Eshaan7/Flask\-Shell2HTTP: Execute shell commands via HTTP server \(via flask's endpoints\)\.](https://github.com/Eshaan7/Flask-Shell2HTTP)
      *  [adnanh/webhook: webhook is a lightweight incoming webhook server to run shell commands](https://github.com/adnanh/webhook)
         *  这个比较牛看起来, Star特别高, go实现
   *  [The Tech Feast: Expose Any Shell Command or Script as a Web API](http://techfeast-hiranya.blogspot.com/2015/06/expose-any-shell-command-or-script-as.html)
      *  bash2http, 和上面的shell2http差不多
*  shell 2 http
   *  remote sh这个工具看起来也不错, 只是在页面填写bash 脚本, 然后指定服务器执行, 等同于web终端的感觉.
*  command web server
*  expose shell scripts as web services
   *  又一个关键词 API
   *  [lukasmartinelli/nigit: Web server that wraps around programs and shell scripts and exposes them as API](https://github.com/lukasmartinelli/nigit)
      *  wrap command to http API
      *  [phonkee/goexpose: Expose shellscripts, postgres queries, redis commands and other as rest server endpoints](https://github.com/phonkee/goexpose)
         *  上一个开发者不维护了, 推荐用这个
   *  [Andreweweith/Web\-Based\-Remote\-Command\-Server: Web server and interface to remotely execute Linux shell commands and display the results](https://github.com/Andreweweith/Web-Based-Remote-Command-Server)
      *  这个是C实现的一个http上输入命令底层执行,然后返回结果的那种.
   *  [bytestream/Web\-Server: During my time at the University of Miami whilst studying CSC524 \- Computer Networks I was also asked to build a basic web server using pure C\. The web server can be used to process GET and POST requests whereby GET requests will serve a static file and POST requests with the addition of POST data will execute the file as if it were CGI and pipe the output back to the client\. Again for security reasons it only serves files in the current working directory, and will also only execute shell programs\.](https://github.com/bytestream/Web-Server)
      *  别人的一个大作业的成果.也是C实现的一个把shell通过API暴露.
   *  [dotnet/command\-line\-api: Command line parsing, invocation, and rendering of terminal output\.](https://github.com/dotnet/command-line-api)
      *  .Net实现的, 感觉我的实现架构比较贴近这个
*  wrap command to http API
   *  [Soaplab2](http://soaplab.sourceforge.net/soaplab2/)
      *  看起来有点像需求的?
   *  [Genivia \- Getting Started with C/C\+\+ XML Data Bindings and XML SOAP/REST Web Services](https://www.genivia.com/dev.html)
   *  [gsoap使用总结 \- 苦涩的茶 \- 博客园](https://www.cnblogs.com/liushui-sky/p/9723397.html)
   *  WebService服务基本概念：就是一个应用程序，它向外界暴露出一个可以通过web进行调用的API，是分布式的服务组件。本质上就是要以标准的形式实现企业内外各个不同服务系统之间的互调和集成。
   *  所以以前其实这个实现形式叫做webservice. 只是我们不止是函数, 包括命令.
   *  所以这里其实针对静态语言以前的思路是实现一个DSL, 然后基于编写的DSL生成可编译的代码, 然后再编译出可执行程序, 对, 针对静态语言,其实这也是个思路, 使用yacc等词法分析器?  
   *  [How to wrap a C library so that it can be called from a web service \- Stack Overflow](https://stackoverflow.com/questions/2240258/how-to-wrap-a-c-library-so-that-it-can-be-called-from-a-web-service)
      *  这里也提到了soaplib
      *  mod-xmlrpc2
* webservice
  * command webservice
    * [c\# \- Consuming Web Service from C\+\+/CLI \- Stack Overflow](https://stackoverflow.com/questions/3574127/consuming-web-service-from-c-cli)
      * emm.
    * [SyBooks Online](http://infocenter.sybase.com/help/index.jsp?topic=/com.sybase.infocenter.dc31727.0600/html/easws/CHDJGCDJ.htm)
* proxy 忽然想到, proxy其实和wrap在这个场景里是同义词.

# 小结
所以根据上面的思路来看

## 流程

* 初始化
  * 解析配置文件, 其中包含CLI或函数对应的API路由
* decoder解析http协议
* dispatch根据路由转发, 路由注册
* 进入函数或传入命令参数进行执行
  * 可能有前置处理参数逻辑, 一般定制性需要强一些
  * 后置输出格式处理逻辑等
* encoder协议拼装返回


## 思路

* 针对支持运行时元编程能力的语言(如python, go), 和我实现思路基本一致, 编写一套配置或叫做DSL, 运行时加载然后动态生成函数等.
* 针对不具有上述能力的语言, 有两种思路
  * 参数传入, 核心调度逻辑的函数基本只有一个
    * 如果是CLI, 则作为参数传入, 有一个统一system调用的函数封装
      * 如果是前置和后置逻辑需要用函数处理, 则像下面一样用字符串找到符号表里的函数指针.
    * 如果是函数, 我想的就是通过动态库的`dlysm`或者`libeffi`来根据配置中填写的函数名, 去进行调度.
  * 编写一套DSL, 在进入编译前, 根据编写的DSL生成一套代码, 这样就支持各式各样的函数了, 就是定制性维护可能代码量膨胀的较快? gsoap和soaplib应该是这套思路吧, 好像有点像是10年前的思路, 看snmp代码的时候也是这种思路, 从mib生成代码.
    * 这个的实现方式应该就多种多样了.