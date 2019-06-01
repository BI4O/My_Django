# Django之DRF的三等级视图封装

* ## 视图函数优化之一级视图APIView

  #### DRF(Django Rest Framework)也有自己封装好的视图父类APIView

  - #### 实际上也继承自django.View

  - #### 扩展了其他的功能,比如:认证,限流,权限等等

  * #### APIView之封装好的request对象（最大的特点）

    1. ##### 目的: 继承自APIView之后,能够通过request获取get,post提交的数据

    2. 获取数据方式:

       - request.query_params: 获取查询参数
       - request.data: 获取表单或者json提交的数据

    

  * #### DRF封装好的Response对象

    1. 目的: 能够使用Response函数返回, 指定的类型的数据, 状态码的设置
2. **rest_framework.Response()**对比原来**http.HttpResponse()/JosnResponse()**的好处:
       - 1, 可以自动根据前端Accept类型, 来返回对应的数据格式
     - 2, 可以通过常量, 来指定返回的状态码
   
    3. 代码格式:

       ~~~python
 from rest_framework.views import APIView
       from rest_framework.response import Response
       
       class BookAPIView(APIView):
       
         def get(self,request):
       		# 使用封装好的request对象	
             print(request.query_params)
       		
               # 使用封装好的Response方法而不是原来的http.JsonResponse/HttpResponse
               return Response("BookAPIView...get",status=status.HTTP_201_CREATED)
       
           def post(self,request):
       
               print(request.data)
       
               return Response("BookAPIView...post")
       ~~~
   
       

  * #### APIView之结合序列化器实现列表视图
  
    **列表视图**：指的是查询全部数据的视图
  
    **详情试图**：指的是查询单个或几个数据的视图

  使用封装好的request对象和Response函数，与序列化器一起结合使用，进行数据库交互

  在 views.py

  ~~~python
    class BookListAPIView(APIView):
      def get(self, request):
            #1,准备数据,假设传来的是查询全部数据
          books = BookInfo.objects.all()
            #2,转换数据
          serializer = BookInfoModelSerializer(instance=books,many=True)
            #4,返回响应
          return Response(serializer.data,status=status.HTTP_200_OK)
    
        def post(self,request):
            # 1,获取参数
            dict_data = request.data
            # 2,校验参数(省略)
            # 3,数据入库,转换数据
            serializer = BookInfoModelSerializer(data=dict_data)
            serializer.is_valid(raise_exception=True)
            serializer.save()
    
            # 4,返回响应, 创建成功返回201
            return Response(serializer.data,status=status.HTTP_201_CREATED)
  ~~~

  

* ## 视图函数优化之二级视图GenericAPIView

  * GenericAPIView除了保留了APIView的封装外，还新增了以下的类属性和类方法

    #### 实际就是抽取了增删改查方法中的序列化器和查询语句

    - 属性:
      - serializer_class: 提供了统一的,序列化器类
      - queryset:指定统一的查询集
      - lookup_field:指定'id'代替原来的pk参数
    - 方法:
      - get_serializer:获取序列化器类
      - get_queryset:获取查询集
      - get_object:在详情试图获取pk参数

  * 这样就可以更方便进行**列表视图**和**详情视图**的编写了

    ###### 列表视图实例

    ~~~python
    # url.py:  books/
    
    class BookInfoListGAView(GenericAPIView):
        # 抽取方法中的序列化器和查询数据
        serializer_class = BookInfoSerializer
        queryset = BookInfo.objects.all()
        # 查（全部数据）
        def get(self,request):
            # 1.准备数据
            # books = BookInfo.objects.all()  这句已经被抽取
            books = self.get_queryset()
            # 2.转换数据
            serializer = self.get_serializer(instance=books,many=True)        
            # 3.返回响应
            return Response(serializer.data, status=status.HTTP_200_OK)
        # 增（单个数据）
        def post(self,request):
            # 1.获取参数
            dict_data = request.data        
            # 2.校验参数（省略）
            # 3.转换数据，数据校验，数据入库
            serializer = self.get_serializer(data=dict_data)
            serializer.is_valid(raise_exception=True)
            serializer.save() # 智能选择create还是update        
            # 4.返回响应
            return Response(serializer.data,status.HTTP_201_CRRATED)
            
            
    ~~~

    ###### 详情视图实例

    ~~~python
    # urls.py:  books/?P<pk>\d+/
    
    class BookDetailGenericAPIView(GenericAPIView):
    	# 同样的抽取序列化器和查询数据
        serializer_class = BookInfoModelSerializer
        queryset = BookInfo.objects.all()
    	# 查
        def get(self,request,pk):
            #1,获取对象
            # book = BookInfo.objects.get(pk=pk)
            book = self.get_object()
            #2,创建序列化器
            serializer = self.get_serializer(instance=book)
            #3,返回响应
            return Response(serializer.data,status=status.HTTP_200_OK)
    	# 改
        def put(self,request,pk):
            # 1,获取参数,准备
            dict_data = request.data
            book = self.get_object()
            # 2,校验参数(省略)
            # 3,修改数据入库,转换数据
            serializer = self.get_serializer(instance=book,data=dict_data)
            serializer.is_valid(raise_exception=True)
            serializer.save()
            # 4,返回响应
            return Response(serializer.data,status=status.HTTP_200_OK)
    	# 删
        def delete(self,request,pk):
            self.get_object().delete()
            # 返回响应
            return Response(status=status.HTTP_204_NO_CONTENT)
    ~~~

    

