---
title: flask_login源码阅读
subtitle: flask_login
date: 2018-11-23 22:40:47
updated:
tags: [Python]
categories: [专业]
---


flask_login实际上触发的操作，应该是通过在session中存储id来触发

@login_required

从下面的源码里可以看到

``` python
def login_required(func):
    @wraps(func)
    def decorated_view(*args, **kwargs):
        if request.method in EXEMPT_METHODS:
            return func(*args, **kwargs)
        elif current_app.config.get('LOGIN_DISABLED'):
            return func(*args, **kwargs)
        elif not current_user.is_authenticated:
            return current_app.login_manager.unauthorized()
        return func(*args, **kwargs)
    return decorated_view

```
首先是一个针对方法的过滤，之后根据config的设置无需登陆。最后就是对用户认证了。

这里需要找到current_user是在哪里注册的，以及是默认对象的实例还是继承后的。

之后，如果没有设置@LoginManager.unauthorized这个句柄注册回调函数的话，在Login Manager对象里的`unauthorized()`方法会触发

这里就提到了一个知识点，回调函数这是通过什么来注册的呢？signal？的确

## 信号

这里需要储备一些flask 关于signals的知识，官网说，主要是在扩展或者外部应用通过发送请求解耦使用的。允许发送者，通知订阅者，某件事情发生了。这里不鼓励订阅者修改数据。

信号看上去和内奸的装饰器`before_request()`等做了相同说的事情。然而，其实还是又不同的。比如关键的`before_request()`方法会以一个确定的顺序执行，并且可以abort请求直接返回一个响应，而信号的执行顺序是不定的，以及并不能改变什么。

(
根据大佬的测试，是实际测试发现signal执行实际是按照注册的顺序执行的，即先通过connect进行注册的回调函数会先被执行
)

信号的主要优点是可以快速安全的订阅。例如，有助于单元测试。

『这里的安全是什么呢？快速倒是可以猜测，当然是不清楚原理的』

你想要知道被渲染成一个请求信号的模板可以允许你做什么。

需要注意，必须要提供一个发送者，除非想要监听来自所有应用的信号。

确保含有一**extra,可以接收更多的参数

`connected_to()`方法似乎可以免去一些设置，但是暂时还不清楚

使用方式：

在自定义的命名空间中定义信号，

从flask_login中的使用来看，是设置到全局，然后要用的时候直接调用这个信号的send方法？

另外提一句，看到好多库都对导入的库做了一个fake,用来raise自己的Error信息哎。噢，不是，通过fake类依旧可以发送信号，但是对于其他的功能就不支持了。哇哦

不过似乎在 1.x 以上的源码中，必须装 blinker ，否则会有模块找不到的异常

flask官方就提供了一个这样的fake库给防止blinker的未安装,

为了保证始终具有一个sender, 如果是一个类发送，那么就传递self参数，如果是一个静态函数发送，那么就传递current_app._get_current_object()对象。

永远不要直接传递current_app作为发送者，因为current_app是一个代理独享，而不是真实的应用对象

当收到信号时，信号完全支持请求上下文。本地上下文变量在Request_started和request_finished之间一致可用， 所以你可以根据需要依赖flask.g和其他的

（是指在这个期间调用嘛？）

不过要注意在发送信号和`request_tearing_down`信号之间的限制

在Blinker1.1,可以使用新的`connect_via()`装饰器

```
from flask import template_rendered

@template_rendered.connect_via(app)
def when_template_rendered(sender, template, context, **extra):
    print 'Template %s is rendered with %s' % (template.name, context)


```

---

了解了flask基本的signal运作，可以知道信号的使用是需要注册回调函数的，但是flask_login的回调函数似乎并不是采用connect这种方式注册的，而是通过装饰器的方式存在类的实例内，


这里如果注册了unauthorized_callback，那么就会直接执行这个。


在`unauthorized`里提到，当被重定向时，可以通过`USE_SESSION_FOR_NEXT`决定通过session来存储next，而不是get的arg 参数。

另外，在这里提到，如果是对blueprint进行重定向，那么可以`blueprint_login_views`。如果不设置。就默认为app的login_view了。

这里对login_view，居然还支持完整的url路径，https之类的，或者就是直接通过url_for调用

默认session中的随机_id的生成，居然含有ip和user-agent，然后进行sha512，返回16进制

login_view注册，为什么还会含有next参数呢？噢，并不是，而是request.url，

有点不好理解这个make_next_param函数，login_url为空是什么意思？噢，这个是在解析存在session里的url，可能存进去的时候有那么些不同

----

接下来就是关键的在login_required这个装饰器中的current_user了

flask_login是如何注册的current_user这个对象呢？又是注册在哪里了？

``` python
current_user = LocalProxy(lambda: _get_user())
```

又用到了werkzeug这个关键库了。

这里只是传递一个函数，为什么要用一次lambda呢？

首先，是`_get_user()`这个方法，其中判断了`has_request_context()`的内容，

这个方法看上去是flask的ctx上下文模块内使用的。其主要功能就是在可以获取到request对象的时候快速使用，获取不到的时候就安静的不使用。

看来在这个请求上下文内记录了大量有效信息。_request_ctx_stack这里是一个LocalStack()对象

