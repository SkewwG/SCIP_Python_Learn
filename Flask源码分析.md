## 1. WSGI，全称 Web Server Gateway Interface

是为 Python 语言定义的 Web 服务器和 Web 应用程序或框架之间的一种简单而通用的接口

Gateway，也就是网关。网关的作用就是在协议之间进行转换。

WSGI可以起到一个接口的功能，前面对接服务器，后面对接app的具体功能
WSGI 中有一个非常重要的概念：每个 python web 应用都是一个可调用（callable）的对象。在 flask 中，这个对象就是 `app = Flask(__name__)` 创建出来的 `app`，就是下图中的绿色 Application 部分。要运行 web 应用，必须有 web server，比如我们熟悉的 apache、nginx ，或者 python 中的 [gunicorn](http://gunicorn.org/) ，我们下面要讲到的 `werkzeug` 提供的 `WSGIServer`，它们是下图的黄色 Server 部分。

**Server 和 Application 之间怎么通信，就是 WSGI 的功能**。它规定了 `app(environ, start_response)` 的接口，server 会调用 application，并传给它两个参数：`environ` 包含了请求的所有信息，`start_response` 是 application 处理完之后需要调用的函数，参数是状态码、响应头部还有错误信息。

## 2. werkzeug和jinja

### 2.1 werkzeug

Werkzeug是一个WSGI工具包，它可以作为web框架的底层库。负责核心的逻辑模块，比如路由、请求和应答的封装、WSGI 相关的函数等；

### 2.2 jinja

负责模板的渲染，主要用来渲染返回给用户的 html 文件内容。

### 2.3 如何理解wsgi, Werkzeug, flask之间的关系
Flask是一个基于Python开发并且依赖jinja2模板和Werkzeug WSGI服务的一个微型框架，对于Werkzeug，它只是工具包，其用于接收http请求并对请求进行预处理，然后触发Flask框架;对于jinja2模板，它用来实现对模板的处理，将模板和数据进行渲染，将渲染后的字符串返回给用户浏览器。

### 2.4 Flask是什么
[Flask](https://link.zhihu.com/?target=https%3A//dormousehole.readthedocs.io/en/latest/design.html)永远不会包含数据库层，也不会有表单库或是这个方面的其它东西。Flask本身只是Werkzeug和Jinja2的之间的桥梁，前者实现一个合适的WSGI应用，后者处理模板。

---
# Flask 源码分析

## 1. 启动流程
```
def application(environ, start_response):               #一个符合wsgi协议的应用程序写法应该接受2个参数  
    start_response('200 OK', [('Content-Type', 'text/html')])  #environ为http的相关信息，如请求头等 start_response则是响应信息  
    return [b'<h1>Hello, web!</h1>']        #return出来是响应内容  
```
*   environ：一个包含所有HTTP请求信息的`dict`对象；

*   start_response：一个发送HTTP响应的函数。

官网的例子如下：
```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```

创建一个flask应用时，要创建一个Flask对象

过滤一下Flask的init方法：
```
class Flask:
    def __init__(self, package_name):

        self.package_name = package_name
        self.root_path = _get_package_path(self.package_name)

        self.view_functions = {}
        self.error_handlers = {}
        self.before_request_funcs = []
        self.after_request_funcs = []
        self.url_map = Map()
```

**view_functions中保存了视图函数(处理用户请求的函数，如上面的hello())，**

**error_handlers 这个字典用来保存所有的错误处理视图函数，字典的 key 是错误类型码**

**before_request_funcs 这个列表用来保存在请求被分派之前应当执行的函数**

**before_first_request_funcs 在接收到第一个请求的时候应当执行的函数。**

**after_request_funcs 这个列表中的函数在请求完成之后被调用，响应对象会被传给这些函数**

**self.url_map 这里设置了一个 url_map 属性，并把它设置为一个 Map 对象，用以保存URI到视图函数的映射，即保存app.route()这个装饰器的信息**

**执行app.run()，当gunicorn收到http请求，去调用app的时候，他实际上是用了Flask 的 __call__方法，这点非常重要！！！调用堆栈如下**
```
app.run()
    run_simple(host, port, self, **options)
        __call__(self, environ, start_response)
            wsgi_app(self, environ, start_response)
```
```
class Flask(_PackageBoundObject):        #Flask类  
    def __call__(self, environ, start_response):    #Flask实例的__call__方法  
        return self.wsgi_app(environ, start_response)  #调用了wsgi_app这个功能
```
**当http请求从server发送过来的时候，他会启动__call__功能，最终实际是调用了wsgi_app功能并传入environ和start_response。代码如下：**
在 `werkzeug.serving:WSGIRequestHandler` 的 `run_wsgi` 中有这么一段代码：
```
def execute(app):
    application_iter = app(environ, start_response)
    try:
        for data in application_iter:
            write(data)
        if not headers_sent:
            write(b'')
    finally:
        if hasattr(application_iter, 'close'):
            application_iter.close()
            application_iter = None
```
可以看到 `application_iter = app(environ, start_response)` 就是调用 `app` 实例，那么它就需要定义了 `__call__` 方法，我们找到 `flask.app：Flask` 对应的内容：
```
def __call__(self, environ, start_response):
    """Shortcut for :attr:`wsgi_app`."""
    return self.wsgi_app(environ, start_response)
```
调用了self.wsgi_app(environ, start_response)

**wsgi_app是flask核心**
```
def wsgi_app(self, environ, start_response):
    # 创建请求上下文，并把它压栈
    ctx = self.request_context(environ)
    ctx.push()
    error = None

    try:
        try:
            # 正确的请求处理路径，会通过路由找到对应的处理函数
            response = self.full_dispatch_request()
        except Exception as e:
            # 错误处理，默认是 InternalServerError 错误处理函数，客户端会看到服务器 500 异常
            error = e
            response = self.handle_exception(e)
        return response(environ, start_response)
    finally:
        if self.should_ignore_error(error):
            error = None
        # 不管处理是否发生异常，都需要把栈中的请求 pop 出来
        ctx.auto_pop(error)
```

`full_dsipatch_request` 的代码如下：
```
def full_dispatch_request(self):
    self.try_trigger_before_first_request_functions()
    try:
        request_started.send(self)
        rv = self.preprocess_request()
        if rv is None:
            rv = self.dispatch_request()
    except Exception as e:
        rv = self.handle_user_exception(e)
    return self.finalize_request(rv)
```
这段代码最核心的内容是 `dispatch_request`，`self.dispatch_request()` 返回的是处理函数的返回结果（比如 hello world 例子中返回的字符串），`finalize_request` 会把它转换成 `Response` 对象。
在 `dispatch_request` 之前我们看到 `preprocess_request`，之后看到 `finalize_request`，它们里面包括了请求处理之前和处理之后的很多 hooks 。这些 hooks 包括：

*   第一次请求处理之前的 hook 函数，通过 `before_first_request` 定义
*   每个请求处理之前的 hook 函数，通过 `before_request` 定义
*   每个请求正常处理之后的 hook 函数，通过 `after_request` 定义
*   不管请求是否异常都要执行的 `teardown_request` hook 函数

`dispatch_request` 要做的就是找到我们的处理函数，并返回调用的结果，也就是**路由的过程**。

## 2. 路由总结：
### rule， endpoint， view_func是怎么互相建立联系的？
### 注册应用 url 对应的处理函数

```
@app.route('/')
def hello():
    return "hello, world!"
等价于
def hello():
    return "hello, world!"

app.add_url_rule('/', 'hello', hello)
```


分析add_url_rule('/', 'hello', hello)
在add_url_rule方法里有这么一句

```
self.view_functions[endpoint] = view_func
```

**view_functions是一个字典，这样就相当于endpoint和view_func建立了连接**

**现在endpoint， view_func可以联系起来了，那么rule和endpoint该怎么连接呢**

**通过MapAdpter的match方法，将path_info和endpoint相连接**

**这样就完成了根据一个网站的路径调用view_func的过程**


### 详细分析：↓

### 2.1 add_url_rule
```
def add_url_rule(self, rule, endpoint=None, view_func=None, **options):
    """Connects a URL rule.  Works exactly like the :meth:`route`
    decorator.  If a view_func is provided it will be registered with the
    endpoint.
    """

    methods = options.pop('methods', None)

    rule = self.url_rule_class(rule, methods=methods, **options)
    self.url_map.add(rule)
    if view_func is not None:
        old_func = self.view_functions.get(endpoint)
        if old_func is not None and old_func != view_func:
            raise AssertionError('View function mapping is overwriting an '
                                 'existing endpoint function: %s' % endpoint)
        self.view_functions[endpoint] = view_func

```

连接 URL 规则。 工作原理与: "route" 装饰器完全一样。 如果提供了 view_func, 它将在endpoint上注册

#### 2.1.1 rule

url规则

#### 2.1.2 endpoint

**endpoint 注册 URL 规则。 Flask 本身假定view function 的名称当作endpoint**

#### 2.1.3 view_func

**要调用的函数**

#### 2.1.4 options

要转发到基础类的选项: '~werkzeug.routing.Rule' 对象。 对Werkzeug的更改是处理方法选项。 方法是将此规则限制为 ("GET"、"POST" ) 等的方法列表。

如果没有传递 view_func, 则需要将endpoint 连接到view_func, 如下：

        app.view_functions['index'] = index


#### 2.1.5 view_functions 是一个字典

**注册了所有视图函数的字典。 这些键将是函数名, 它们也用于生成 url, 而这些值是函数对象本身。要注册视图函数, 使用: "route" 修饰器。**

注册：↓

```
self.view_functions[endpoint] = view_func
```

**add_url_rule传递的第二个参数endpoint 是函数名，传递的第三个参数view_func是函数本身**

主要做的事情就是更新 `self.url_map` 和 `self.view_functions` 两个变量。

`url_map` 是 `werkzeug.routeing:Map` 类的对象            self.url_map = Map()

`rule` 是 `werkzeug.routing:Rule` 类的对象               url_rule_class = Rule


### 2.2. Class Map
```
m = Map([
    Rule('/', endpoint='index'),
    Rule('/downloads/', endpoint='downloads/index'),
    Rule('/downloads/<int:id>', endpoint='downloads/show')
])

urls = m.bind("example.com", "/")
ret1 = urls.match("/", "GET")
print(ret1)                             # ('index', {})
ret2 = urls.match("/downloads/42")      # 默认get
print(ret2)                             # ('downloads/show', {'id': 42})
ret3 = urls.match("/downloads/")        
print(ret3)                             # ('downloads/index', {})
ret4 = urls.match("/downloads")         # 报错
print(ret4)                             # RequestRedirect: 301 Moved Permanently: None
ret5 = urls.match("/missing")           # 报错
print(ret5)                             # 404 Not Found: The requested URL was not found on the server.
```

map 类存储所有 URL 规则和一些配置参数。 某些配置值仅存储在 "映射" 实例上, 因为这些参数影响所有规则, 而另一部分则只是缺省值, 并且可以为每个规则重写。 请注意, 除了将 "规则" 作为关键字参数外, 还必须指定所有参数!

#### 2.2.1 rules

**Rule实例组成的序列**

#### 2.2.2 def bind

```
def bind(self, server_name, script_name=None, subdomain=None, url_scheme='http', default_method='GET', path_info=None, query_args=None):
```

**返回一个新的: 类: "MapAdapter"**

**"server_name" 必须被传递,传递网站的域名**

**"script_name" 将默认为 "'/", 如果没有进一步指定或 "无"。**

**如果 path_info 没有传递给: "match", 它将传递默认路径信息给bind**

#### 2.2.3 def match
```
def match(self, path):
```
**werkzeug 中是怎么实现 match 方法的？Map 保存了 Rule 列表，match 的时候会依次调用其中的 rule.match 方法，如果匹配就找到了**


### 2.3. Class MapAdapter

**Map实例调用了bind之后，返回了MapAdapter类，并且调用了MapAdapter的match方法**

```
class MapAdapter(object):
    def match(self, path_info=None, method=None, return_rule=False, query_args=None):
```

#### 2.3.1 传递的值
**传递匹配方法的path_info 和 method（默认GET）**

你会收到一个 "NotFound" 异常, 指示没有匹配到 URL。 "NotFound" 异常也是一个 WSGI 应用程序, 您可以调用它来获取默认页 "NotFound" 页

你会收到一个 "MethodNotAllowed" 异常, 指示此 URL 存在匹配项, 但不支持当前请求方法。这对于 RESTful 应用程序非常有用。

你会收到一个 "new_url" 属性的 "RequestRedirect" 异常。 如果请求 "/foo", 但正确的 URL 是 "/foo/" 可以使用 "RequestRedirect" 实例作为响应类似的对象类似于所有其他子类的 "HTTPException"。

#### 2.3.2 返回值
**如果存在匹配项，你可以获得一个元组(endpoint, arguments)。 如果 `return_rule`设置True, 在这个情况下，你获得的元组是(rule, arguments)**

**如果没有传递path info, 则使用默认路径信息 (如果未显式定义, 则默认为根 URL)。**

##### path_info

用于匹配路径信息。 重写绑定的路径信息。

##### method

用于匹配HTTP 方法。 重写指定的方法。

##### return_rule

返回匹配的规则, 而不是只是endpoint (默认为 "False")。

##### query_args

用于自动重定向为字符串或字典的可选查询参数。 当前无法使用查询参数进行 URL 匹配。

### 2.4. Class Rule
#### 2.4.1 其实是 url 到 endpoint 的转换：通过 url 找到处理该 url 的 endpoint

```
def __init__(self, string, defaults=None, subdomain=None, methods=None,
                 build_only=False, endpoint=None, strict_slashes=None,
                 redirect_to=None, alias=False, host=None):

Rule('/', endpoint='index'),
Rule('/downloads/', endpoint='downloads/index'),
Rule('/downloads/<int:id>', endpoint='downloads/show')
```

Rule表示一个 URL 模式。 有一些 "规则" 选项可以改变它的行为方式并传递给 "规则" 构造函数。请注意, 除了规则字符串之外, 所有参数 * 都必须是关键字参数, 以便在Werkzeug升级时不中断应用程序。

##### String

规则字符串基本上只是通常的** URL 路径**, 其占位符的格式为 "<converter(arguments):name>", 其中转换器和参数是可选的。 如果未定义转换器, 则使用 "默认" 转换器, 在正常配置中表示 "string".</converter(arguments):name>

以斜线结尾的 URL 规则是分支 url, 另一些则是叶。如果启用了** "strict_slashes" (默认值)**, 则在没有尾随斜线的情况下匹配的所有分支 url 都将触发重定向到与所追加的缺失斜线相同的 URL。

##### endpoint

规则的endpoint 。可以是任何。对函数、字符串、数字等的引用。 首选的方法是**使用字符串**, 因为endpoint 用于生成 URL。

##### defaults

具有相同endpoint的其他规则的默认可选字典是一个有点棘手, 但有用的, 如果你想有唯一的网址

```
url_map = Map([
                Rule('/all/', defaults={'page': 1}, endpoint='all_entries'),
                Rule('/all/page/<int:page>', endpoint='all_entries')
            ])
```

如果用户现在访问 "http://example.com/all/page/1", 他将被重定向到 "http://example.com/all/"。 如果 "Map" 实例上的 "redirect_defaults" 被禁用, 这只会影响 URL 生成。

##### subdomain

此规则的subdomain规则字符串。如果未指定规则, 则仅匹配映射的 "default_subdomain"。 如果map没有绑定subdomain， 此功能将被禁用。

如果您希望在不同的子域上拥有用户配置文件, 并且将所有子域转发到您的应用程序, 则会非常有用。

```
url_map = Map([
                Rule('/', subdomain='<username>', endpoint='user/homepage'),
                Rule('/stats', subdomain='<username>', endpoint='user/stats')
            ])
```

##### methods

此规则适用的 http 方法序列。 如果未指定, 则允许使用所有方法。例如, **如果希望 "POST" 和 "GET" 的endpoints不同**, 这可能很有用。 **如果定义了methods 并且路径匹配, 但匹配的methods 不在此列表中或该路径的另一个规则列表中, 则引发的错误是类型 "MethodNotAllowed", 而不是 "NotFound"。** 如果 "GET" 是存在的方法列表和 ' HEAD ' 不是, ' HEAD ' 是自动添加

## 3. 上下文（application context 和 request context）

**问：什么是上下文？**

答：每一段程序都有很多外部变量。只有像Add这种简单的函数才是没有外部变量的。一旦你的一段程序有了外部变量，这段程序就不完整，不能独立运行。你为了使他们运行，就要给所有的外部变量一个一个写一些值进去。这些值的集合就叫上下文。

来源：知乎 https://www.zhihu.com/question/26387327/answer/32611575

**问：请求上下文是如何、在何时被创建的呢？**

答：在服务器调用应用的时候，Flask 的 `wsgi_app` 中有这样的语句，就是创建了请求上下文并压栈。
```
def wsgi_app(self, environ, start_response): 
    ctx = self.request_context(environ) 
    ctx.push()
```

**flask 中有两种上下文：程序上下文`application context` 和 请求上下文`request context`。**

上下文有关的内容定义在 `globals.py` 文件
```
# context locals
_request_ctx_stack = LocalStack()
_app_ctx_stack = LocalStack()
current_app = LocalProxy(_find_app)
request = LocalProxy(partial(_lookup_req_object, 'request'))
session = LocalProxy(partial(_lookup_req_object, 'session'))
g = LocalProxy(partial(_lookup_app_object, 'g'))
```


* current_app ：当前激活的程序实例，比如helloworld例子中的app |
* g ：用作处理请求时的临时存储，每次请求都重设 |
* request ：请求对象，包含了客户端HTTP请求的内容，例如获取客户端请求报文头部中包含的User-Agent信息 request.headers.get("User-Agent") |
* session ：用户会话，字典格式，存储请求间需要记住的信息 |

在 flask 中，视图函数需要知道它执行情况的请求信息（请求的 url，参数，方法等）以及应用信息（应用中初始化的数据库等），才能够正确运行。把这些信息作为**类似全局变量的东西**，视图函数需要的时候，可以使用 **from flask import request** 获取。

`application context`演化出来两个变量 `current_app` 和 `g`;

`request context` 则演化出来 `request` 和 `session`。

**请求上下文**

```
from flask import request
@app.route('/useragent/')
def userAgent():
    user_agent = request.headers.get('User-Agent')
    return '<p>Your browser is %s</p>' % user_agent
```

**程序上下文**
```
from flask import current_app
# print('current_app.name:',current_app.name)
app_ctx = app.app_context()
app_ctx.push()
current_app.name
app_ctx.pop()
```

**问：看起来request像是一个全局变量，那么，一个全局变量为什么可以在一个多线程环境中随意使用呢?**

前面已经知道request由request context演化过来，而`request context` 就是一个 `LocalStack` 的实例：则先分析下_request_ctx_stack = LocalStack()。

由变量名可知，_request_ctx_stack是一个栈，跟进LocalStack类分析源码


```
class LocalStack(object):

    def __init__(self):
        self._local = Local()

    def push(self, obj):
        rv = getattr(self._local, 'stack', None)
        if rv is None:
            self._local.stack = rv = []
        rv.append(obj)
        return rv

    def pop(self):
        stack = getattr(self._local, 'stack', None)
        if stack is None:
            return None
        elif len(stack) == 1:
            release_local(self._local)
            return stack[-1]
        else:
            return stack.pop()
            
    @property
    def top(self):
        """The topmost item on the stack.  If the stack is empty,
        `None` is returned.
        """
        try:
            return self._local.stack[-1]
        except (AttributeError, IndexError):
            return None
```
可以看到类LocalStack定义了push，pop，top方法，符合一个栈的功能。

从__init__方法里看到，self._local = Local()，继续跟入类Local

```
class Local(object):
    __slots__ = ('__storage__', '__ident_func__')

    def __init__(self):
        # 数据保存在 __storage__ 中，后续访问都是对该属性的操作
        object.__setattr__(self, '__storage__', {})
        object.__setattr__(self, '__ident_func__', get_ident)

    def __call__(self, proxy):
        """Create a proxy for a name."""
        return LocalProxy(self, proxy)

    # 清空当前线程/协程保存的所有数据
    def __release_local__(self):
        self.__storage__.pop(self.__ident_func__(), None)

    # 下面三个方法实现了属性的访问、设置和删除。
    # 注意到，内部都调用 `self.__ident_func__` 获取当前线程或者协程的 id，然后再访问对应的内部字典。
    # 如果访问或者删除的属性不存在，会抛出 AttributeError。
    # 这样，外部用户看到的就是它在访问实例的属性，完全不知道字典或者多线程/协程切换的实现
    def __getattr__(self, name):
        try:
            return self.__storage__[self.__ident_func__()][name]
        except KeyError:
            raise AttributeError(name)

    def __setattr__(self, name, value):
        ident = self.__ident_func__()
        storage = self.__storage__
        try:
            storage[ident][name] = value
        except KeyError:
            storage[ident] = {name: value}

    def __delattr__(self, name):
        try:
            del self.__storage__[self.__ident_func__()][name]
        except KeyError:
            raise AttributeError(name)
```
Local类有两个成员变量，分别是`__storage__`和`__ident_func__`

`__storage__`是一个字典，最外面字典 key 是线程或者协程的 identity，value 是另外一个字典，这个内部字典就是用户自定义的 key-value 键值对。用户访问实例的属性，就变成了访问内部的字典，外面字典的 key 是自动关联的。

`__ident_func__`是一个函数。这个函数的含义是，获取当前线程的id（或协程的id）。

Local类还自定义了`__getattr__`和`__setattr__`这两个方法，也就是说，我们在操作`self.local.stack`时， 会调用`__setattr__`和`__getattr__`方法。

`LocalProxy` 是一个 `Local` 对象的代理，负责把所有对自己的操作转发给内部的 `Local`对象。

通过LocalStack和LocalProxy这样的Python魔法，每个线程访问当前请求中的数据(request, session)时， 都好像都在访问一个全局变量，但是，互相之间又互不影响。

## 4. 请求
对于 WSGI server 来说，请求又变成了文件流，它要读取其中的内容，把 HTTP 请求包含的各种信息保存到一个字典中，调用 WSGI app； 对于 flask app 来说，请求就是一个对象，当需要某些信息的时候，只需要读取该对象的属性或者方法就行了。

访问 flask 的请求对象非常简单，只需要 `from flask import request`：
```
from flask import request

with app.request_context(environ):
    assert request.method == 'POST'
```

requests继承自 `werkzeug.wrappers:Request`
```
from werkzeug.wrappers import Request as RequestBase
class Request(RequestBase):
    @property
    def max_content_length(self):
        """Read-only view of the ``MAX_CONTENT_LENGTH`` config key."""
        ctx = _request_ctx_stack.top
        if ctx is not None:
            return ctx.app.config['MAX_CONTENT_LENGTH']

    @property
    def endpoint(self):
        """The endpoint that matched the request.  This in combination with
        :attr:`view_args` can be used to reconstruct the same or a
        modified URL.  If an exception happened when matching, this will
        be ``None``.
        """
        if self.url_rule is not None:
            return self.url_rule.endpoint
```
添加了一些属性，这些属性和 flask 的逻辑有关

**`@property` 装饰符能够把类的方法变成属性，这是 python 中经常见到的用法**

既然requests继承自 `werkzeug.wrappers:Request`，跟入werkzeug.wrappers:Request 分析源码
```
class Request(BaseRequest, AcceptMixin, ETagRequestMixin,
              UserAgentMixin, AuthorizationMixin,
              CommonRequestDescriptorsMixin):

    """Full featured request object implementing the following mixins:

    - :class:`AcceptMixin` for accept header parsing
    - :class:`ETagRequestMixin` for etag and cache control handling
    - :class:`UserAgentMixin` for user agent introspection
    - :class:`AuthorizationMixin` for http auth handling
    - :class:`CommonRequestDescriptorsMixin` for common headers
    """
```
这里使用了混入编程。

继续跟入BaseRequest类

```
class BaseRequest(object):
    def __init__(self, environ, populate_request=True, shallow=False):
        self.environ = environ
        if populate_request and not shallow:
            self.environ['werkzeug.request'] = self
        self.shallow = shallow
```
就是简单的将environ变量保存下来。

接着看AcceptMixin这些类：
```
class AcceptMixin(object):

    @cached_property
    def accept_mimetypes(self):
        """List of mimetypes this client supports as
        :class:`~werkzeug.datastructures.MIMEAccept` object.
        """
        return parse_accept_header(self.environ.get('HTTP_ACCEPT'), MIMEAccept)

    @cached_property
    def accept_charsets(self):
        """List of charsets this client supports as
        :class:`~werkzeug.datastructures.CharsetAccept` object.
        """
        return parse_accept_header(self.environ.get('HTTP_ACCEPT_CHARSET'),
                                   CharsetAccept)
```
方法被 `@cached_property` 装饰

那么分析下 `@cached_property`源码
```
class cached_property(property):

    def __init__(self, func, name=None, doc=None):
        self.__name__ = name or func.__name__
        self.__module__ = func.__module__
        self.__doc__ = doc or func.__doc__
        self.func = func

    def __set__(self, obj, value):
        obj.__dict__[self.__name__] = value

    def __get__(self, obj, type=None):
        if obj is None:
            return self
        value = obj.__dict__.get(self.__name__, _missing)
        if value is _missing:
            value = self.func(obj)
            obj.__dict__[self.__name__] = value
        return value
```
这个装饰器实现了 `__set__` 和 `__get__` 方法，访问它装饰的属性，就会调用 `__get__` 方法，这个方法先在 `obj.__dict__` 中寻找是否已经存在对应的值。如果存在，就直接返回；如果不存在，调用底层的函数 `self.func`，并把得到的值保存起来，再返回。这也是它能实现缓存的原因：因为它会把函数的值作为属性保存到对象中。


## 5. response 响应
HTTP 响应分为三个部分： 状态栏（HTTP 版本、状态码和说明）、头部（以冒号隔开的字符对，用于各种控制和协商）、body（服务端返回的数据）

视图函数返回值即为响应

**响应的两种方法：**

1.  视图函数直接返回一个元组 (response, status, headers)

2.  视图函数返回一个make_resonse()函数产生的响应对象

第一个是返回的数据，第二个是状态码，第三个是头部字典
```
@app.route('/')
def hello_world():
    return 'Hello, World!', 201, {'X-Foo': 'bar'}
```



如果不想返回一个元组，Flask视图函数还可以返回Response对象。make_response() 
函数可接受一或多个参数（和视图函数的返回值一样），并返回一个 Response 对
象。
```
from flask import make_response
@app.route('/response/')
def response():
    resp = make_response('<h1>Bad Request</h1>',400)
    return resp
```


## 6. Session
**问：什么是session**

答：由于HTTP协议是无状态的协议，所以服务端需要记录用户的状态时，就需要用某种机制来识具体的用户，这个机制就是Session.典型的场景比如购物车，当你点击下单按钮时，由于HTTP协议无状态，所以并不知道是哪个用户操作的，所以服务端要为特定的用户创建了特定的Session，用用于标识这个用户，并且跟踪用户，这样才知道购物车里面有几本书。这个Session是保存在服务端的，有一个唯一标识。在服务端保存Session的方法很多，内存、数据库、文件都有。集群的时候也要考虑Session的转移，在大型的网站，一般会有专门的Session服务器集群，用来保存用户会话，这个时候 Session 信息都是放在内存的，使用一些缓存服务比如Memcached之类的来放 Session。

来源：知乎 https://www.zhihu.com/question/19786827/answer/28752144

**结论：**
1，session 在服务器端，cookie 在客户端（浏览器）
2，session 默认被存在在服务器的一个文件里（不是内存）
3，session 的运行依赖 session id，而 session id 是存在 cookie 中的，也就是说，如果浏览器禁用了 cookie ，同时 session 也会失效（但是可以通过其它方式实现，比如在 url 中传递 session_id）

在 flask 中使用 session 只要使用 `from flask import session` 导入这个变量，在代码中就能直接通过读写它和 session 交互。

flask 会在请求过来的时候自动解析 cookie 的值，把它变成 `session` 变量。在视图函数中可以使用它的值，也可以对它进行更新。最后再返回的 response 中，flask 也会自动把 session 写回到 cookie。

核心代码在save_session方法里：
```
def save_session(self, app, session, response):
        domain = self.get_cookie_domain(app)
        path = self.get_cookie_path(app)

        # 如果 session 变成了空字典，flask 会直接删除对应的 cookie
        if not session:
            if session.modified:
                response.delete_cookie(app.session_cookie_name,
                                       domain=domain, path=path)
            return

        # 是否需要设置 cookie。如果 session 发生了变化，就一定要更新 cookie，否则用户可以 `SESSION_REFRESH_EACH_REQUEST` 变量控制是否要设置 cookie
        if not self.should_set_cookie(app, session):
            return

        httponly = self.get_cookie_httponly(app)
        secure = self.get_cookie_secure(app)
        expires = self.get_expiration_time(app, session)
        val = self.get_signing_serializer(app).dumps(dict(session))
        response.set_cookie(app.session_cookie_name, val,
                            expires=expires,
                            httponly=httponly,
                            domain=domain, path=path, secure=secure)
```
从 `app` 和 `session` 变量中获取所有需要的信息，然后调用 `response.set_cookie` 设置最后的 `cookie`。这样客户端就能在 cookie 中保存 session 有关的信息，以后访问的时候再次发送给服务器端，以此来实现有状态的交互。


## 参考资料

https://segmentfault.com/a/1190000010954763

http://mingxinglai.com/cn/2016/08/flask-source-code/

https://juejin.im/post/5a32513ff265da430f321f3d

http://cizixs.com/2017/01/10/flask-insight-introduction