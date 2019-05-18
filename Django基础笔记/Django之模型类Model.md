# Django之模型类Model

- #### 模型类之数据库配置

  因为django默认的数据库采用的是**sqlite3**，所以要用**Mysql数据库**的话需要配置数据库

  - 在  **项目**/**项目同名pkg**/setting.py  中找到并修改

    ~~~python
    DATA_BASE = {
        'default':{
            'ENGINE': 'django.db.backends.mysql',
            'NAME':'数据库的名字',      # 数据库database必须提前在mysql那边创建好
            'USER':'root',            # 登录数据库的用户名 
            'PASSWORD':'mysql',      # 用户登录数据库的密码
            'HOST':'localhost',       # 数据库所在电脑的ip
            'PORT':3306               # 可以指定端口号
        }
    }
    ~~~

  - 然后在  **项目**/**项目同名pkg**/\__init__.py  下添加

    ```python
    import pymysql  # 没有的话要pip安装
    pymysql.install_as_MySQLdb()
    ```

  - 在**mysql**创建数据库

    `create database xxx`

  

- #### 模型类之创建模型类

  注意**应用**应该先在  项目/**项目同名pkg**/setting.py 的 INSTALL_APP 里面注册好
  在  **项目**/**应用pkg**/models.py  下

  ~~~python
  from django.db import models
  
  # 创建图书类
  class BookInfo(models.MOdel):
      btitle = models.CharField(max_length=20,verbose_name='书名')
      bpub_date = models.DateField(verbose_name='发布日期')
      bread = models.IntegerField(verbose_name='阅读量',default=o)
      bcomment = models.InteragerField(verbose_name='评论数',default=0)
      is_delete = models.BooleanField(verbose_name='逻辑删除',default=False)
      
      # 重新命名表名，默认为 '应用名_模型类名'
      class Meta:
          db_table = 'tb_books' 
          verbose_name = '图书'  # 在admin站点中显示的名称
          verbose_name_plural = verbose_name  # 显示的复数名称
      
      # 美化输出
      def __str__(self):
          return self.btitle    # 定义每个数据对象显示的信息
          
  
          
  # 同理创建英雄类    
  class HeroInfo(models.Model):
      GENDER_CHOSE = (
      	(0, 'female'),
      	(1, 'male'),
      )
      hname = models.CharField(max_length=20, verbose_name='名称')
      hgender = models.SmallIntegerField(choices=GENDER_CHOISE, default=0, verbose_name='性别')
      hcomment = models.CharField(max_length=200, null=True, verbose_name='描述信息')
      #on_delete=models.CASCADE, 删除书籍的时候,将其下面的所有英雄也删除
      hbook = models.ForeignKey(BookInfo, on_delete=models.CASCADE, verbose_name='图书')  # 外键
      is_delete = models.BooleanField(default=False, verbose_name='逻辑删除')
  
      class Meta:
          db_table = 'tb_heros'  # 指明数据库的表名
          verbose_name = '英雄'
          verbose_name_plural = verbose_name
  
      def __str__(self):
          return self.hname
  ~~~

- #### 模型类之注册注册应用

  目的是为了让**模型类**写好的**表结构**落实到mysql**数据库**中

  模型类

- #### 模型类之数据库迁移

  * 注意在迁移之前确保该应用已经注册到 **项目**/**项目同名pkg**/setting.py
注册应用语句如果不知道怎么写可以在  **项目**/**应用pkg**/apps.py  中查看
    
  ```python
    INSTALLED_APPS = [
        ...
      '应用名.apps.应用名(开头大写)Config'
    ]
    ```
    
  * 数据库迁移
  
    * 生成迁移文件在 **项目**/**应用pkg**/models.py
  
      `python manage.py makemigrations`
  
    * 执行迁移文件
  
      `python manager.py migrate`
  
    至此，创建好的模型类才会形成数据库中的表，但此时表内并无数据
  
  

* #### 模型类之添加数据

  * 在终端的**mysql交互环境**中，以123.sql数据为例子，需要该文件的绝对路径

    `source ~/123.sql`
    
    

