# Django之RESTful设计风格

目的：理解前后端分离与不分离的区别

* #### 什么是DRF

  Django Restful framework：Django Restful框架

* #### Web应用开发模式

  * **前后端不分离**模式下，服务器做的事
    * 接受request
    * 加工(查询数据，渲染模板)
    * 返回response
  * **前后端分离**，服务器做的事
    * 接收ajax请求
    * 返回数据（json格式）

* #### REST设计风格/RESTful开发理念 的简介

  * ###### 采用的是前后端分离的模式
    
    写接口的时候要遵守restful规范
  * ##### restful常见的规范
    
    以接口名为api，项目名为example为例
    
    1. 接口域名
    
       - 尽量将api部署在专用域名下
         http://api.example.com
       - 如果api接口很简单，那就考虑放在主域名/项目名下
         http://example.org/api/
    
    2. 版本
    
       - 应该将api的版本放在url路由中
         http://www.example.com/app/1.0/api
         http://www.example.com/app/1.1/api
         http://www.example.com/app/1.2/api
    
    3. 路径
    
      - 不要包含动词，而是根据HTTP动词/请求方式`request.method`来判断该返回什么，详见下面的**RESTful风格接口分析**
      - api中的名词应该使用复数形式，无论请求的是单资源还是多资源
    
    4. HTTP动词
       对于资源的具体操作类型，叫做HTTP动词
    
       - GET(select)：从服务器中取出资源（一项或多项）
       - POST(create)：在服务器新建一个资源
       - PUT(update)：在服务器更新资源
       - DELETE(delete)：从服务器删除资源
    
       下面是一些例子
    
       > GET /zoos：列出所有动物园
       > POST /zoos：新建一个动物园（上传文件）
       > GET /zoos/ID：获取某个指定动物园的信息
       > PUT /zoos/ID：更新某个指定动物园的信息（提供该动物园的全部信息）
       > PATCH /zoos/ID：更新某个指定动物园的信息（提供该动物园的部分信息）
       > DELETE /zoos/ID：删除某个动物园
       > GET /zoos/ID/animals：列出某个指定动物园的所有动物
       > DELETE /zoos/ID/animals/ID：删除某个指定动物园的指定动物
    
    5. 过滤信息：Filter
    
    6. 状态码：Status Codes
    
    7. 返回结果
    
    8. 超媒体
    
       

