# Django之搭建DRF工程与创建序列化器

### DRF之使用准备

- ##### 需要先安装

  `pip install djangorestframework`

- ##### 注册DRF应用

  在  **项目**/**项目同名pkg**/setting.py

  ```python
  INSTALLED_APPS = [
      ...
      'booktest.apps.BooktestConfig'  # 别忘了要序列化的应用(以booktest为例)也是要先注册的
      'rest_framework'                  # 这里注册我们要用的DRF（本质也是个应用）
  ]
  ```

- 编写：视图、序列化器和路由

  

### DRF之序列化器Serializer介绍

- ##### 什么是序列化（在pure python中）

  把**变量**从**内存**中**变成可存储或传输数据**的过程称之为**序列化**，在Python中叫pickling，在其他语言中也被称之为serialization，marshalling，flattening等等，都是一个意思。

  序列化之后，就可以把序列化后的内容写入磁盘，或者通过网络传输到别的机器上。

  反过来，把变量内容从序列化的对象重新读到内存里称之为反序列化，即unpickling。

- ##### 什么是序列化器（在DjangoRestFramework框架中）

  集体来说：json字符串   和   模型类   之间的转换器

  简单来说：是结构型数据（如表格，模型类）和非结构型数据（如json,xml）的转换器

- ##### 在DRF中的两个作用

  - 序列化：    model 2 json          模型类对象转json字符串数据
  - 反序列化： json 2 model          json字符串数据转模型类对象



### DRF之使用序列化器Serializer进行序列化

- ##### 序列化器Serializer的定义

  1. 新建序列化文件  **项目**/**应用pkg**/serializer.py  ，以后序列器全部写在这个文件里面

  2. 定义类，继承自rest_framework.serializers.Serializer

  3. 定义**序列化规则**否则序列化器不懂如何转化**模型类数据**为**json数据**
     规则如下

     - 和模型类的字段名字一致
     - 和模型类的字段类型一致
     - 和模型类的字段限制信息一致

     - 参数 read_only：只读
     - 参数 label：暂时理解成字段的解析

     例如这样，在  **项目**/**应用pkg**/serializer.py 

     ```python
     from rest_framework import serializers
     
     class BookInfoSerializer(serializers.Serializer):
         # 定义你设计的BookInfo的序列化器的转化规则
         id = serializers.IntegerField(label='编号', read_only=True)
         btitle = serializers.CharField(max_length=20,label='书名')
         bpub_date = serializers.DateField(label="发布日期")
         bread = serializers.IntegerField(default=0,label="阅读量")
         bcomment = serializers.IntegerField(default=0,label="评论量")
         is_delete = serializers.BooleanField(default=False,label="逻辑删除")
     ```

- ##### 序列化器 Serializer 序列化模型类对象

  - 使用序列化器将一个**书籍对象**转成**json数据**

    从数据库中取出一个书籍样本，并把它序列化，看看序列化后产生了什么数据
    在  Pycharm的Termial下输入  `python manager.py shell`  进入Ipython交互环境

    ```python
    In [1]:from booktest.serializers import BookInfoSerializer
    In [2]:from booktest.models import BookInfo
    In [3]:# 查询模型类并赋给变量book
    In [4]:book = BookInfo.objects.get(pk=1)
    In [5]:# 创建序列化器实例，并把模型类数据喂给它，转化为json数据
    In [6]:serializser = BookInfoSerializer(instance=book)
    In [7]:print(serializser.data)  # 我们只想要json数据中的data
    Out [7]:'{'id':1, 'btitle':'射雕英雄传'...}'
    ```

  - 使用序列化器将多个书籍对象组成的**列表**转成**json数据**

    ```python
    In [1]:from booktest.serializers import BookInfoSerializer
    In [2]:from booktest.models import BookInfo
    In [3]:# 查询模型类并赋给变量books
    In [4]:books = BookInfo.objects.all()
    In [5]:# 创建序列化器实例2，并把模型类数据喂给它，转化为json数据,列表数据要设many参数为真
    In [6]:serializser2 = BookInfoSerializer(instance=books, many=True)
    In [7]:print(serializser2.data)  # 我们只想要json数据中的data
    Out [7]:[OrderDict([('id',1),('btitle','射雕英雄传')]),
             OrderDict([('id',2),('btitle',  '天龙八部')]),
            ...
            ]
    ```