current_app同样也是，直接取_app_ctx_stack的栈顶对象

接下来看看LocalProxy对象，又调用了@implements_bool这个装饰器。

哇哇哇，接下来都是flask的代码，甚至werkzeug的代码，可怕，表现作为一个werkzeug local的代理对象。转发所有的操作到一个被代理的对象。唯一的不支持的操作是右操作数和任何赋值。



## LocalStack对象

将Local对象作为一个栈，在Local对象的实例里建了一个stack的list

## Werkzeug 

## Local对象

使用thread local对象虽然可以基于线程存储全局变量，但是在Web应用中可能会存在如下问题：

有些应用使用的是greenlet协程，这种情况下无法保证协程之间数据的隔离，因为不同的协程可以在同一个线程当中。
即使使用的是线程，WSGI应用也无法保证每个http请求使用的都是不同的线程，因为后一个http请求可能使用的是之前的http请求的线程，这样的话存储于thread local中的数据可能是之前残留的数据。
为了解决上述问题，Werkzeug开发了自己的local对象，这也是为什么我们需要Werkzeug的local对象[3]

为了解决上述问题，Werkzeug开发了自己的local对象，这也是为什么我们需要Werkzeug的local对象


get_ident获取线程id号


为什么这里都用object.__setattr__呢

如果没记错，这个和@property是起类似的效果的，但是这个顺序比较优先吧

除了这个装饰器，还有一个内置的property方法，

简单地说，property将一些代码（get_temperature和set_temperature）附加到成员属性（temperature）的访问入口。任何获取temperature值的代码都会自动调用get_temperature()，而不是去字典表（__dict__）中进行查找。同样的，任何赋给temperature值的代码也会自动调用set_temperature()。

## LocalManager

get_ident方法，不能被重载？，但是可以被用来和其他上下文本地对象相连接，可以在构造时被传递进来

在通过make_middleware返回时，使用了ClosingIterator传递进去了一个上下文结束时要执行的清理操作。

middleware装饰器，使用了functools的update_wrapper方法

implements_bool装饰器功能又是什么呢？

## LocalProxy
相比Local对象，似乎更底层？LocalStack调用到了它

重定向所有操作到一个被代理的对象。

如果想要一个可以通过一个函数查找到的对象，就用LocalProxy注册

初始化的时候设置了一个判断，如果local不是一个Local或者LocalManager对象，那就说明传递进来的是一个被装饰的callable类

这里判断你是不是LocalManager似乎是通过__release_local属性来判断的。

__slots__里的这些对象在哪里初始化的呢？

在flask gloabls里传递进LocalProxy之前调用了functools的partial方法（偏函数）

只是从前向后，将已知参数固定为默认？但是必须设置的时候不具有默认参数

哦，只是在查找请求上下文时，让其一定已经拥有了request属性

调用request和Session时，就会去请求上下文和应用上下文栈的栈顶

所以这里这个代理的作用就只是封装一层？


request和session这两个LocalProxy还是不理解

然后flask_login的current_user就是将_get_user()这个对象进行了LocalProxy

## Request对象将environ的WSGI环境继续了一个只读封装

Request.from_values可以从url字符串生成一个request对象

werkzeug可以进行单元测试，存在一个虚拟对象



WSGI协议

哇，Werkzeug监听文件变化然后自动重载，原来是用了socker做的？


## Reference
1. [Signals — Flask 1\.0\.2 documentation](http://flask.pocoo.org/docs/1.0/signals/)
2. [Flask Signals详解](https://www.jianshu.com/p/756ed0267f53)
3. [Werkzeug(Flask)之Local、LocalStack和LocalProxy](https://www.jianshu.com/p/3f38b777a621)
4. [Werkzeug 文档](https://werkzeug-docs-cn.readthedocs.io/zh_CN/latest)


greenlet协程 概念，线程内资源无法隔离

为什么一定要用proxy，而不能直接调用Local或LocalStack对象呢？这主要是在有多个可供调用的对象的时候会出现问题，

直接使用LocalStack对象，user一旦赋值就无法再动态更新了，而使用Proxy，每次调用操作符(这里[]操作符用于获取属性)，都会重新获取user，从而实现了动态更新user的效果。

_load_user方法里，有获取匿名用户和 登录用户的部分

其中有根思Session_protect来判断的部分，

flask_login除了支持从Cookie读取用户，还支持从Request和header读取用户

只是从request读取用户，需要开发者自己用request_loader装饰出一个读取的回调函数，不太能想象用户数据怎么存储在request里，难道是根据不同的request请求头设置权限？

当前的测试，看来还是需要一个对用户的确认？暂时还没用到用户类呢

_load_user还是从session中获取到了user id，这是不是说明没能成功Expire掉？

的确，是不能打开remember,因为remember就是为了在session不存在了的情况下，能够实现自动登录而使用的。



----


flask socketio使用

mmp,因为没装eventlet,就一直运行不起来这个服务器，tmd，你倒是早说啊，依赖里都没提到。

终于正常了，卧槽，Flask-socketio的库的作者今天刚好改了东西，导致了这个错误。mmp，我今天怎么这么倒霉

B站的websocket，那份代码估计是对Bilibili前端代码进行了解密？

可以看一下是怎么做的。