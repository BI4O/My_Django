# Django之模板Templates

通常我们展示给用户的应该是html页面而不是字串，这时候就需要有一个容易来城防这些html文件了，它就是Templates

- 创建模板文件

  在    **项目**/    下新建**templates**普通文件夹

- 配置模板目录

  在    **项目**/**项目同名pkg**/setting.py  中找到

  ```python
  TENMLPATES = [
      {
          # 设置模板文件目录，是把项目的绝对路径BASE_DIR+templates
          'DIRS':[os.path.join(BASE_DIR, 'templates')]
      }
  ]
  ```

  其中   BASE_DIR = '…………/**项目**'   是项目的绝对路径，在startproject时候就已经在setting.py中自动生成了

- 在模板文件中新建多个文件夹，每个文件夹只放一个应用的html文件

- 使用模板文件

  - 加载模板文件：去模板目录下面获取html文件的内容，得到一个模板对象

  - 定义模板的上下文：向模板文件传递数据

  - 模板渲染：得到一个标准的html内容

    以**book应用**下的**index视图函数**为例

    - 先在  **项目**/**templates**/  下新建  **book**/index.html

    - 然后在  **项目**/**book应用pkg**/views.py 下

      ```python
      # 导入django加载模板用的加载器loder和上下文管理器RequestContext
      from django.template import loader,PequestContext
      
      def index(request):
          # return HttpResponse('Hello world')
          # 1.加载模板文件temp(写入该html文件在templates下的相对路径)
          temp = loader.get_template('book/index.html')
      	# 2.定义模板上下文，以键值对的方式注入变量
          context = RequestContext(request，{})
          # 3.模板渲染，产生标准的html内容
          res_html = temp.render(context)
          # 最后返回这个渲染好的html
          return HttpResponse(res_html)
          
      ```

      在实际工作中，django还给我们一个render函数用于简化上述代码

      ```python
      # 导入这个render函数
      from django.shortcuts import render
      
      def index(request)：
      	return render(request, 'book/index.html',{})
      ```

      关于模板上下文的{ 'key' : 'value' }用法，在views.py中传入的数据，在index.html中用{{ key }}来获取

      ```python
      def index(request):
          return render(request, 'book/index.html',{
              'var':'hello',
              'list';[1,2,3]
          })
      ```

      ```html
      <body>
          <h1>这是一个模板文件</h1>
          使用模板变量
          {{ var }}<br/>
          使用列表
          {{ list }}<br/>
          <ul>
              {% for i in list %}
              	<li>{{ i }}</li>
              {{% endfor %}}
          </ul>
      </body>
      ```

- 其他静态资源管理（图片等）

  在   **项目**/  下新建文件夹 **static**    以后图片等静态资源放这里
  在   **项目**/**项目同名pkg**/setting.py 下配置静态文件 **访问地址** 和 **资源文件夹**

  ```python
  # 指的浏览器的访问地址
  STATIC_URL = '/static/'
  # 指这个路由索要资源要去往的后台文件夹status
  STATICFILES_DIRS = [os.path.join(BASE_DIR, 'static')]
  ```

  