- ##### 序列化器Serializer的关联对象嵌套序列化

  问题来了，序列化器怎么转换复杂的关联对象关系呢？

  比如用hero的序列化器怎么显示它属于哪个book呢？

  具体来说，怎么定义**模型类**的**外键**的**序列化规则**呢？

  在  **项目**/**应用pkg**/serializer.py 

  ```python
  from rest_framework import serializers
  
  class BookInfoSerializer(serializers.Serializer):
      # 折叠已经写好的书籍序列化器类
  
  class HeroInfoSerializer(serializers.Serializer):
      """英雄数据序列化器"""
      # 先写好除外键以外的其他字段的序列化规则
      GENDER_CHOICES = (
          (0, 'male'),
          (1, 'female')
      )
      id = serializers.IntegerField(label='ID', read_only=True)
      hname = serializers.CharField(label='名字', max_length=20)
      hgender = serializers.ChoiceField(choices=GENDER_CHOICES,
                                        label='性别', required=False)
      hcomment = serializers.CharField(label='描述信息', 
                                       max_length=200, 
                                       required=False, 
                                       allow_null=True)
      
      
      # 书写外键的序列化规则(四选其一即可，多写会报错)
      
      # 方法一 外键字段将不能用作反序列化
      hbook = serializers.PrimaryKeyRelatedField(label='图书', read_only=True)
      # ''{...,'hbook': 2}
      
      
  	# 方法二 外键字段将会被用作反序列化的参数校验
  	hbook = serializers.PrimaryKeyRelatedField(label='图书',
                                                 queryset=BookInfo.objects.all())
      # {...,'hbook': 2}
      
      
      # 方法三 直接返回关联的模型类的__str__方法的返回值,返回值取决于你怎么写这个方法
      hbook = serializers.StringRelatedField(label='图书')
      # {...,'hbook': '天龙八部'}
      
      
      # 方法四 直接返回这个模型类的序列化器类,相当于返回了整个序列化器的json内容了
      hbook = BookInfoSerializer()
      '''
      {..., 'hbook': OrderedDict([('id', 2), 
      							('btitle', '天龙八部'),
      							('date','1986-07-24'),
                                  ('bread', 36), ('bcomment', 40),
                                  ('image', None)])}
      '''
  ```

  注意many参数在序列化规则的中使用
  在一方学序列化多方的时候，需要在序列化规则处添加参数many=True

  

### DRF之使用序列化器Serializer进行反序列化

