# Django之视图函数View与类视图

### 视图函数Views.py--定义和关联

与Manager和Template交互，负责给浏览器返回html内容

* 定义视图函数（函数名可以自己起）

  wo在  **项目**/**应用pkg**/views.py  

  ~~~python
  from django.http import HttpResponse
  
  # 定义视图函数,必须要有request输入,以index（名字自己起）为例，
  def index(request):
      # TODO 进行处理，与M和T进行交互
      return HttpResponse（'Hello World'）
  ~~~

  

* 进行url配置，建立url和视图的对应关系

  1. 在  **项目**/**应用pkg**/  下新建urls.py


  ~~~python
  from django.conf.urls import url
  from 应用名 import views
  
  # 创建一个列表，固定名字叫urlpatterns不要乱起(注意有s)
  urlpatterns = [
      # 通过url函数 把/index路由和视图函数index关联起来
      url(r'^index/$', views.index)  
      # r为正则匹配，注意要用严格匹配，^为首$为尾
  ]
  ~~~

  同时在  **项目**/**项目同名pkg**/urls.py  中找到并更新urlpatterns

  ~~~python
  urlpatterns = [
      url(r'^admin', include(admin.site.urls)),  # 配置项目
      url(r'^', include('应用名.urls'))  # 包含这个应用的url.py
  ]
  ~~~

  在Pycharm中的Terminal输入命令以启动服务器

  ~~~shell
  puthon manager.py runserver
  ~~~

  

* 浏览器访问  127.0.0.1:8000/**index**

  Django做的就是拿着**' index/ '**去正则匹配（**注意：Django会自动补全/斜杠**）

  1. 先去                    **项目**/**项目同名pkg**/urls.py 的 urlpatterns         中匹配
  2. 然后到                **项目**/**应用pkg**/urls.py  的 urlpatterns                中匹配
  3. 找到                    url(r'**^index/$**', views.index)                               匹配成功，生成request
  4. 找到                    **项目**/**应用pkg**/views.py  的  index()                    执行index函数
  5. 返回                    HttpResponse（'Hello Word'）
  6. 浏览器显示         Hello Word

  需要注意：在第一次在urls.py匹配r'^'  这时候的index/并没有削减内容就去往include的地址继续匹配，如果在第一次urls.py匹配时候，r'^in'也可以匹配成功，但是接下来会变成带着'dex/'进入下一个urls.py中匹配，鉴于这种削减匹配的机制，一般我们在**项目pkg**/urls 中对要转下一个urls.py 的匹配一般写作 r'^'， 即不削减内容。
  
  
  
* （了解）那如果这个视图函数有除了request以外的其他参数呢？

  ~~~python
  # 例如 项目/应用pkg/views.py 中有这样一个视图函数
  def func(request, name, age):
      return HttpResponse(f'我是{name},今年{age}了')
  ~~~

  那么这时候需要在  **项目**/**应用pkg**/urls.py  写更复杂的分组正则

  ~~~PYTHON
  urlpatterns = [
      url(r'^(?P<name>\w+)/(?P<age>\d+)/$', views.func)
  ]
  ~~~

  

* ##### 视图函数之请求对象request

  其实request对象本身就可以携带参数
  这些参数都可以用在视图函数中进行加工，最后返回response

  1. 通过request对象获得**查询参数**

     - request.GET                                  是一个字典
     - request.GET.get(key)                           获取request的单个key,value
     - request.GET.getlist(key)                       获取单个key，对应多个value,得到的是list列表

  2. 通过request对象获取form表单，**非查询参数**，常见请求方式有POST,PUT,DELETE,PATCH
     注意需要将bytes类型转换成dict格式，用到decode(), loads()

     * request.POST: 获取表单数据
     * request.body: 获取json数据

  3. 通过request获取**请求方式**

     * request.method

  4. 通过request获取**COOKIE**

     * request.COOKIE

  5. 通过request获取**登陆的用户**（没有登陆则等于匿名用户AnonymousUser）

     * request.user

       

