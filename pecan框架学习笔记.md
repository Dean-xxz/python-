## Pecan框架学习笔记
***
### 一、Pecan框架简介及安装

#### 1.框架简介
&nbsp;&nbsp; Pecan是一个路由对象分发的oython web框架。本质上可以将url通过分割为每一部分，然后对每一部分查找对应处理该URL部分的处理类，处理后，继续交给后面部分的URL处理，直到所有URL部分都被处理后，调用最后分割的URL对应的处理函数处理。

&nbsp;&nbsp;Pecan框架的目标是实现一个采用对象分发方式进行URL路由的轻量级Web框架。它非常专注于自己的目标，它的大部分功能都和URL路由以及请求和响应的处理相关，而不去实现模板、安全以及数据库层，这些东西都可以通过其他的库来实现。
  + 不用自己写WSGI application了
  + 请求路由很容易就可以实现了
#### 2.框架安装
  + 安装pecan
        pip install pecan
  + 创建项目
        pecan create test_pecan
  + 项目初始化
        python setup.py develop
  + 启动项目
        pecan serve config.py

&nbsp;&nbsp;酱紫，访问http://localhost:8080 你的第一个pecan项目就运行成功了
***
### 二、框架结构及文件作用
  + **public**：存放web应用所需的一些img，css和js
  + **test_pecan/controllers/**: 包含所有的控制器，也是api具体逻辑实现的地方
  + **test_pecan/controllers/root.py**：包含更路经对应的控制器
  + **test_pecan/controllers/v1/**: 存放v1版本的api
  + **test_pecan/model**：定义数据库相关，配合sqlalchemy使用效果更佳
  + **test_pecan/templates**：存储html或json模板文件
  + **test_pecan/tests**：存放测试用例
  + **test_pecan/app.py**: Pecan应用入口，包含初始化代码
  + **config.py**：包含Pecan应用的配置，会被app.py使用
  + **setup.py**：web应用的安装部署
  + **setup.cfg**： web应用的安装部署
***
### 三、配置及路由分发
#### 1.配置文件config.py
      # Server Specific Configurations
      server = {
          'port': '8080',
          'host': '0.0.0.0'
      }

      # Pecan Application Configurations
      app = {
          'root': 'test_pecan.controllers.root.RootController',
          'modules': ['test_pecan'],
          'static_root': '%(confdir)s/public',
          'template_path': '%(confdir)s/test_pecan/templates',
          'debug': True,
          'errors': {
              404: '/error/404',
              '__force_dict__': True
          }
      }
  + **modules**:其中包含的python包，是app.py文件所在的包，即setup_app方法所在的包
  + **root:** 即RootController所在的路径，/路径
  + **server:** 应用运行的ip和端口号

#### 2.根路径控制器root.py
&nbsp;&nbsp;在root.py中我们通常可以使用下面的方式来进行get和post方法逻辑

    from pecan import rest
    import pecan

    class RootController(rest.RestController):

        @pecan.expose()
        def get(self):
            return 'This is RootController GET.'

        @pecan.expose()
        def post(self):
            return 'This is RootController POST.'

        _custom_actions = {
           'test': ['GET', 'POST'],
       }

       @pecan.expose()
       def test(self):
           if pecan.request.method == 'POST':
               return 'This is RootController test POST.'
           elif pecan.request.method == 'GET':
               return 'This is RootController test GET.'

#### 3.对象分发式路由（重点）

    class v1Controller(rest.RestController):
        @pecan.expose()
        def get(self):
            return 'This is v1Controller GET.'


    class RootController(rest.RestController):

        v1 = v1Controller()

  + 在RootController类中，生成了一个v1Controller对象，这个对象就是用来做分发式路由的，因此可以使用如下命令去调用它：

        curl -X GET http://127.0.0.1:8080/v1


***
### 四、RESTful支持

&nbsp;&nbsp;我们上面举例的controller都是继承自pecan.rest.RestController，这种controller称为RESTful controller，专门用于实现RESTful API的，因此在Openstack中使用特别多，Pecan还支持普通的controller，称为Generic controller。Generic controller继承自object对象，默认没有实现对RESTful请求的方法。简单的说，RESTful controller帮我们规定好了get_one(),get_all(),get(),post()等方法对应的HTTP请求，而Generic controller则没有

#### 1.RESTful controller
  + get_one: GET /books/1
  + get_all: GET /books/
  + get: GET /books/1 or GET /books/
  + new: GET /books/new
  + edit: GET /books/1/edit
  + post: POST /books/
  + put: PUT /books/1
  + get_delete: GET /books/1/delete
  + delete: DELETE /books/1

        from pecan import rest
        import pecan

        class RootController(rest.RestController):

            @pecan.expose()
            def get(self):
                return 'This is RootController GET.'

            @pecan.expose()
            def post(self):
                return 'This is RootController POST.'

#### 2.Generic controller

    class RootController(rest.RestController):
         _custom_actions = {
             'test': ['GET'],
         }

         @expose()
         def test(self):
             return 'hello'

对于RestController中没有预先定义好的方法，我们可以通过控制器的_custom_actions属性来指定其能处理的方法。

    curl http://localhost:8080/test
***

### 五、wsme规范请求和响应
wsme模块可以用来规范API的请求和响应值，并且可以和pecan模块很好的结合在一起

#### 1.wsme支持的type
  + str：对应json type（String）
  + unicode：对应json type（String）
  + int：对应json type（Number）
  + float：对应json type（Number）
  + bool：对应json type（Boolean）
  + Decimal：对应json type（String）
  + date：对应json type（String（YYYY-MM-DD））
  + time：对应json type（String (hh:mm:ss)）
  + datetime：对应json type（String (YYYY-MM-DDThh:mm:ss)）
  + Arrays：对应json type（Array）
  + None：对应json type（null）
  + Complex types：对应json type（Object）


        from wsmeext.pecan import wsexpose
        from pecan import rest

        class RootController(rest.RestController):

            @wsexpose(int, int)
            def get_one(self, arg):
                return 1

  + **第一个int表示返回值**必须为int
  + **第二个int表示请求参数**必须为int

#### 2.wsme检测类对象
可以使用wsme检测接口返回值必须为某一个对象

    from wsme import types as wtypes
    from wsmeext.pecan import wsexpose
    from pecan import rest

    class User(wtypes.Base):
        name = wtypes.text
        age = int

    class Users(wtypes.Base):
        users = [User]

    class RootController(rest.RestController):

    	@wsexpose(User, int)		# 返回值为User类对象
        def get_one(self, id):
            if id == 1:
                user_info = {
                	'name': 'yangsijie',
                	'age': 23
            	}
            return User(**user_info)
        	# 或者也可以使用下面这种方式：
            # if id == 1:
            # return User(name='yangsijie', age=23)

        @wsexpose(Users)	# 返回值为Users类对象
        def get_all(self):
            user_info_list = [
                {
                	'name': 'yangsijie',
                	'age': 23
            	},
            	{
                	'name': 'panna',
                	'age': 23
            	}
            ]
            users_list = [User(**user_info) for user_info in user_info_list]
            return Users(users=users_list)

#### 3.测上传的参数是否为complex type类型

    class RootController(rest.RestController):

        @wsexpose(None, body=User)	# 检查参数必须为User类对象
        def post(self, user):
            print user.name
            print user.age

测试一下：

    curl -X POST http://127.0.0.1:8081 -H "Content-Type: application/json" -d '{"user": {"name": "yangsijie", "age": 30}}'

#### 4.使用wsme设置status_code
使用wsme默认返回的状态码为200、400和500。如果将之前的例子做略微的修改，可以让其返回的状态值不一样

    from wsmeext.pecan import wsexpose
    from pecan import rest

    class RootController(rest.RestController):

        @wsexpose(int, int, status_code=201)	# 之前如果成功的话，返回的是200，现在将其改为201
        def get_one(self, arg):
            return 1

但是在平时的使用中，我们肯定需要根据不同的判断，返回不同的status_code，那怎么办呢？我们可以使用如下的方式去实现：

  + 使用wsme.api.Response在返回值的同时指定status_code
  + 抛出wsme.exc.ClientSideError错误的同时，指定错误原因及status_code
  + 抛出自定义错误，并且在自定义错误中指定错误原因及status_code


        import wsme
        from wsme import types as wtypes
        from wsmeext.pecan import wsexpose
        from pecan import rest

        class BookNotFound(Exception):		# 自定义错误
            message = 'Book with ID={id} Not Found'		# 定义错误原因
            code = 404									# 定义status_code

            def __init__(self, id):
                message = self.message.format(id=id)
                super(BookNotFound, self).__init__(message)

        class Book(wtypes.Base):
            id = int
            name = wtypes.text

        class BookController(rest.RestController):
        	@wsexpose(Books, int)
            def get_one(self, id):
                if id == 1:
                    raise BookNotFound(id=id)	# 使用自定义错误返回
                elif id == 2:
                    raise wsme.exc.ClientSideError('ID: \'1\' is wrong!!!', status_code=403)										# 使用wsme.exc.ClientSideError抛出错误
                else:
                    return wsme.api.Response(Books(), status_code=204)
***
### 六、任务实践
自己写一个玩一下吧！！！
