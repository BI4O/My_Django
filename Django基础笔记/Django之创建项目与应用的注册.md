## Django预习

#### 基本思想

* MVT与MVC
  * MVT是继承MVC思想的
    1. **M : Model 模型**，与数据库进行交互
    
    2. **V : view 视图**，接收请求，进行处理，与M\T进行交互，返回应答，相当于大脑
    
    3. **T : Template 模板**，产生html页面
    
       

#### 环境配置

1. 创建虚拟环境

   ~~~shell
   mkvirtualenv django11 -p python==3.5
   ~~~

2. 在这个虚拟环境安装django1.11.11

   ~~~shell
   workon django11
   pip install django==1.11.11
   ~~~

   

#### 项目开发

* **创建项目**
  
首先终端中cd到你想要创建项目的路径下，输入命令（以p11为例）
  
  ~~~shell
  django-admin startproject p11
~~~
  
cd进入p11文件夹可以看到
  
* manage.py			   项目的管理文件
  
  * **p11**						    与项目文件同名的pythonpackage
     * \_\_init\_\_.py		  说明这是一个python的包
     * settings.py		 项目的配置文件（使用什么数据库）
     * urls.py		        进行url路由的配置（找到对应的视图函数）
   * wsgi.py		       web服务器和Django交互的入口	
  
 * Django一个**项目**是由很多个**模块或者叫应用**组成的

    * **创建应用**
      在Pycharm的Terminal下输入命令（以app1为例）

      ~~~shell
      python manage.py startapp app1
      ~~~

   * 创建好的app1会成为**项目目录下**的一个python package,里面会有

     * \__init__.py
     * model.py    写和数据库相关的内容
     * views.py    接受请求，视图函数
     * test.py    些测试代码的文件
     * admin.py    网站后台管理相关
     * **migrate**    用于数据库迁移的pkg

   * **应用注册：** 建立应用与项目之间的关系
     在 **p11**/**p11**/setting.py 中找到并添加应用

     ``` python
     INSTALLED_APPS = （
     	"app1",    # 添加这个元素
     ）
     ```

     终端下运行项目文件manage.py

     ```shell
     python manager,py runserver
     ```

     也可以指定ip和端口来运行，不指定的话默认是端口8000
     
     ~~~shell
     python manager,py runserver 127.0.0.1:8888
     ~~~
     
     It worked! 表示注册成功