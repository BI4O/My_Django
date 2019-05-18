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

  Pycharm的Terminal下输入

  ~~~shell
  python manager.py createsuperuser
  ~~~

  需要自定新管理员名、邮箱、密码

  ~~~shell
  python manager.py runserver
  ~~~

  然后浏览器访问 ip/端口号/admin，输入刚刚的新管理员用户名和密码，但是你会发现你创建好的表/模型类并没有在这里显示，因为你还需要注册，告诉Django框架**根据注册的模型类**来生成对应的**表管理**界面

- 注册**模型类**

  在  **项目**/**应用pkg**/admin.py

  ~~~python
  # 记得导入模型类,以BookInfo为例
  from 应用.models import BookInfo
  
  admin.site.register(BookInfo)
  ~~~

  然后就可以在后台管理页面  ip/端口号/admin  看到注册的模型类

  

* ### 后台管理之自定义管理界面

  1. #### 自定义模型类显示的名称

     如果想让BookInfo 处显示成别的属性如书名，可以在模型类处加个魔法方法

     在  **项目**/**应用pkg**/views.py

     ~~~python
     class BookInfo(models.Model):
         book_name = CharField(max_length=20)
         def __str__(self):
             return self.book_name
     ~~~

  2. #### 自定义模型类显示更多的信息（如某些属性的信息）

     想要在管理员页面显示某个模型类的更详细的内容，需要自定义模型管理类，模型管理类告诉了Django在生成管理页面上面显示哪些内容

     在  **项目**/**应用pkg**/admin.py   输入以下

     ~~~python
     from django.contrib import admin
     from .models import BookInfo,HeroInfo
     
     
     # 第一步，自定义模型管理类
     # 命名就是模型类名+Admin别乱起
     class BookInfoAdmin(admin.ModelAdmin):
         """图书模型管理类，这里可以改写很多属性实现个性化定制"""
     
         
         # 第二步，重写模型类的属性
     	# 列表的元素必须是你已经在模型类定义的字段名的str形式
         list_display = ['id','book_name','pub_date']
       
     
     
     # 第三步，重新注册模型类
     # 模型类的注册也要改一改,因为改写了模型管理类的属性
     # admin,site.register(BookInfo) 这行原来的注册要删掉否则报错
     admin,site.register(BookInfo，BookInfoAdmin)
     ~~~

     事实上，还有很多的属性可以重写，详情见父类**ModelAdmin**的属性

     

     拓展：以上list_display的元素都是**模型类的属性(字段)**，那如果我想添加**加工过的模型类的属性(字段)**呢？

     在  **项目**/**应用pkg**/models.py   下，在**模型类**中添加**加工方法**和**方法说明**

     ~~~python
     # 这是某个已经写好的模型类
     class BookInfo(models.Model):
         # 前面的字段定义略
     		...        
             
         # 新建一个加工方法
         def bookneckname(self):
             # 加工代码...
             new_name = self.bname + 'haha'
             
             # 这里直接返回加工好的属性
             return new_name  
         
         
         # 不想在后台上显示这个属性叫MY_DATE,那就改写一下
     	# 格式：加工方法名.short_description = '自定义属性名'
         bookneckname.short_description = '书名别名'
         
         # 想让这个属性也能根据某些属性来排序
         # 格式：加工方法名.admin_order_field = '模型类的某个属性'
         bookneckname.admin_order_field = "bpub_date"
     ~~~

     最后把这个**方法名**放到    **项目**/**应用pkg**/admin.py 的   **模型管理类**  的   list_display 中了

     ~~~python
     class BookInfoAdmin(admin.ModelAdmin):
         """图书模型管理类，这里可以改写很多属性实现个性化定制"""
     
         list_display = ['id','book_name','pub_date'，'bookneckname']
     ~~~

     

  3. #### 增加右侧过滤栏

     在  **项目**/**应用pkg**/admin.py  在模型管理类中添加 list_filter  列表

     ~~~python
     class HeroInfoAdmin(admin.ModelAdmin):
         ...
         # 过滤列表的元素必须是该模型类定义过的属性或者方法，以名字的str格式传入
         list_filter = ['hbook', 'hgender']
     ~~~

     那么在管理页面就可以有以上元素的过滤器了

     

  4. #### 增加搜索框

     在  **项目**/**应用pkg**/admin.py  在模型管理类中添加 search_fields 列表

     ~~~python
     class HeroInfoAdmin(admin.ModelAdmin):
         ...
         search_fields = ['hname']
     ~~~

     只有添加过的属性才可以被搜索到

     

  5. #### 编辑页分组显示

     点击某行的id链接，进入编辑界面，当**属性/字段**数量太多的时候，可以设计成分组显示

     在  **项目**/**应用pkg**/admin.py  在模型管理类中添加 fieldsets 元组

     格式：

     fieldsets = (
         ('组1标题',{'fields':('字段1','字段2')}),
         ('组2标题',{'fields':('字段3','字段4')}),
     )

     ~~~python
     class BookInfoAdmin(admin.ModelAdmin):
         ...
         # fields = ['btitle', 'bpub_date']  把原来的注释了才行
         fieldsets = (
             ('基本信息', {'fields': ['btitle', 'bpub_date']}),
             ('高级信息', {
                 'fields': ['bread', 'bcomment'],
                 'classes': ('collapse',)  # 是否折叠显示
             })
         )
     ~~~

     

  6. #### 关联对象的显示

     在一对多的模型类关系中，可以显示关联对象的数据
     在  **项目**/**应用pkg**/models.py  的模型类中添加加工方法

     ~~~python
     # 在英雄中显示所属书籍
     class HeroInfo(models.Model):
         ...
         hbook = models.ForeignKey(
           BookInfo, on_delete=models.CASCADE, related_name="heros", verbose_name='图书')  # 外键
     
         #显示书籍阅读量
         def book_bread(self):
             return self.hbook.bread
     ~~~

     然后  项目/应用pkg/admin.py  的注册模型类中添加加工方法名book_read  到  list_display  列表中

     ~~~python
     class HeroInfoAdmin(admin.ModelAdmin):
     	list_display = ("id","hname","hbook","book_bread")
     ~~~

     同理  在模型类中   定义加工方法

     ~~~python
     # 在书籍中显示包含的英雄
     class BookInfo(models.Model):
         ...
         #配置英雄显示
         def my_heros(self):
             # return self.heros.all()
             return [hero.hname for hero in self.heros.all()]
     
         my_heros.short_description = "英雄"
     ~~~

     在  模型管理类中  添加加工方法

     ~~~python
     class BookInfoAdmin(admin.ModelAdmin)
     	list_display = ("id","btitle","bread","bcomment","my_date","my_heros")
     ~~~

     

  7. #### 站点信息调整

     想要在管理员界面自定义**应用**显示名
     先在  **项目**/**应用pkg**/admin.py   的末尾添加

     ~~~python
     admin.site.site_header = '项目名'      # 原来是 'Django管理'
     admin.site.index_title = '应用名'      # 原来是 '站点管理'
     admin.site.site_title = '后台管理系统'  # 原来是 'Django站点管理员'
     ~~~

     

  8. #### 给记录添加图片


     能给模型类上传图片，在后台管理可以显示

     1. 在  **项目**/**应用pkg**/models.py  的某个模型类中添加ImageField类型的字段

        ~~~python
        class BookInfo(models.Model):
            bimage = model.ImageField(ver_bose='图片'，
                                     null=True,
                                     blank=True,
                                     upload_to='文件夹名'
                                     )
        ~~~

        其中upload_to 表示要传到哪个文件夹

     2. 迁移（因为模型类被修改）

        - python manage.py makemigrations
        - python manage.py migrate

     3. 将字段添加到管理员显示的分组中

        ~~~python
        class BookInfoAdmin(admin.ModelAdmin):
            ...
            #fieldsets以组的形式表示编辑字段(和上面的fields互斥)
            fieldsets = (
                ("基础组", {
                    'fields': ('btitle', 'bpub_date',"bimage")
                }),
                ...
            )
        ~~~

     4. 设置图片上传的路径
        一般为了统一管理所有图片等静待资源，都会把它们储存在  **项目**/**static**  下

        那么必须告诉django你要把这些东西放在static下，也就是配置图片上传位置
        在  **项目**/**项目同名pkg**/setting.py  中添加如下

        ~~~python
        MEDIA_ROOT = os.path.join(BASE_DIR,"static")
        ~~~

     5. 最后在管理页面编辑处上传图片即可

        - 注意最终上传的文件目录是  **项目**/**static**/**应用名**
          表示在**static文件夹**中也要把静态资源以**应用文件夹**来分开存放