* ###### 模型类之日志信息的查看（了解）

  1. 配置mysql的配置文件，放开日志

     `sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`  并放开68，69行

  2. 重启mysql

     `sudo service mysql restart`

  3. 监听

     `sudo tail -f /var/log/mysql/mysql.log`

  4. 准备好数据库：建库，迁移数据库

  5. 在Pycharm的Terminal下进行数据库的增删改查

     `python manager.py shell` 进入Ipython交互环境，进行增删改查

     

* #### 模型类之增删改查

  * **增加数据**
    在 **项目**/manage.py   下

    ```python
    # 方法一
    book = BookInfo(btitle='西游记'，bpub_date='1990-1-1')
    book.save()
    
    # 方法二（更方便，自动save）
    BookInfo.objects.create(btitle='红楼梦', bpub_date='1991-1-1')
    
    # 在创建英雄时候，注意指定外键
    HeroInfo.objects.create(hname='孙女悟空', hbook_id=1)
    ```

  * **普通基本查询数据**，以BookInfo模型类为例
    在 **项目**/manager.py 下
    
    * ##### 基本查询
      
      1. get(条件)    查单一结果
      2. all(条件)      查多个结果
      3. count(条件)      查符合该条件的样本数
      
      ~~~python
      # 单记录方法一
      print(BookInfo.objects.get(id=1))
      # 方法二
      print(BookInfo.objects.get(pk=1))  # pk等价id
      # 方法三
      print(BookInfo.objects.filter(id=1).first())
      
      # 查询表的样本量
      print(BookInfo.objects.count())
      ~~~
      
      
      
    * ##### 过滤查询
    
      基本格式：模型类名.objects.filter/exclude(字段名__比较运算符=值)
    
      filter(条件)   过滤出多个结果
    
      ~~~python
      # 查询某个字段中包含某些关键字的样本(开头、结尾、非空)
      print(BookInfo.objects.filter(字段名__contains='keywork'))
      print(BookInfo.objects.filter(字段名__startwith='keywork'))
      print(BookInfo.objects.filter(字段名__endwith='keywork'))
      
      
      # 查询非空的样本
      print(BookInfo.objects.filter(字段名__isnull=False))
      
      # 查询编号为1，3，5的样本
      print(BookInfo.objects.filter(id__in=[1, 3, 5]))
      
      # 查询编号大于3的样本，gt是grater than的简写，同理小于用lt
      print(BookInfo.objects.filter(id__gt=3))
      # 查询大于等于3，同理改用gte:grater than equal,同理小于等于用lte
      print(BookInfo.objects.filter(id__gte=3))
      # 如果该字段类型是日期即DateField，同样可以比较大小来查询
      print(BookInfo.objects.filter(date__gt=3))
      ~~~
    
      对于**日期类型的字段**的过滤查询有些特殊，还有year/month/day/hour/minute/second关键字可以用于过滤
    
      ~~~python
      # 等于查询
      print(BookInfo.objects.filter(bpub_date__year=1980))
      # 不等于查询
      print(BookInfo.objects.filter(bpub_date__gt=date(1980)))
      ~~~
    
      
    
    * ##### 不等于查询
    
      exclude(条件)   排除掉符合该条件的结果
    
      ~~~python
      print(BookInfo.objects.exclude(id=3))
      ~~~
    
      
    
    * ##### 多重过滤 
    
      filter(条件1, 条件2)                    查询符合条件1**且**符合条件2
    
      filter(条件1).filter(条件2)          查询符合条件1**且**符合条件2
    
      filter(条件1).exclude(条件2)     查询符合条件1**且**不符合条件2
  
    * ##### F/Q/Sum/Max/order_by 查询（高级查询）
    
      **高级filter过滤**
      
      ```python
      from django.db.models import F,Q,Sum
      
      """
      F 用于比较大小，只能包裹类型位数字的字段
      Q 用于实现查询中的and/or/not逻辑，也是只能包裹类型为数字的字段
      Sum  顾名思义用于查询某个字段的全部记录的和，也是只能包裹类型为数字的字段
      Max 查询某个数字型字段中全部记录中的最大值
      order_by 按照某个数字型字段的大小来进行排序
      """
      
      # 用F进行高级查询
      # 两个字段类型都是整数型intager
      # 比如查询bread字段对应的记录大于等于bcomment字段对应的记录的样本
      print(BookInfo.objects.filter(bread__gte=F('bcomment')))
      
      # 查询 字段 > 2x字段  的记录
    print(BookInfo.objects.filter(bread__gte=2*F('bcomment')))
      ```
      
      ```python
      # 用Q进行高级查询
      # 多条件筛选查询,如查询bread大于30且id小于3的样本
      print(BookInfo.objects.filter(bread_gt=20, id_lt=3))
      # 使用Q实现and逻辑：Q包裹后用 & 隔开
      print(BookInfo.objects.filter(Q(bread_gt=30) & Q(id_gt=3)))
      
      # 使用Q实现or逻辑：Q包裹后用 | 隔开
      print(BookInfo.objects.filter(Q(bread_gt=30) | Q(id_gt=3))
      
      # 使用Q实现not逻辑：Q前加波浪号 ~
    print(BookInfo.objects.filter(~Q(bread_gt=30))
      ```
      
      
      
    
    **用aggregate(Sum/Max('字段名'))查询**，返回的是一个字典，结合Sum/Max/Min使用
    
      ```python
      from django.db.models import Sum
      # 用Sum查询某个字段所有记录的和
    print(BookInfo.objects.aggregate(Sum('字段名')))
      ```
    
      ```python
      # 用Max查询某个字段中记录的最大值
      print(BookInfo.objects.aggregate(Max('字段名')))
      ```
    
      
    
    **order_by('字段名') 来对接过进行排序**
    
    ```python
    # 升序排列
    print(BookInfo.objects.order_by('字段名'))
    # 降序排列（字段名前加负号）
    print(BookInfo.objects.order_by('-字段名'))
    ```
    
  * ##### 关联查询
  
    一对应的类   到   多对应的类（注意多模型类名要全小写+__set）
  
    ```python
    # 查询id=3的书里面包含的所有英雄
    # 一模型类名.objects.get(条件).多模型类名_set.all()
    print(BookInfo.objects.get(id=3).heroinfo_set.all())
    print(BookInfo.objects.filter(heroinfo__字段=值))
    ```
  
    多对应的类   到   一对应的类
  
    ~~~python
    # 查询英雄列表里第一个英雄在哪本书
    # 多类模型名.objects.get(条件).关系类属性名
    print(HeroInfo.objects.get(id=1).hbook)
    print(HeroInfo.objects.filter(hbook__字段=值))
    ~~~
  
    
  
  * ##### 修改数据
  
    * 方法一，save()
  
      ~~~python
      hero = HeroInfo.objects.get(hname='猪八戒')
      hero.hname = '猪悟能'
      hero.save()
      ~~~
  
    * 方法二，update()  推荐用法，够简单
  
      ~~~python
      HeroInfo.objects.filter(hname='沙悟净').update(hname='沙僧')
      ~~~
  
      
  
  * ##### 删除数据
  
    ~~~python
    HeroInfo.objects.get(id=3).delete()
    HeroInfo.objects.filter(id=3).delete()
    ~~~
  
    
  
* **惰性、缓存、限制**

  * 惰性查询：只有用到了才去查询
    目的：知道惰性、缓存、限制、含义

    ```python
    res = 模型类名.objects.all()  # 执行这行代码的时候并没有查询
    print(res)  # 只有用到了时候才查询,也就是生成sql语句
    ```

  * 缓存：使用前面查询的结果

    ~~~python
    # 执行了上面的代码后，再次执行查询
    print(res)  # 这行代码也不会实际查询，而是返回以前的查询结果
    ~~~

  * 限制：使用切片的方式得到想要的结果
  
    ~~~python
    print(BookInfo.objects.all()[0:2])
    ~~~