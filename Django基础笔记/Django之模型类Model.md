# Django之模型类Model

- #### 模型类之数据库配置

  因为django默认的数据库采用的是**sqlite3**，所以要用**Mysql数据库**的话需要配置数据库

  - 在  **项目**/**项目同名pkg**/setting.py  中找到并修改

    ~~~python
    DATA_BASE = {
        'dafault':{
            'ENGINE': 'django.db.backends.mysql',
            'NAME':'数据库的名字',  # 数据库database必须提前在mysql那边创建好
            'USER':'链接数据的用户名',
            'PASSWORD':'该用户对应的登陆数据库密码'，
            'HOST':'localhost', # 数据库所在电脑的ip
            'PORT':'3306'  # 可以指定端口号
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
      bread = models.IntegerField(verbose='阅读量',dafault=o)
      bcomment = models.InteragerField(verbose_name='评论数',default=0)
      is_delete = models.BooleanField(verbose_name='逻辑删除'，default=False)
      
      # 重新命名表名，默认为 '应用名_模型类名'
      class Meta:
          # 就是要指定db_table这个变量，记住这个
          db_table = 'tb_books' 
      
      # 美化输出
      def __str__(self):
          self.btitle
          
  
          
  # 同理创建英雄类    
  class HeroInfo(models.Model):
      GENDER_CHOSE = (
      	(0, 'female'),
      	(1, 'male'),
      )
      hname = models.CharField(max_length=20, verbose_name='名称')
      hgender = models.SmallIntegerField(choices=GENDER_CHOICES, default=0, verbose_name='性别')
      hcomment = models.CharField(max_length=200, null=True, verbose_name='描述信息')
      #on_delete=models.CASCADE, 删除书籍的时候,将其下面的所有英雄也删除
      hbook = models.ForeignKey(BookInfo, on_delete=models.CASCADE, verbose_name='图书')  # 外键
      is_delete = models.BooleanField(default=False, verbose_name='逻辑删除')
  
      class Meta:
          db_table = 'tb_heros'
  
      #输出对象的时候,显示内容
      def __str__(self):
          return self.hname
  ~~~

  

- #### 模型类之数据库迁移

  目的是为了让**模型类**写好的**表结构**落实到mysql**数据库**中

  * 注意在迁移之前确保该应用已经注册到 **项目**/**项目同名pkg**/setting.py

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

  * **查询数据**，**以BookInfo模型类为例**
    在 **项目**/manager.py 下
    
* 基本查询
    
      ~~~python
      # 单记录方法一
      print(BookInfo.objects.get(id=1))
      # 方法二
      print(BookInfo.objects.get(pk=1))  # pk等价id
      # 方法三
      print(BookInfo.objects.filter(id=1).first())
      
      # 查询表的样本量
      print(BookInfo.objects.count())
      
      # 查询某个字段中包含某些关键字的样本(开头、结尾、非空)
      print(BookInfo.objects.filter(字段名_contains='keywork'))
      print(BookInfo.objects.filter(字段名_startwith='keywork'))
      print(BookInfo.objects.filter(字段名_endwith='keywork'))
      
      # 查询非空的样本
      print(BookInfo.objects.filter(字段名_isnull=False))
      
      # 查询编号为1，3，5的样本
      print(BookInfo.objects.filter(id_in=[1, 3, 5]))
  
      # 查询编号大于3的样本，gt是grater than的简写，同理小于用lt
      print(BookInfo.objects.filter(id_gt=3))
      # 查询大于等于3，同理改用gte:grater than equal,同理小于等于用lte
      print(BookInfo.objects.filter(id_gte=3))
      # 如果该字段类型是日期即DateField，同样可以比较大小来查询
      print(BookInfo.objects.filter(date_gt=3))
      
      # 查询id不等于3的样本
      print(BookInfo.objects.exclude(id=3))
      ~~~
    
    * F/Q/Sum/Max/order_by 查询（高级查询）
    
      ```python
      from django.db.models import F,Q,Sum
      
      """
      F 用于字段名转化为对应的数字，只用于包裹类型为数字的字段
      Q 用于实现查询中的and/or/not逻辑，也是只能包裹类型为数字的字段
      Sum  顾名思义用于查询某个字段的全部记录的和，也是只能包裹类型为数字的字段
      Max 查询某个数字型字段中全部记录中的最大值
      order_by 按照某个数字型字段的大小来进行排序
      """
      
      # 用F进行高级查询
      # 两个字段类型都是整数型intager
      # 比如查询bread字段对应的记录大于等于bcomment字段对应的记录的样本
      print(BookInfo.objects.filter(bread_gte=F('bcomment')))
      
      # 查询 字段 > 2x字段  的记录
      print(BookInfo.objects.filter(bread_gte=2*F('bcomment')))
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
    
      ```python
      # 用Sum查询某个字段所有记录的和
      print(BookInfo.objects.aggregate(Sum('字段名')))
      ```
    
      ```python
      # 用Max查询某个字段中记录的最大值
      print(BookInfo.objects.aggregate(Max('字段名')))
      ```
    
      ```python
      # 升序排列
      print(BookInfo.objects.order_by('字段名'))
      # 降序排列（字段名前加负号）
      print(BookInfo.objects.order_by('-字段名'))
      ```
    
      
    
  * 修改数据
  
  * 删除数据
  
* **关联查询**

* **惰性、缓存、限制**