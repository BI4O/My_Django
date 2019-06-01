# Django

​	真正的傻瓜式入门整理
​	让每一个 ~~傻瓜~~ 初学者都能从零开始开发属于自己的web应用
​																																						————BI4O倾力呈现

* #### 什么是Django

  Django是一个开放源代码的Web应用框架，由Python写成。采用了MVT的框架模式，即模型M，视图V和模版T。它最初是被开发来用于管理劳伦斯出版集团旗下的一些以新闻内容为主的网站的，即是CMS（内容管理系统）软件。并于2005年7月在BSD许可证下发布。这套框架是以比利时的吉普赛爵士吉他手Django Reinhardt来命名的。

* #### 什么是MVT

  | 模块                        | 负责的内容                                                   |
  | --------------------------- | ------------------------------------------------------------ |
  | 模型（Model），即数据存取层 | 处理与数据相关的所有事务： 如何存取、如何验证有效性、包含哪些行为以及数据之间的关系等。 |
  | 模板(Template)，即表现层    | 处理与表现相关的决定： 如何在页面或其他类型文档中进行显示。  |
  | 视图（View），即业务逻辑层  | 存取模型及调取恰当模板的相关逻辑。模型与模板的桥梁。         |

* #### Django基础——搭建项目的基本架构

  * 完整的web应用，从新建项目及应用开始
    * [Django之创建项目与应用的注册](https://github.com/BI4O/My_Django/blob/master/Django%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0/Django%E5%88%9B%E5%BB%BA%E9%A1%B9%E7%9B%AE%E4%B8%8E%E5%BA%94%E7%94%A8%E7%9A%84%E6%B3%A8%E5%86%8C.md)
  * 网站的访问到最终呈现本质是浏览器向服务器发出请求信息`Request`，
    后台的视图函数/类视图`View`对它进行加工，然后向浏览器返回响应信息`Response`的过程
    * [Django之视图函数View与类视图](https://github.com/BI4O/My_Django/blob/master/Django%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0/Django%E4%B9%8B%E8%A7%86%E5%9B%BE%E5%87%BD%E6%95%B0View%E4%B8%8E%E7%B1%BB%E8%A7%86%E5%9B%BE.md)
  * 为什么网页返回的是漂亮的网页而不是一串冰冷的响应信息`Response`呢？
    * [Django之模板Templates](https://github.com/BI4O/My_Django/blob/master/Django%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0/Django%E4%B9%8B%E6%A8%A1%E6%9D%BFTemplates.md)
  * 网页应用不能虚有其表，它是有内在（数据）的
    * [Django之模型类Model](https://github.com/BI4O/My_Django/blob/master/Django%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0/Django%E4%B9%8B%E6%A8%A1%E5%9E%8B%E7%B1%BBModel.md)
  * 管理员专用的后台管理界面
    - [Django之ORM与后台管理](https://github.com/BI4O/My_Django/blob/master/Django%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0/Django%E4%B9%8BORM%E4%B8%8E%E5%90%8E%E5%8F%B0%E7%AE%A1%E7%90%86.md)
  
* #### Django高级——搭建DRF工程

  可以理解为，DRF是Django的一个小插件，大大简化了增删改查的代码

  * DRF(Django Restful framework)的由来
    - [Django之RESTful设计风格](https://github.com/BI4O/My_Django/blob/master/Django%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0/Django%E4%B9%8BRESTful%E8%AE%BE%E8%AE%A1%E9%A3%8E%E6%A0%BC.md)
  * 搭建DRF工程先从序列化器说起
    - [Django之搭建DRF工程与创建序列化器](https://github.com/BI4O/My_Django/blob/master/Django%E5%9F%BA%E7%A1%80%E7%AC%94%E8%AE%B0/Django%E4%B9%8B%E6%90%AD%E5%BB%BADRF%E5%B7%A5%E7%A8%8B%E4%B8%8E%E5%88%9B%E5%BB%BA%E5%BA%8F%E5%88%97%E5%8C%96%E5%99%A8.md)
  * DRF对视图函数的封装
    - 