* #### RESTful风格接口分析

  |     功能     | 请求方式 |    路由     |
  | :----------: | :------: | :---------: |
  | 查询全部书籍 |   GET    |   /books    |
  | 增加单本书籍 |   POST   |   /books    |
  | 查询单本书籍 |   GET    | /books/{id} |
  | 修改单本书籍 |   PUT    | /books/{id} |
  | 删除单本书籍 |  DELETE  | /books/{id} |

  * ##### 利用REST风格来进行接口设计类视图 
    
  在  **项目**/**应用pkg**/view.py  下
    
  1. 查询所有书籍
    
       ~~~python
       from django.view import View
       from django.http import JsonResponse
       
       # 1.列表视图查看所有书籍
       class BookInfo(View):
           def get(self,request):
       		# 获取参数  略
               # 校验参数  略
               # 查询数据，数据转换
               book_list = []
               for book in books:
                   book_dict = {
                       'id':book.id,
                       'btitle':book.title
                       ...
                   }
                   book_list.append(book_dict)
               # 返回响应(非字典型输入要设safe为False,状态码200表示查询成功)
               return JsonResponse(book_list, safe=False, status=200)
     ~~~
    
     在urls.py注册路由，用postman的GET方式访问  127.0.0.1:8000/books
    
  2. 创建书籍记录
    
       ~~~python
       from django.view import View
       from django.http import JsonResponse
       
       class BookInfo(View):
           def get(self,request):
       		return
           def post(self,request):
               # 获取参数：request.body是json数据，request.POST是表单数据
               request.POST.get('btitle')
               request.POST.get('bpub_date')
               request.POST.get('bread')
               request.POST.get('bcomment')
               # 校验参数  略
               # 数据入库，转换数据
               book = BookInfo.objects.create(btitle=btitle,
                                              bpub_date=bpub_date,
                                              bread=bread,
                                              bcomment=bcomment,
                                             )
               
             book_dict = {
                    'id':book.id,
                       'btitle':book.title
                       'bpub_date':book.bpub_date
                   	'bread':book.bread
                   	'bcomment':book.bcomment
                   }
               # 返回响应（字典型输入不用设safe参数，状态码201表示数据创建成功）
               return JsonResponse(book_dict,status=201)
       ~~~
       
       在urls.py注册路由，填好body表单数据，用postman的POST方式访问  127.0.0.1:8000/books
       
    3. 获取单本书书籍记录
    
       ~~~python
       from django.view import View
       from django.http import JsonResponse
       
       class BookInfo(View):
           def get(self,request):
       		return
           def post(self,request):
               return
       class BookDetail(View):
           def get(self, pk)
           # 获取参数 pk也就是id
           # 校验参数 pk的校验比较简单，已经在路由url()处校验完成
           # 查询数据，转换数据
           book = BookInfo.objects.get(pk=pk)
           # 转换成字典数据类型
           book_dict = {
               'id':book.id,
               'btitle':book.title
               'bpub_date':book.bpub_date
               'bread':book.bread
               'bcomment':book.bcomment
           }
           # 返回响应response
           return JsonResponse(book_dict,status=200)
       ~~~
    
       在urls.py注册路由，用postman的GET方式访问  127.0.0.1:8000/books/id
    
  * #### 总结

    在开发REST API接口时，我们在**视图**中需要做的最核心的事是：

    1. **将数据库数据序列化为前端所需要的格式，并返回；**

       具体来说：模型类对象  ----->  python字典  ----->  json数据形式 -----> 返回给前端使用

    2. **将前端发送的数据反序列化为模型类对象，并保存到数据库中。**

       具体来说：前端传送的数据  -------> 经过验证 -----> python的字典   用于输入  接受前端数据时使用

  * #### 思考

    在Django框架下开发REST API的视图中，**路由输入已经规范化**，虽然每个视图具体操作的数据不同，但增、删、改、查的实现流程基本套路化，所以**增删改查**这部分代码也是**规范化并封装**的（这也是催生DRF的原因）：

    **增**：校验请求数据 -> 执行反序列化过程 -> 保存数据库 -> 将保存的对象序列化并返回

    **删**：判断要删除的数据是否存在 -> 执行数据库删除

    **改**：判断要修改的数据是否存在 -> 校验请求的数据 -> 执行反序列化过程 -> 保存数据库 -> 将保存的对象序列化并返回

    **查**：查询数据库 -> 将数据序列化并返回

    

* #### DRF（Django_Restful_Framework）效果展示

  * ##### DRF的诞生缘由：

    在**Django**框架下开发**REST API**，虽然每个视图操作数据不同，但增删改查基本套路化，可以进行代码复用
    
  * ##### 具体来说DRF做了

    1. 对视图函数View进行了封装
    2. 对数据模型Model进行了封装
    3. 对路由也进行了封装

  * ##### 没有使用DRF的视图函数用法，类视图如下

    ###### 假设已经建好数据库和模型类，并已经数据库迁移

    在 **项目**/**应用pkg**/views.py  

    ~~~python
    class BookView(VIew)：
    	# 查
    	def get(self,request):
            # 省略100行代码 包括获取参数，校验参数，查询数据库，返回响应
            return
        
        # 增
        def post(self,request):
            # 省略100行代码 包括获取参数，校验参数，数据入库，返回响应
            return
        
        # 改
        def put(self,request):
            # 省略100行代码
            return
        
        # 删
        def delete(self,request):
            # 省略100行代码
    ~~~

    最后在  项目/项目同名pkg/url.py  和  项目/应用pkg/urls.py  中进行路由注册

  * ##### 使用DRF用法

    ###### 假设已经建好数据库和模型类，并已经数据库迁移

    在  **项目**/**应用pkg**/serializers.py

    ~~~python
    from rest_framework.viewsets import ModelViewSet  # 模型注册类
    from rest_framework import serializers # 导入序列器
    
    # 编写序列化类视图
    class BookInfoSerializer(serializer.MOdelSerializer):
        class Meta:
            model = Bookinfo        # 指的序列化内容来自这个模型类
            fields = '__all__'      # 指的序列化器包含该模型的所有字段
            
    # 编写视图注册类
    class BookViewSet(ModelViewSet):
        serializer_class = BookInfoSerializer
        queryset = BookInfo.objects.all()
  ~~~
    
  最后在  项目/项目同名pkg/url.py  和  项目/应用pkg/urls.py  中进行路由注册，完成类视图的编写，可通过浏览器请求（**前提遵守REST规范**），实现全部的数据库增删改查
    
    