* ## 视图函数优化之三级视图GenericAPIView+MiXin
  * #### MiXin

    MiXin封装了增删改查视图的基本视图行为（获取参数、校验参数、数据转换、数据入库）

    ##### 常见的MiXin类

      ```python
      名称                  提供方法         作用                 
      ListModelMixin       list           获取所有的数据(get)    
      CreateModelMixin     create         新建数据(post)      
      RetrieveModelMixin   retrieve       获取单个对象(get)     
      UpdateModelMixin     update         修改单个对象(put)     
      DestroyModelMixin    destroy        删除单个对象(delete)  
      ```

    ###### 结合二级视图应用于列表视图

    ~~~python
    class BookListMixinGenericAPIView(mixins.ListModelMixin, mixins.CreateModelMixin, GenericAPIView):
        serializer_class = BookInfoModelSerializer
        queryset = BookInfo.objects.all()
    
        def get(self, request):
            return self.list(request)
    
        def post(self, request):
            return self.create(request)
    ~~~

    ###### 结合二级视图应用于详情试图

    ~~~python
    class BookDetailMixinGenericAPIView(mixins.RetrieveModelMixin,
                                        mixins.UpdateModelMixin,
                                        mixins.DestroyModelMixin,
                                        GenericAPIView):
        serializer_class = BookInfoModelSerializer
        queryset = BookInfo.objects.all()
        lookup_field = "id"
    
        def get(self, request, id):
            return self.retrieve(request)
    
        def put(self, request, id):
            return self.update(request)
    
        def delete(self, request, id):
            return self.destroy(request)
    ~~~

    

  * #### 三级视图

    * 特点作用:

       1. 继承自了二级视图GenericAPIView,mixin
       2. 不同的三级视图, 具有了不同的功能 

    * ##### 常见的三级视图类

         ```python
         三级视图类              继承的父类                        方法      作用
         CreateAPIView    GenericAPIView，CreateModelMixin      post     创建数据
         ListAPIView      GenericAPIView，ListModelMixin        get      获取所有数据
         RetrieveAPIView  GenericAPIView，RetrieveModelMixin    get      获取单个数据
         DestroyAPIView   GenericAPIView，DestroyModelMixin     delete   删除单个数据
         UpdateAPIView    GenericAPIView，UpdateModelMixin      put      修改单个数据
         ```
    
       ###### 应用于列表视图
    
       ~~~python
   class BookListThreeView(CreateAPIView,ListAPIView):
           serializer_class = BookInfoModelSerializer
           queryset = BookInfo.objects.all()
       ~~~
    
       ###### 应用于详情试图
    
       ~~~python
       class BookDetailThreeView(RetrieveAPIView,UpdateAPIView,DestroyAPIView):
           serializer_class = BookInfoModelSerializer
           queryset = BookInfo.objects.all()
       ~~~
    