- ##### 序列化器Serializer  反序列化的校验

  前端传过来的json数据如果是乱写的，根本不符合序列化规则，这时候还把它转化为模型类，岂不是乱套？

  - ##### 全字段类型和参数简单校验

    同样在Termial输入 `python manager.py shell` 进入到 Ipython 交互环境下

    ~~~python
    In [1]:from booktest.serializers import BookInfoSerializer
    In [2]:from booktest.models import BookInfo
    In [3]:# 假设前端传来的数据是data(json形式数据要用dict=json.loads(json)转化成dict)
    In [4]:book_dict = {
        		'btitle':'aaa',
        		'bpub_date':'1990-1-1',
        		'bread':10
    		} 
    In [5]:# 创建序列化器实例，并把dict数据喂给它，转化为模型类数据
    In [6]:serializer = BookInfoSerializer(data=book_dict)
    In [7]:serializer.valid(raise_exception=True)  # 返回True或False
    Out [7]:True               # 表明字段类型是合法的，字段类型校验成功
    In [8]:print(serializer.data)
    Out [8]:{
        		'btitle':'aaa',
        		'bpub_date':'1990-1-1',
        		'bread':10
        		'bcomment':0           # 自动补全的字段
        		'is_delete':False      # 自动补全的字段
    		} # 能打印出来dict表明这个传过来的数据是合法的
    ~~~

  - ##### 单字段校验与多字段校验

    **单字段校验**：假如有个别字段检验在正则化处无法完成，那么需要借助函数/方法来完成校验
    **多字段校验**：假如两个字段应该符合某种关系才能创建成功，比如验证码图片与验证码数字必须一致，也要借助函数/方法来进行校验

    思路是在序列化器的定义类里面增加校验方法
    在  **项目**/**应用pkg**/serializer.py   中

    ~~~python
    from rest_framework import serializers
    
    class BookInfoSerializer(serializers.Serializer):
    	...
        
        # 单字段校验方法，方法取名必须为  validate_字段名,value为字段的对应值
        def validate_btitle(self, value):
            # 检验代码
            if value xxxx:
                raise serializers.ValidationError('检验错误信息')
            return value
        
        # 多字段校验方法，方法取名必须为  validate，value为字段对应的值的字段
        def validate(self, value):
            # 以bread和bcomment为例子
            if bread < bcomment:
                raise serializers.ValidationError('检验错误信息')
            return value
    ~~~

    

* ##### 序列化器Serializer  反序列化入库

  把所有字段对应的值都校验完了，就可以入库了

  ~~~python
  class BookInfoSerializer(serializers.Serializer):
      """图书数据序列化器"""
      ...
  	# 重写create方法，用于数据入库，在shell中使用save()会自动执行
      def create(self, validated_data):
          """新建"""
          # validated_data表示已经通过校验的值
          """
          book = BookInfo()
          book.btitle = xxx
          book.pub_date = xxx
          ...
          """
          # 以上代码都可以用下面的拆包表示一句完成
          book = BookInfo(**validated_data)
          
          return book
  	
      # 重写update方法，在
      def update(self, instance, validated_data):
          """更新，instance为要更新的对象实例"""
          instance.btitle = validated_data.get('btitle', instance.btitle)
          instance.bpub_date = validated_data.get('bpub_date', instance.bpub_date)
          instance.bread = validated_data.get('bread', instance.bread)
          instance.bcomment = validated_data.get('bcomment', instance.bcomment)
          return instance
  ~~~

  然后

  在Termial中输入`python manager.py shell`进入Ipython交互环境中进行尝试数据入库

  ~~~python
  In [1]:from booktest.serializers import BookInfoSerializer
  In [2]:from booktest.models import BookInfo
  In [3]:# 假设前端传来的数据是data(json形式数据要用dict=json.loads(json)转化成dict)
  In [4]:book_dict = {
      		'btitle':'aaa',
      		'bpub_date':'1990-1-1',
      		'bread':10
  		} 
  In [5]:# 创建序列化器实例，并把dict数据喂给它，转化为模型类数据
  In [6]:serializer = BookInfoSerializer(data=book_dict)
  In [7]:serializer.valid(raise_exception=True)  # 返回True或False
  Out [7]:True               # 表明字段类型是合法的，字段类型校验成功
  In [8]:serializer.save()
  Out [8]:<BookInfo:'aaa'>   # 显示什么取决于模型类的__str__方法返回值
  ~~~

  注意：

  1. 如果创建序列化器对象的时候，没有传递instance实例，则调用save()方法的时候，create()被调用
  2. 相反，如果传递了instance实例，则调用save()方法的时候，update()被调用。

  具体来说：

  创建序列化器实例的时候，只有data传进来：执行create()方法创建新纪录

  ~~~python
  serializer = BookInfoSerializer(data=data)
  ~~~

  创建序列化器实例的时候，有data和instance数据传进来：执行update()方法进行记录更新

  ~~~python
  serializer = BookInfoSerializer(instance=instance, data=data)
  ~~~

  