## 1. WSGI��ȫ�� Web Server Gateway Interface

��Ϊ Python ���Զ���� Web �������� Web Ӧ�ó������֮���һ�ּ򵥶�ͨ�õĽӿ�

Gateway��Ҳ�������ء����ص����þ�����Э��֮�����ת����

WSGI������һ���ӿڵĹ��ܣ�ǰ��Խӷ�����������Խ�app�ľ��幦��
WSGI ����һ���ǳ���Ҫ�ĸ��ÿ�� python web Ӧ�ö���һ���ɵ��ã�callable���Ķ����� flask �У����������� `app = Flask(__name__)` ���������� `app`��������ͼ�е���ɫ Application ���֡�Ҫ���� web Ӧ�ã������� web server������������Ϥ�� apache��nginx ������ python �е� [gunicorn](http://gunicorn.org/) ����������Ҫ������ `werkzeug` �ṩ�� `WSGIServer`����������ͼ�Ļ�ɫ Server ���֡�

**Server �� Application ֮����ôͨ�ţ����� WSGI �Ĺ���**�����涨�� `app(environ, start_response)` �Ľӿڣ�server ����� application��������������������`environ` �����������������Ϣ��`start_response` �� application ������֮����Ҫ���õĺ�����������״̬�롢��Ӧͷ�����д�����Ϣ��

## 2. werkzeug��jinja

### 2.1 werkzeug

Werkzeug��һ��WSGI���߰�����������Ϊweb��ܵĵײ�⡣������ĵ��߼�ģ�飬����·�ɡ������Ӧ��ķ�װ��WSGI ��صĺ����ȣ�

### 2.2 jinja

����ģ�����Ⱦ����Ҫ������Ⱦ���ظ��û��� html �ļ����ݡ�

### 2.3 ������wsgi, Werkzeug, flask֮��Ĺ�ϵ
Flask��һ������Python������������jinja2ģ���Werkzeug WSGI�����һ��΢�Ϳ�ܣ�����Werkzeug����ֻ�ǹ��߰��������ڽ���http���󲢶��������Ԥ����Ȼ�󴥷�Flask���;����jinja2ģ�壬������ʵ�ֶ�ģ��Ĵ�����ģ������ݽ�����Ⱦ������Ⱦ����ַ������ظ��û��������

### 2.4 Flask��ʲô
[Flask](https://link.zhihu.com/?target=https%3A//dormousehole.readthedocs.io/en/latest/design.html)��Զ����������ݿ�㣬Ҳ�����б������������������������Flask����ֻ��Werkzeug��Jinja2��֮���������ǰ��ʵ��һ�����ʵ�WSGIӦ�ã����ߴ���ģ�塣

---
# Flask Դ�����

## 1. ��������
```
def application(environ, start_response):               #һ������wsgiЭ���Ӧ�ó���д��Ӧ�ý���2������  
    start_response('200 OK', [('Content-Type', 'text/html')])  #environΪhttp�������Ϣ��������ͷ�� start_response������Ӧ��Ϣ  
    return [b'<h1>Hello, web!</h1>']        #return��������Ӧ����  
```
*   environ��һ����������HTTP������Ϣ��`dict`����

*   start_response��һ������HTTP��Ӧ�ĺ�����

�������������£�
```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello World!"

if __name__ == "__main__":
    app.run()
```

����һ��flaskӦ��ʱ��Ҫ����һ��Flask����

����һ��Flask��init������
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

**view_functions�б�������ͼ����(�����û�����ĺ������������hello())��**

**error_handlers ����ֵ������������еĴ�������ͼ�������ֵ�� key �Ǵ���������**

**before_request_funcs ����б��������������󱻷���֮ǰӦ��ִ�еĺ���**

**before_first_request_funcs �ڽ��յ���һ�������ʱ��Ӧ��ִ�еĺ�����**

**after_request_funcs ����б��еĺ������������֮�󱻵��ã���Ӧ����ᱻ������Щ����**

**self.url_map ����������һ�� url_map ���ԣ�����������Ϊһ�� Map �������Ա���URI����ͼ������ӳ�䣬������app.route()���װ��������Ϣ**

**ִ��app.run()����gunicorn�յ�http����ȥ����app��ʱ����ʵ����������Flask �� __call__���������ǳ���Ҫ���������ö�ջ����**
```
app.run()
    run_simple(host, port, self, **options)
        __call__(self, environ, start_response)
            wsgi_app(self, environ, start_response)
```
```
class Flask(_PackageBoundObject):        #Flask��  
    def __call__(self, environ, start_response):    #Flaskʵ����__call__����  
        return self.wsgi_app(environ, start_response)  #������wsgi_app�������
```
**��http�����server���͹�����ʱ����������__call__���ܣ�����ʵ���ǵ�����wsgi_app���ܲ�����environ��start_response���������£�**
�� `werkzeug.serving:WSGIRequestHandler` �� `run_wsgi` ������ôһ�δ��룺
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
���Կ��� `application_iter = app(environ, start_response)` ���ǵ��� `app` ʵ������ô������Ҫ������ `__call__` �����������ҵ� `flask.app��Flask` ��Ӧ�����ݣ�
```
def __call__(self, environ, start_response):
    """Shortcut for :attr:`wsgi_app`."""
    return self.wsgi_app(environ, start_response)
```
������self.wsgi_app(environ, start_response)

**wsgi_app��flask����**
```
def wsgi_app(self, environ, start_response):
    # �������������ģ�������ѹջ
    ctx = self.request_context(environ)
    ctx.push()
    error = None

    try:
        try:
            # ��ȷ��������·������ͨ��·���ҵ���Ӧ�Ĵ�����
            response = self.full_dispatch_request()
        except Exception as e:
            # ������Ĭ���� InternalServerError �����������ͻ��˻ῴ�������� 500 �쳣
            error = e
            response = self.handle_exception(e)
        return response(environ, start_response)
    finally:
        if self.should_ignore_error(error):
            error = None
        # ���ܴ����Ƿ����쳣������Ҫ��ջ�е����� pop ����
        ctx.auto_pop(error)
```

`full_dsipatch_request` �Ĵ������£�
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
��δ�������ĵ������� `dispatch_request`��`self.dispatch_request()` ���ص��Ǵ������ķ��ؽ�������� hello world �����з��ص��ַ�������`finalize_request` �����ת���� `Response` ����
�� `dispatch_request` ֮ǰ���ǿ��� `preprocess_request`��֮�󿴵� `finalize_request`���������������������֮ǰ�ʹ���֮��ĺܶ� hooks ����Щ hooks ������

*   ��һ��������֮ǰ�� hook ������ͨ�� `before_first_request` ����
*   ÿ��������֮ǰ�� hook ������ͨ�� `before_request` ����
*   ÿ��������������֮��� hook ������ͨ�� `after_request` ����
*   ���������Ƿ��쳣��Ҫִ�е� `teardown_request` hook ����

`dispatch_request` Ҫ���ľ����ҵ����ǵĴ������������ص��õĽ����Ҳ����**·�ɵĹ���**��

## 2. ·���ܽ᣺
### rule�� endpoint�� view_func����ô���ཨ����ϵ�ģ�
### ע��Ӧ�� url ��Ӧ�Ĵ�����

```
@app.route('/')
def hello():
    return "hello, world!"
�ȼ���
def hello():
    return "hello, world!"

app.add_url_rule('/', 'hello', hello)
```


����add_url_rule('/', 'hello', hello)
��add_url_rule����������ôһ��

```
self.view_functions[endpoint] = view_func
```

**view_functions��һ���ֵ䣬�������൱��endpoint��view_func����������**

**����endpoint�� view_func������ϵ�����ˣ���ôrule��endpoint����ô������**

**ͨ��MapAdpter��match��������path_info��endpoint������**

**����������˸���һ����վ��·������view_func�Ĺ���**


### ��ϸ��������

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

���� URL ���� ����ԭ����: "route" װ������ȫһ���� ����ṩ�� view_func, ������endpoint��ע��

#### 2.1.1 rule

url����

#### 2.1.2 endpoint

**endpoint ע�� URL ���� Flask ����ٶ�view function �����Ƶ���endpoint**

#### 2.1.3 view_func

**Ҫ���õĺ���**

#### 2.1.4 options

Ҫת�����������ѡ��: '~werkzeug.routing.Rule' ���� ��Werkzeug�ĸ����Ǵ�����ѡ� �����ǽ��˹�������Ϊ ("GET"��"POST" ) �ȵķ����б�

���û�д��� view_func, ����Ҫ��endpoint ���ӵ�view_func, ���£�

        app.view_functions['index'] = index


#### 2.1.5 view_functions ��һ���ֵ�

**ע����������ͼ�������ֵ䡣 ��Щ�����Ǻ�����, ����Ҳ�������� url, ����Щֵ�Ǻ���������Ҫע����ͼ����, ʹ��: "route" ��������**

ע�᣺��

```
self.view_functions[endpoint] = view_func
```

**add_url_rule���ݵĵڶ�������endpoint �Ǻ����������ݵĵ���������view_func�Ǻ�������**

��Ҫ����������Ǹ��� `self.url_map` �� `self.view_functions` ����������

`url_map` �� `werkzeug.routeing:Map` ��Ķ���            self.url_map = Map()

`rule` �� `werkzeug.routing:Rule` ��Ķ���               url_rule_class = Rule


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
ret2 = urls.match("/downloads/42")      # Ĭ��get
print(ret2)                             # ('downloads/show', {'id': 42})
ret3 = urls.match("/downloads/")        
print(ret3)                             # ('downloads/index', {})
ret4 = urls.match("/downloads")         # ����
print(ret4)                             # RequestRedirect: 301 Moved Permanently: None
ret5 = urls.match("/missing")           # ����
print(ret5)                             # 404 Not Found: The requested URL was not found on the server.
```

map ��洢���� URL �����һЩ���ò����� ĳЩ����ֵ���洢�� "ӳ��" ʵ����, ��Ϊ��Щ����Ӱ�����й���, ����һ������ֻ��ȱʡֵ, ���ҿ���Ϊÿ��������д�� ��ע��, ���˽� "����" ��Ϊ�ؼ��ֲ�����, ������ָ�����в���!

#### 2.2.1 rules

**Ruleʵ����ɵ�����**

#### 2.2.2 def bind

```
def bind(self, server_name, script_name=None, subdomain=None, url_scheme='http', default_method='GET', path_info=None, query_args=None):
```

**����һ���µ�: ��: "MapAdapter"**

**"server_name" ���뱻����,������վ������**

**"script_name" ��Ĭ��Ϊ "'/", ���û�н�һ��ָ���� "��"��**

**��� path_info û�д��ݸ�: "match", ��������Ĭ��·����Ϣ��bind**

#### 2.2.3 def match
```
def match(self, path):
```
**werkzeug ������ôʵ�� match �����ģ�Map ������ Rule �б�match ��ʱ������ε������е� rule.match ���������ƥ����ҵ���**


### 2.3. Class MapAdapter

**Mapʵ��������bind֮�󣬷�����MapAdapter�࣬���ҵ�����MapAdapter��match����**

```
class MapAdapter(object):
    def match(self, path_info=None, method=None, return_rule=False, query_args=None):
```

#### 2.3.1 ���ݵ�ֵ
**����ƥ�䷽����path_info �� method��Ĭ��GET��**

����յ�һ�� "NotFound" �쳣, ָʾû��ƥ�䵽 URL�� "NotFound" �쳣Ҳ��һ�� WSGI Ӧ�ó���, �����Ե���������ȡĬ��ҳ "NotFound" ҳ

����յ�һ�� "MethodNotAllowed" �쳣, ָʾ�� URL ����ƥ����, ����֧�ֵ�ǰ���󷽷�������� RESTful Ӧ�ó���ǳ����á�

����յ�һ�� "new_url" ���Ե� "RequestRedirect" �쳣�� ������� "/foo", ����ȷ�� URL �� "/foo/" ����ʹ�� "RequestRedirect" ʵ����Ϊ��Ӧ���ƵĶ���������������������� "HTTPException"��

#### 2.3.2 ����ֵ
**�������ƥ�������Ի��һ��Ԫ��(endpoint, arguments)�� ��� `return_rule`����True, ���������£����õ�Ԫ����(rule, arguments)**

**���û�д���path info, ��ʹ��Ĭ��·����Ϣ (���δ��ʽ����, ��Ĭ��Ϊ�� URL)��**

##### path_info

����ƥ��·����Ϣ�� ��д�󶨵�·����Ϣ��

##### method

����ƥ��HTTP ������ ��дָ���ķ�����

##### return_rule

����ƥ��Ĺ���, ������ֻ��endpoint (Ĭ��Ϊ "False")��

##### query_args

�����Զ��ض���Ϊ�ַ������ֵ�Ŀ�ѡ��ѯ������ ��ǰ�޷�ʹ�ò�ѯ�������� URL ƥ�䡣

### 2.4. Class Rule
#### 2.4.1 ��ʵ�� url �� endpoint ��ת����ͨ�� url �ҵ������ url �� endpoint

```
def __init__(self, string, defaults=None, subdomain=None, methods=None,
                 build_only=False, endpoint=None, strict_slashes=None,
                 redirect_to=None, alias=False, host=None):

Rule('/', endpoint='index'),
Rule('/downloads/', endpoint='downloads/index'),
Rule('/downloads/<int:id>', endpoint='downloads/show')
```

Rule��ʾһ�� URL ģʽ�� ��һЩ "����" ѡ����Ըı�������Ϊ��ʽ�����ݸ� "����" ���캯������ע��, ���˹����ַ���֮��, ���в��� * �������ǹؼ��ֲ���, �Ա���Werkzeug����ʱ���ж�Ӧ�ó���

##### String

�����ַ���������ֻ��ͨ����** URL ·��**, ��ռλ���ĸ�ʽΪ "<converter(arguments):name>", ����ת�����Ͳ����ǿ�ѡ�ġ� ���δ����ת����, ��ʹ�� "Ĭ��" ת����, �����������б�ʾ "string".</converter(arguments):name>

��б�߽�β�� URL �����Ƿ�֧ url, ��һЩ����Ҷ�����������** "strict_slashes" (Ĭ��ֵ)**, ����û��β��б�ߵ������ƥ������з�֧ url ���������ض�������׷�ӵ�ȱʧб����ͬ�� URL��

##### endpoint

�����endpoint ���������κΡ��Ժ������ַ��������ֵȵ����á� ��ѡ�ķ�����**ʹ���ַ���**, ��Ϊendpoint �������� URL��

##### defaults

������ͬendpoint�����������Ĭ�Ͽ�ѡ�ֵ���һ���е㼬��, �����õ�, ���������Ψһ����ַ

```
url_map = Map([
                Rule('/all/', defaults={'page': 1}, endpoint='all_entries'),
                Rule('/all/page/<int:page>', endpoint='all_entries')
            ])
```

����û����ڷ��� "http://example.com/all/page/1", �������ض��� "http://example.com/all/"�� ��� "Map" ʵ���ϵ� "redirect_defaults" ������, ��ֻ��Ӱ�� URL ���ɡ�

##### subdomain

�˹����subdomain�����ַ��������δָ������, ���ƥ��ӳ��� "default_subdomain"�� ���mapû�а�subdomain�� �˹��ܽ������á�

�����ϣ���ڲ�ͬ��������ӵ���û������ļ�, ���ҽ���������ת��������Ӧ�ó���, ���ǳ����á�

```
url_map = Map([
                Rule('/', subdomain='<username>', endpoint='user/homepage'),
                Rule('/stats', subdomain='<username>', endpoint='user/stats')
            ])
```

##### methods

�˹������õ� http �������С� ���δָ��, ������ʹ�����з���������, **���ϣ�� "POST" �� "GET" ��endpoints��ͬ**, ����ܺ����á� **���������methods ����·��ƥ��, ��ƥ���methods ���ڴ��б��л��·������һ�������б���, �������Ĵ��������� "MethodNotAllowed", ������ "NotFound"��** ��� "GET" �Ǵ��ڵķ����б�� ' HEAD ' ����, ' HEAD ' ���Զ����

## 3. �����ģ�application context �� request context��

**�ʣ�ʲô�������ģ�**

��ÿһ�γ����кܶ��ⲿ������ֻ����Add���ּ򵥵ĺ�������û���ⲿ�����ġ�һ�����һ�γ��������ⲿ��������γ���Ͳ����������ܶ������С���Ϊ��ʹ�������У���Ҫ�����е��ⲿ����һ��һ��дһЩֵ��ȥ����Щֵ�ļ��Ͼͽ������ġ�

��Դ��֪�� https://www.zhihu.com/question/26387327/answer/32611575

**�ʣ���������������Ρ��ں�ʱ���������أ�**

���ڷ���������Ӧ�õ�ʱ��Flask �� `wsgi_app` ������������䣬���Ǵ��������������Ĳ�ѹջ��
```
def wsgi_app(self, environ, start_response): 
    ctx = self.request_context(environ) 
    ctx.push()
```

**flask �������������ģ�����������`application context` �� ����������`request context`��**

�������йص����ݶ����� `globals.py` �ļ�
```
# context locals
_request_ctx_stack = LocalStack()
_app_ctx_stack = LocalStack()
current_app = LocalProxy(_find_app)
request = LocalProxy(partial(_lookup_req_object, 'request'))
session = LocalProxy(partial(_lookup_req_object, 'session'))
g = LocalProxy(partial(_lookup_app_object, 'g'))
```


* current_app ����ǰ����ĳ���ʵ��������helloworld�����е�app |
* g ��������������ʱ����ʱ�洢��ÿ���������� |
* request ��������󣬰����˿ͻ���HTTP��������ݣ������ȡ�ͻ���������ͷ���а�����User-Agent��Ϣ request.headers.get("User-Agent") |
* session ���û��Ự���ֵ��ʽ���洢�������Ҫ��ס����Ϣ |

�� flask �У���ͼ������Ҫ֪����ִ�������������Ϣ������� url�������������ȣ��Լ�Ӧ����Ϣ��Ӧ���г�ʼ�������ݿ�ȣ������ܹ���ȷ���С�����Щ��Ϣ��Ϊ**����ȫ�ֱ����Ķ���**����ͼ������Ҫ��ʱ�򣬿���ʹ�� **from flask import request** ��ȡ��

`application context`�ݻ������������� `current_app` �� `g`;

`request context` ���ݻ����� `request` �� `session`��

**����������**

```
from flask import request
@app.route('/useragent/')
def userAgent():
    user_agent = request.headers.get('User-Agent')
    return '<p>Your browser is %s</p>' % user_agent
```

**����������**
```
from flask import current_app
# print('current_app.name:',current_app.name)
app_ctx = app.app_context()
app_ctx.push()
current_app.name
app_ctx.pop()
```

**�ʣ�������request����һ��ȫ�ֱ�������ô��һ��ȫ�ֱ���Ϊʲô������һ�����̻߳���������ʹ����?**

ǰ���Ѿ�֪��request��request context�ݻ���������`request context` ����һ�� `LocalStack` ��ʵ�������ȷ�����_request_ctx_stack = LocalStack()��

�ɱ�������֪��_request_ctx_stack��һ��ջ������LocalStack�����Դ��


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
���Կ�����LocalStack������push��pop��top����������һ��ջ�Ĺ��ܡ�

��__init__�����￴����self._local = Local()������������Local

```
class Local(object):
    __slots__ = ('__storage__', '__ident_func__')

    def __init__(self):
        # ���ݱ����� __storage__ �У��������ʶ��ǶԸ����ԵĲ���
        object.__setattr__(self, '__storage__', {})
        object.__setattr__(self, '__ident_func__', get_ident)

    def __call__(self, proxy):
        """Create a proxy for a name."""
        return LocalProxy(self, proxy)

    # ��յ�ǰ�߳�/Э�̱������������
    def __release_local__(self):
        self.__storage__.pop(self.__ident_func__(), None)

    # ������������ʵ�������Եķ��ʡ����ú�ɾ����
    # ע�⵽���ڲ������� `self.__ident_func__` ��ȡ��ǰ�̻߳���Э�̵� id��Ȼ���ٷ��ʶ�Ӧ���ڲ��ֵ䡣
    # ������ʻ���ɾ�������Բ����ڣ����׳� AttributeError��
    # �������ⲿ�û������ľ������ڷ���ʵ�������ԣ���ȫ��֪���ֵ���߶��߳�/Э���л���ʵ��
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
Local����������Ա�������ֱ���`__storage__`��`__ident_func__`

`__storage__`��һ���ֵ䣬�������ֵ� key ���̻߳���Э�̵� identity��value ������һ���ֵ䣬����ڲ��ֵ�����û��Զ���� key-value ��ֵ�ԡ��û�����ʵ�������ԣ��ͱ���˷����ڲ����ֵ䣬�����ֵ�� key ���Զ������ġ�

`__ident_func__`��һ����������������ĺ����ǣ���ȡ��ǰ�̵߳�id����Э�̵�id����

Local�໹�Զ�����`__getattr__`��`__setattr__`������������Ҳ����˵�������ڲ���`self.local.stack`ʱ�� �����`__setattr__`��`__getattr__`������

`LocalProxy` ��һ�� `Local` ����Ĵ�����������ж��Լ��Ĳ���ת�����ڲ��� `Local`����

ͨ��LocalStack��LocalProxy������Pythonħ����ÿ���̷߳��ʵ�ǰ�����е�����(request, session)ʱ�� �������ڷ���һ��ȫ�ֱ��������ǣ�����֮���ֻ���Ӱ�졣

## 4. ����
���� WSGI server ��˵�������ֱ�����ļ�������Ҫ��ȡ���е����ݣ��� HTTP ��������ĸ�����Ϣ���浽һ���ֵ��У����� WSGI app�� ���� flask app ��˵���������һ�����󣬵���ҪĳЩ��Ϣ��ʱ��ֻ��Ҫ��ȡ�ö�������Ի��߷��������ˡ�

���� flask ���������ǳ��򵥣�ֻ��Ҫ `from flask import request`��
```
from flask import request

with app.request_context(environ):
    assert request.method == 'POST'
```

requests�̳��� `werkzeug.wrappers:Request`
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
�����һЩ���ԣ���Щ���Ժ� flask ���߼��й�

**`@property` װ�η��ܹ�����ķ���������ԣ����� python �о����������÷�**

��Ȼrequests�̳��� `werkzeug.wrappers:Request`������werkzeug.wrappers:Request ����Դ��
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
����ʹ���˻����̡�

��������BaseRequest��

```
class BaseRequest(object):
    def __init__(self, environ, populate_request=True, shallow=False):
        self.environ = environ
        if populate_request and not shallow:
            self.environ['werkzeug.request'] = self
        self.shallow = shallow
```
���Ǽ򵥵Ľ�environ��������������

���ſ�AcceptMixin��Щ�ࣺ
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
������ `@cached_property` װ��

��ô������ `@cached_property`Դ��
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
���װ����ʵ���� `__set__` �� `__get__` ������������װ�ε����ԣ��ͻ���� `__get__` ����������������� `obj.__dict__` ��Ѱ���Ƿ��Ѿ����ڶ�Ӧ��ֵ��������ڣ���ֱ�ӷ��أ���������ڣ����õײ�ĺ��� `self.func`�����ѵõ���ֵ�����������ٷ��ء���Ҳ������ʵ�ֻ����ԭ����Ϊ����Ѻ�����ֵ��Ϊ���Ա��浽�����С�


## 5. response ��Ӧ
HTTP ��Ӧ��Ϊ�������֣� ״̬����HTTP �汾��״̬���˵������ͷ������ð�Ÿ������ַ��ԣ����ڸ��ֿ��ƺ�Э�̣���body������˷��ص����ݣ�

��ͼ��������ֵ��Ϊ��Ӧ

**��Ӧ�����ַ�����**

1.  ��ͼ����ֱ�ӷ���һ��Ԫ�� (response, status, headers)

2.  ��ͼ��������һ��make_resonse()������������Ӧ����

��һ���Ƿ��ص����ݣ��ڶ�����״̬�룬��������ͷ���ֵ�
```
@app.route('/')
def hello_world():
    return 'Hello, World!', 201, {'X-Foo': 'bar'}
```



������뷵��һ��Ԫ�飬Flask��ͼ���������Է���Response����make_response() 
�����ɽ���һ��������������ͼ�����ķ���ֵһ������������һ�� Response ��
��
```
from flask import make_response
@app.route('/response/')
def response():
    resp = make_response('<h1>Bad Request</h1>',400)
    return resp
```


## 6. Session
**�ʣ�ʲô��session**

������HTTPЭ������״̬��Э�飬���Է������Ҫ��¼�û���״̬ʱ������Ҫ��ĳ�ֻ�����ʶ������û���������ƾ���Session.���͵ĳ������繺�ﳵ���������µ���ťʱ������HTTPЭ����״̬�����Բ���֪�����ĸ��û������ģ����Է����ҪΪ�ض����û��������ض���Session�������ڱ�ʶ����û������Ҹ����û���������֪�����ﳵ�����м����顣���Session�Ǳ����ڷ���˵ģ���һ��Ψһ��ʶ���ڷ���˱���Session�ķ����ܶ࣬�ڴ桢���ݿ⡢�ļ����С���Ⱥ��ʱ��ҲҪ����Session��ת�ƣ��ڴ��͵���վ��һ�����ר�ŵ�Session��������Ⱥ�����������û��Ự�����ʱ�� Session ��Ϣ���Ƿ����ڴ�ģ�ʹ��һЩ����������Memcached֮������� Session��

��Դ��֪�� https://www.zhihu.com/question/19786827/answer/28752144

**���ۣ�**
1��session �ڷ������ˣ�cookie �ڿͻ��ˣ��������
2��session Ĭ�ϱ������ڷ�������һ���ļ�������ڴ棩
3��session ���������� session id���� session id �Ǵ��� cookie �еģ�Ҳ����˵���������������� cookie ��ͬʱ session Ҳ��ʧЧ�����ǿ���ͨ��������ʽʵ�֣������� url �д��� session_id��

�� flask ��ʹ�� session ֻҪʹ�� `from flask import session` ��������������ڴ����о���ֱ��ͨ����д���� session ������

flask �������������ʱ���Զ����� cookie ��ֵ��������� `session` ����������ͼ�����п���ʹ������ֵ��Ҳ���Զ������и��¡�����ٷ��ص� response �У�flask Ҳ���Զ��� session д�ص� cookie��

���Ĵ�����save_session�����
```
def save_session(self, app, session, response):
        domain = self.get_cookie_domain(app)
        path = self.get_cookie_path(app)

        # ��� session ����˿��ֵ䣬flask ��ֱ��ɾ����Ӧ�� cookie
        if not session:
            if session.modified:
                response.delete_cookie(app.session_cookie_name,
                                       domain=domain, path=path)
            return

        # �Ƿ���Ҫ���� cookie����� session �����˱仯����һ��Ҫ���� cookie�������û����� `SESSION_REFRESH_EACH_REQUEST` ���������Ƿ�Ҫ���� cookie
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
�� `app` �� `session` �����л�ȡ������Ҫ����Ϣ��Ȼ����� `response.set_cookie` �������� `cookie`�������ͻ��˾����� cookie �б��� session �йص���Ϣ���Ժ���ʵ�ʱ���ٴη��͸��������ˣ��Դ���ʵ����״̬�Ľ�����


## �ο�����

https://segmentfault.com/a/1190000010954763

http://mingxinglai.com/cn/2016/08/flask-source-code/

https://juejin.im/post/5a32513ff265da430f321f3d

http://cizixs.com/2017/01/10/flask-insight-introduction