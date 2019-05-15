# Django之ORM与后台管理

## orm框架

* Object 对象 - 类
* Relationship 关系 - 关系型数据库中的表
* Mapping 映射 - 建立python中的类与mysql中的表之间的关系

#### Django中生成表（设计，生成前迁移文件，执行迁移）

* 在**应用pkg下**的**models.py中**设计类

  ~~~python
  from django.db import models
  # 在迁移完成后，数据库中将生成名为"应用名_模型类名小写（这里是bookinfo）"的一个table
  class BookInfo(models.Model):
      # CharField说明的是一个字符串,生成一个叫book_name的字段
  	book_name = models.CharField(max_length=20)
      # DataField说明是一个日期类型，生成一个叫pub_data的字段
      pub_date = models.DataField()
      # 另外还有一个字段自动生成，是id
  ~~~

* **模型类**生成**表**

  1. 生成迁移文件

     ~~~shell
     python manager.py makemigrations
     ~~~

     迁移文件根据model.py中定义的类来生成

  2. 执行迁移，在数据库中生成对应的迁移文件就在**应用pkg**的**migration**中，最后用命令来执行迁移,这样数据库中就会生成该表

     ```shell
     python manager.py migrate
     ```

     Django中默认生成的是sqlite的数据表，需要先安装sqlite

     ```shell
     sudu apt-get install sqliteman
     ```

     安装好再执行迁移，然后打开这个数据库

     ~~~shell
     sqliteman
     ~~~

#### Django中通过模型类来操作数据库

- 首先进入这个项目的终端

  可以在pycharm的Terminal下输入
  也可以在终端中先workon，再输入

  ~~~shell
  python manager.py shell
  ~~~

  然后就可以在python环境下用**模型类**进行数据库的**增删改查**

  ~~~python
  from 应用名.models import BookInfo
  # 生成实例对象
  b = BookInfo()
  # 定义实例对象的字段1
  b.book_name = '天龙八部'
  from datetime import data
  # 定义实例对象的字段2
  b.pub_date = data(1990-1-1)
  # 在数据库中生成这个实例成为数据记录,再次提醒，字段id会自动生成
  b.save()
  
  # 查询数据库
  b2 = BookInfo.objects.get(id=1)
  # 查看类型
  b2
  # 查看它的字段
  b2.book_name
  # 更改它的信息,然后要执行save方法才能更改数据库
  b2.book_name = '天龙八部'
  b2.save()
  
  # 删除这个记录
  b2.delete()
  ~~~

#### 类与类之间的关系设计（外键的设计）

- 回到  **项目**/**应用pkg**/model.py

  ~~~python
  from django.db import models
  # 一类
  class BookInfo(models.Model):
  	book_name = models.CharField(max_length=20)
      pub_date = models.DataField()
  
  # 新增一个英雄模型类（表）
  # 多类
  class HeroInfo(models.Model):
      hero_name = models.CharField(max_length=20)
      hero_gender = models.BooleanField(default=False)
      # 关系属性hero_book建立图书类和英雄类之间的一对多关系
      # 在数据库中这个字段名叫hero_book_id
      hero_book = models.ForeignKey('BookInfo')
  ~~~

  因为更新（新增）了模型类，要再来一次迁移

  - 生成迁移文件（在  **应用pkg**/**migrations**/）

  ~~~shell
  python manager.py makemigrations
  ~~~

  - 执行迁移

  ~~~shell
  python manager.py migrate
  ~~~

  - 新增英雄实例
    在Pycharm的Terminal下输入

  ~~~python
  from 应用名.models import BookInfo，HeroInfo
  b = BookInfo()
  b.book_name = '天龙八部'
  from datetime import data
  b.pub_date = data(1990-1-1)
  b.save()
  
  # 创建英雄实例，实例赋值
  h = HeroInfo()
  h.hero_name = '段誉'
  # 绑定外键,赋值必须是与之关联的一个”实例对象“
  h.hero_book = b
  h.save()
  
  # 查看英雄信息
  h2 = HeroInfo.objects.get(id=1)
  h2.hero_name  # '段誉'
  
  # 多类查询绑定了的单类，及其信息
  h2.hero_book  # <BookInfo: BookInfo object>
  h2.hero_book.book_name  # '天龙八部'
  
  # 单类查询绑定了的多类
  # 用 单类实例.绑定多类名_set.all()
  b.heroinfo_set.all()  # [<HeroInfo: HeroInfo object>]
  ~~~



## 后台管理

Django提供后台管理表的功能，要使用后台管理，首先要做的几件事
同样的，在  **项目**/**项目同名pkg**/setting.py   下

- **语言**和**时区**本地化

  修改项目的配置文件  **项目**/**项目同名pkg**/setting.py

  ~~~python
  # 找到并修改
  LANGUAGE_CODE = 'zh-hans'  # 使用中文
  TIME_ZONE = 'Asia/Shanghai'  # 中国时区
  ~~~

- 创建**管理员**

  Pycharm的Terminal

  ~~~shell
  python manager.py createsuperuser
  ~~~

  需要自定新管理员名、邮箱、密码

  ~~~shell
  python manager.py runserver
  ~~~

  然后浏览器访问 ip/端口号/admin，输入刚刚的新管理员用户名和密码，但是你会发现你创建好的表/模型类并没有在这里显示，因为你还需要注册，告诉Django框架根据注册的模型类来生成对应的表管理界面

- 注册**模型类**

  在  **项目**/**应用pkg**/admin.py

  ~~~python
  # 记得导入模型类,以BookInfo为例
  from 应用.models import BookInfo
  
  admin.site.register(BookInfo)
  ~~~

  然后就可以在后台管理页面  ip/端口号/admin  看到注册的模型类

  如果想让BookInfo 处显示成别的属性如书名，可以在模型类处加个魔法方法

  ~~~python
  class BookInfo(models.Model):
      book_name = CharField(max_length=20)
      def __str__(self):
          return self.book_name
  ~~~

- 自定义管理页面（可选操作）

  想要在管理员页面显示某个模型类的更详细的内容，需要自定义模型管理类，模型管理类告诉了Django在生成管理页面上面显示哪些内容

  在  **项目**/**应用pkg**/admin.py

  ~~~python
  # 自定义模型管理类
  class BookInfoAdmin(admin.ModelAdmin):
      """图书模型管理类"""
      # 列表的元素必须是你已经在模型类定义的字段名的str形式
      list_display = ['id','book_name','pub_date']
      
  # 同时该模型类的注册也要改一改
  # admin,site.register(BookInfo) 这行原来的注册要删掉否则报错
  admin,site.register(BookInfo，BookInfoAdmin)
  ~~~