* ##### 视图函数之响应对象response

  * 常见的响应对象

    * HttpResponse()  用于返回文本信息

      ~~~python
      from django.http import HttpResponse
      
      # 返回文本的视图函数
      def http_response(request):
          # 参数content指要返回的文本内容
          # 参数status指返回的响应码，不指定的话默认为200表示发送成功
          # 参数content指的是返回的内容类型说明，会在响应体处显示
          response = HttpResponse(
              content='这是HttpPesponse返回的文本信息',
          	status=404,
              content_type='text/html; charset=utf-8'
          )
          
          return response
      ~~~

    * JsonResponse()  用于返回json数据

      返回单个字典

      ~~~python
      from django.http import JsonResponse
      
      # 返回json数据的视图函数
      def json_response(request):
          data_dict = {'name':'老王', 'age':'23'}
          # 把写好的字典扔进给data，JsonResponse会把它自动转为json数据放到响应体
          response = JsonResponse(data=data_dict) 
          return response
      ~~~

      返回多个字典（要用列表把它们装起来）

      ~~~python
      from django.http import JsonResponse
      
      # 返回json数据的视图函数
      def json_response(request):
          data1 = {'name':'老王', 'age':'23'}
          data2 = {'name':'老李', 'age':'24'}
          data3 = {'name':'张三', 'age':'25'}
          data_list = [data1, data2, data3]
      	# safe参数默认为True，如果传入的数据为非字典形式的话，要设为False
          response = JsonResponse(data=data_list,safe=False) 
          return response
      ~~~

    * HttpResponseForbidden返回禁止访问的警告，状态码status默认为403

      ~~~python
      from django.http import HttpResponseForbidden
      def forbidden(request):
          return HttpResponseForbidden('禁止访问')
      ~~~

      

* ##### 视图函数之重定向redirect

  * 一般重定向redirect都会配合反解析reverse一起使用

    首先，要在 **项目**/**项目同名pkg**/urls.py  添加空间名参数spacename，名字随便起，尽量见名知意

    ~~~python
    urlpatterns = [
        url('^', include('应用名.urls', namespace='ns'))
    ]
    ~~~

    然后，在  **项目**/**应用pkg**/urls.py  添加代名参数name，名字随便起，尽量见名知意

    ~~~python
    urlpatterns = [
        url('^redirect$', views.index, name='idx')
    ]
    ~~~

    最后在路由函数中在reverse中传入 'namespace : name' 参数

    ~~~python
    from django.shortcuts import redirect
    def redirect1(request):
        # 如果是重定向到本应用的其他视图函数
        # response = redirect('本应用某视图函数的路由url')
        # 如果是重定向到其他应用的某个视图函数
        response = redirect(reverse('ns:idx'))
        return response
    ~~~

  * 还可以返回第三方网站

    ~~~python
    def redirect2(request):
        # response = redirect('www.baidu.com') 这样会重定向到127.0.0.1:8000/www.baidu.com
        response = redirect('http://www.baidu.com')
        return response
    ~~~

    

* ##### 视图函数之操作cookie

  可以记录用户的操作信息用response记录为cookie，下次就可以通过resquest取得以前该用户的cookie

  * 设置cookie，用response的set_cookie()方法

    ~~~python
    from datetime import datetime
    from django.http import HttpResponse
    
    def set_cookie(request):
        response = HttpResponse('欢迎光临玩具店')
        response.set_cookie('time',datetime.now())
        return response
    ~~~
  ~~~
  
  ~~~
  
* 取出cookie，用request.COOKIE的get()方法
  
    ~~~python
    def get_cookie(request):
        time = request.COOKIES.get()
        return HttpResponse('您上次的登录时间为%s' % time)
  ~~~
  
* ##### 视图函数之操作session

  这里复习一下session和cookie的区别
  cookie的发明为了是解决HTTP协议中缺少无状态缺陷的问题
  后面由于cookie的不安全性诞生的 session 也叫会话，是一种服务器端的机制

  * **cookie机制**：

    - **存储：**正统的cookie分发是通过扩展HTTP协议来实现的，**服务器**通过在HTTP的**响应头**即`response`的`Head`中加上一行特殊的指示以提示**浏览器**按照指示生成相应的cookie存储下来（关闭会话就会销毁，但你也可以设置有效时间更长，那就得存在客户端所在本地硬盘了，而且每种浏览器都有自己的储存格式，不会串用）
    - **使用：**而cookie的使用是由**浏览器**按照一定的原则在后台自动**发送给服务器**的。浏览器检查所有存储的cookie，如果某个cookie所声明的作用范围大于等于将要请求的资源所在的位置，则把该cookie附在请求资源的HTTP**请求头**即`request`的`Head`上发送给服务器，<u>这就是为什么淘宝上你登录了再打开新链接也显示已登录</u>

  * **session机制**

    - **存储：**服务器使用一种类似于散列表的结构（也可能就是使用散列表）来保存信息。如果**客户端请求** `request`不包含session id，则为此客户端创建一个session并且生成一个与此session相关联的session id，session id的值应该是一个既不会重复，又不容易被找到规律以仿造的字符串，这个session id将被在本次**响应**`response`中返回给**客户端/浏览器**保存（例如cookie）。<u>注意session存在服务器，而session id存在客户端/浏览器，就像你拿的银行卡只有卡号-id，而你有多少钱的信息是存在银行系统（服务器）的</u>
    - **使用：**客户端/**浏览器**发起一个**请求**`request`，服务器首先检查这个**请求**`request`里是否已包含了一个session id，如果已包含则说明以前已经为此**浏览器**创建过session，**服务器**就按照session id把这个session检索出来使用（检索不到，会新建一个）

    简单来说
    cookie机制采用的是在**客户端**保持状态的方案，
    而session机制采用的是在**服务器端**保持状态的方案。

  * 指定用redis来存储session id的话需要先安装拓展配置（默认用sqlite3）

    1. 在Pycharm的Terminal下输入 `pip install django-redis`
    2. 在 **项目**/**项目同名pkg**/setting.py  最下面添加这段

    ```python
    CACHES = {
        "default": {
            "BACKEND": "django_redis.cache.RedisCache",
            "LOCATION": "redis://127.0.0.1:6379/1",
            "OPTIONS": {
                "CLIENT_CLASS": "django_redis.client.DefaultClient",
            }
        }
    }
    SESSION_ENGINE = "django.contrib.sessions.backends.cache"
    SESSION_CACHE_ALIAS = "default"
    ```

  * 设置session

    在 **项目**/**应用pkg**/views.py  中

    ```python
    from django.http import HttpResponse
    
    def set_session(request):
        # 
        request.session['username'] = 'laowang'
        
        return HttpResponse('set_session')
    ```

    

# 类视图

视图可以不是一个函数，而是一个类

- #### 类视图之使用步骤

  1. 定义类，继承自view，里面的方法需要标准的请求方式（小写）来命名
     还是在 **项目**/**应用pkg**/views.py  下

     ```python
     from django.views import View
     from django.http import HttpResponse
     
     # 继承自View这个类视图
     class PersonView(View):
         
         def get(self,request):
             return HttpResponse('这是一个get请求')
         
         def post(self,request):
             return HttpResponse('这是一个post请求')
     ```

  2. 定义完类视图后同样需要编写路由（应用路由和项目路由，后者略）
     在应用路由  **项目**/**应用pkg**/urls.py  下

     ```python
     from django.conf.urls import url
     
     urlpatterns = {
         
         # 格式 r'^匹配路由/$',views.类名.as_view()  记得最后有（）
      	url(r'^person/$',views.PersonView.as_view())   
     }
     ```

- #### 类视图之优点（对比普通的视图函数）

  - 可读性更好
  - 可以使用面向对象的思想，通过继承来提高代码的复用率

- #### 类视图之增加装饰器

  - 目的是给类增加额外的功能

    1. 方法一：写好装饰器，然后去urls.py去稍作修改
       在  项目/应用pkg/views.py  (也就是在视图函数的开头写好装饰器，不推荐)

       ~~~python
       def user_login_data(view_func):
       	def wrapper(request,*args,**kwargs):
               
               print('给%s增加了额外的功能 % request.method')
               
               return view_func(request,*args,**kwargs)
           
           return wrapper
       ~~~

       记得要在  项目/应用pkg/urls.py 下注册

       ~~~python
       from django.conf.urls import url
       
       urlpatterns = {
           # 格式就是   view.类视图名.as_view()  变成 装饰器名(view.类视图名.as_view())
           url(r'^xxx$', 装饰器名(view.被装饰的类视图名.as_view()))
       }
       ~~~

    2. 方法二：写好装饰器，然后使用django提供的method_decorator
       
       在 **项目**/**应用pkg**/views.py  下（推荐使用,更加灵活）
       
       * 装饰器可以放在类的上面，但是要指定装饰哪个方法
         1. name='dispatch'  装饰全部方法
         2. name='get'  只装饰get方法
       * 装饰器也可以直接放在类里面的某个方法上面，表示只装饰该方法
       
       ```python
       from django.utils.decorators import method_decorator
       from django.views import View
       from django.http import HttpRespons
       
       # name='dispatch' 表示类里的全部方法都装饰
       # name='get' 表示类里面get方法被装饰
       @method_decorator(装饰器名, name='dispatch')
       class PersonView(View):
           
           def get(self,request):
               return HttpResponse('这是一个get请求')
           
           @method_decorator(装饰器名)
           def post(self,request):
               return HttpResponse('这是一个post请求')
       ```
  
- ### 类视图之中间件

  - 视图函数执行的时候，若是需要很多个装饰器（例如csrf保护），都要写个@在上面，一不小心还会漏写，岂不是又麻烦，又不美观？

  - 中间件MIDDLEWARE是  **项目**/**项目同名pkg**/setting.py  里的配置项，里面**每个元素**都代表着一个**装饰器**，这里写的装饰器是**整个项目**里面**所有的视图函数**都戴上了的装饰器（其中就有csrf保护装饰器）

  - 那么如果想把一个**自定义装饰器**应用到项目里面的**所有视图函数**上，就可以在这里添加元素

    1. 首先在

    在  **项目**/**项目同名pkg**/settings.py

    ~~~python
    MIDDLEWARE = [
        ...
        # 加上你写的装饰器的reference
        '应用名.中间件文件名.装饰器名',
    ]
    ~~~

    