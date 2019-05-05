## SQLAlchemy学习笔记
***
### 一、ORM框架简介
  对象关系映射是企业及开发环境中比较主流的的开发方法，一般以中间件的形式存在，主要实现程序对象到关系数据库数据的映射。
  ***
### 1.ORM方法论的三个核心原则

  + 简单性：以最基本的形式建模数据
  + 传达性：数据库结构被任何人都能立即的语言文档化
  + 精确性：基于数据库模型创建正确标准化了的结构

### 2.一般的ORM包括以下四部分

  + 一个对持久类对象进行CURD操作的API
  + 一个语言或API用来规定类和类属性相关的查询
  + 一个规定maping metadata的工具
  + 一种技术可以让ORM的实现同事务对象一起进行dirty checking, lazy association fetching以及其他的优化操作
```
        class User(object):
            def __init__(self, id, name):
                self.id = id
                self.name = name
        [
            User(1, “feng”),
            User(2, “shang”),
            User(3, “huo”),
        ]
```

数据库中每次查出来的数据都用一个类表示，这个类的属性和数据库中标的字段一一对应，多条数据就是一个list。

***
## 二、SQLAlchemy介绍
Python中最为有名的通用ORM框架就是这个SQLAlchemy，下面一起来学习一下如何使用它。

### 1.安装环境

#### （1）安装SQLAlchemy
    pip install SQLAlchemy

#### （2）安装mysql
    yum install mysql-server mysql
    service mysqld restart
    sysctmctl restart mysql.service

#### （3）创建数据库
    create database sqlalchemy;

#### （4）数据库授权连接
    GRANT ALL PRIVILEGES ON *.* TO 'fxq'@'%' IDENTIFIED BY ‘123456’;

#### （5）初始化连接
    from sqlalchemy import create_engine
    engine = create_engine('mysql://fxq:123456@192.168.100.101/sqlalchemy', echo=True)
  + echo参数为True时，会显示每条执行的sql语句，可以自行关闭
  + create_engine()返回一个engine实例，数据库语法通过它会被解释成python的类方法
  + 参数说明：
    - mysql:指定是哪种数据库连接（这里是mysql，可以是psql或者别的……）
    - 第一个fxq：数据库用户名
    - 123456：用户名fxq的密码
    - 192.168.100.101：数据库ip
    - sqlalchemy：需要连接的数据库名称

#### （6）创建表格

##### （6.1） 通过sql语句创建表格
    from sqlalchemy import create_engine
    from sqlalchemy.orm import sessionmaker

    sql = '''create table student(
        id int not null primary key,
        name varchar(50),
        age int,
        address varchar(100));
    '''

  + engine = create_engine('mysql+pymysql://fxq:123456@192.168.100.101/sqlalchemy')
  + conn = engine.connect()
  + conn.execute(sql)
  + engine.connect() #表示获取到数据库连接。类似我们在MySQLdb中游标course的作用。

##### （6.2） 通过ORM方式创建表格
    from sqlalchemy import create_engine, MetaData, Table, Column, Integer, String

    engine = create_engine('mysql+pymysql://fxq:123456@192.168.100.101/sqlalchemy')
    metadata = MetaData(engine)


    student = Table('student', metadata,
                Column('id', Integer, primary_key=True),
                Column('name', String(50), ),
                Column('age', Integer),
                Column('address', String(10)),
    )

    metadata.create_all(engine)
  + MetaData类主要用于保存表结构，连接字符串等数据，是一个多表共享的对象
  + metadata = MetaData(engine) 绑定一个数据源的metadata
  + metadata.create_all(engine) 是来创建表，这个操作是安全的操作，会先判断表是否存在。

### 3 table类
构造函数：  

    Table.__init__(self, name, metadata,*args, **kwargs)

  + name:表名
  + metadata：共享的元数据
  + *args：Column 是列定义
  + 可变参数 **kwargs 定义：
    - schema： 此表的结构名称，默认None
    - autoload： 自动从现有表中读入表结构，默认False
    - autoload_with： 从其他engine读取结构，默认None
    - include_columns：True表示此项数组中的列名将被引用，默认为None，表示均被引用
    - mustexist：为True表示此表必须在其他的python应用中定义，必须是metadata的一部分，默认为false
    - useexisting：为True表示此表必须被其他应用定义过，将忽略结构定义，默认为false
    - owner：表的所有者，默认为None
    - quote：设置为True时，如果表名是sql关键词，将强制转义，默认为false
    - quote_schema：设置为True时，如果列名是sql关键词，将强制转义，默认为false
    - mysql_engine：mysql专用，可以设置“InnoDB”或“MyISAM”

### 4 Column类
构造函数：

    Column.__init__(self,  name,  type_,  *args,  **kwargs)

  + name：列名
  + type_:类型
  + *args： Constraint（约束）, ForeignKey（外键）, ColumnDefault（默认）, Sequenceobjects（序列）定义，key（列名的别名，默认为None）
  + 可变参数 **kwargs：
    - primary_key：如果是True，则设置为主键
    - nullable：是否允许为null，默认为True
    - default: 默认值，默认是None
    - index：是否为索引，默认是True
    - unique：是否唯一，默认false
    - onupdate：指定一个更新时间值，这个操作是定义在SQLAlchemy中，不是在数据库里的，当更新一条数据时设置，大部分用于updateTime这类字段
    - autoincrement：设置为整型自动正常，只有没有默认值，并且是integer类型，默认是True
    - quote: 如果列名是关键词，强制转义，默认为false

### 5 创建会话
  + Session的主要目的是建立与数据库的会话，它维护你加载和关联的所有数据库对象。它是数据库查询（Query）的一个入口
  + 在Sqlalchemy中，数据库的查询操作是通过Query对象来实现的。而Session提供了创建Query对象的接口。
  + ORM通过session与数据库建立连接进行通信，

        from sqlalchemy.orm import sessionmaker

        DBSession = sessionmaker(bind=engine)
        session = DBSession()
  + 通过sessionmake方法创建一个Session工厂，然后在调用工厂的方法来实例化一个Session对象

### 6 添加数据

    from sqlalchemy import create_engine, Column, Integer, String
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.orm import sessionmaker


    engine = create_engine('mysql+pymysql://fxq:123456@192.168.100.101/sqlalchemy')
    DBsession = sessionmaker(bind=engine)
    session = DBsession()

    Base = declarative_base()

    class Student(Base):
        __tablename__ = 'student'
        id = Column(Integer, primary_key=True)
        name = Column(String(100))
        age = Column(Integer)
        address = Column(String(100))

    student1 = Student(id=1001, name='ling', age=25, address="beijing")
    student2 = Student(id=1002, name='molin', age=18, address="jiangxi")
    student3 = Student(id=1003, name='karl', age=16, address="suzhou")

    session.add_all([student1, student2, student3])
    session.commit()
    session.close()

### 7 查询数据
通过Session的query()方法创建一个查询对象。这个函数的参数数量是可变的，参数可以是任何类或者是类的描述的集合。下面来看一个例子：

    from sqlalchemy import Column
    from sqlalchemy import Integer
    from sqlalchemy import String
    from sqlalchemy import create_engine
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.orm import sessionmaker

    Base = declarative_base()
    class Student(Base):
        __tablename__ = 'student'
        id = Column(Integer, primary_key=True)
        name = Column(String(50))
        age = Column(Integer)
        address = Column(String(100))

    engine = create_engine('mysql+pymysql://fxq:123456@192.168.100.101/sqlalchemy')
    DBSession = sessionmaker(bind=engine)
    session = DBSession()

    my_stdent = session.query(Student).filter_by(name="fengxiaoqing2").first()
    print(my_stdent)

#### （1）filter（）过滤表的条件
与sql语法一一对应

      equals:
      query(Student).filter(Student.id == 10001)
      not equals:
      query(Student).filter(Student.id != 100)
      LIKE:
      query(Student).filter(Student.name.like(“%feng%”))

      IN:
      query(Student).filter(Student.name.in_(['feng', 'xiao', 'qing']))
      not in
      query(Student).filter(~Student.name.in_(['feng', 'xiao', 'qing']))

      AND:
      from sqlalchemy import and_
      query(Student).filter(and_(Student.name == 'fengxiaoqing', Student.id ==10001))

      或者
      query(Student).filter(Student.name == 'fengxiaoqing').filter(Student.address == 'chengde')

      OR:
      from sqlalchemy import or_
      query.filter(or_(Student.name == 'fengxiaoqing', Student.age ==18))

#### （2）返回列表和单项

    my_stdent = session.query(Student).filter(Student.name.like("%feng%")).all()
    print(my_stdent)

    my_stdent = session.query(Student).filter(Student.name.like("%feng%")).first()
    print(my_stdent)

    my_stdent = session.query(Student).filter(Student.name.like("%feng%")).one()
    print(my_stdent)

+ all() 返回一个列表
+ one() 返回且仅返回一个查询结果。当结果的数量不足一个或者多于一个时会报错。把上面的all改成one就报错了
+ first() 返回至多一个结果，而且以单项形式，而不是只有一个元素的tuple形式返回这个结果.

#### （3）filter()和filter_by()的区别
  + Filter： 可以像写 sql 的 where 条件那样写 > < 等条件，但引用列名时，需要通过 类名.属性名 的方式
  + filter_by： 可以使用 python 的正常参数传递方法传递条件，指定列名时，不需要额外指定类名。，参数名对应名类中的属性名，但似乎不能使用 > < 等条件。
  + 当使用filter的时候条件之间是使用“=="，fitler_by使用的是"="
  + filter不支持组合查询，只能连续调用filter来变相实现。而filter_by的参数是**kwargs，直接支持组合查询

        user1 = session.query(User).filter_by(id=1).first()
        user1 = session.query(User).filter(User.id==1).first()

        q = sess.query(IS).filter(IS.node == node and IS.password == password).all()

### 8 更新数据
更新就是查出来，直接更改就可以了
    from sqlalchemy import Column
    from sqlalchemy import Integer
    from sqlalchemy import String
    from sqlalchemy import create_engine
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.orm import sessionmaker

    Base = declarative_base()
    class Student(Base):
        __tablename__ = 'student'
        id = Column(Integer, primary_key=True)
        name = Column(String(50))
        age = Column(Integer)
        address = Column(String(100))

    engine = create_engine('mysql+pymysql://fxq:123456@192.168.100.101/sqlalchemy')
    DBSession = sessionmaker(bind=engine)
    session = DBSession()

    my_stdent = session.query(Student).filter(Student.id == 1002).first()
    my_stdent.name = "fengxiaoqing"
    my_stdent.address = "chengde"
    session.commit()
    student1 = session.query(Student).filter(Student.id == 1002).first()
    print(student1.name, student1.address)

### 9 删除数据
删除其实也是跟查询相关的，直接查出来，调用delete()方法直接就可以删除掉。

    from sqlalchemy import Column
    from sqlalchemy import Integer
    from sqlalchemy import String
    from sqlalchemy import create_engine
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.orm import sessionmaker

    Base = declarative_base()
    class Student(Base):
        __tablename__ = 'student'
        id = Column(Integer, primary_key=True)
        name = Column(String(50))
        age = Column(Integer)
        address = Column(String(100))

    engine = create_engine('mysql+pymysql://fxq:123456@192.168.100.101/sqlalchemy')
    DBSession = sessionmaker(bind=engine)
    session = DBSession()


    session.query(Student).filter(Student.id == 1001).delete()
    session.commit()
    session.close()

### 10 统计、分组、排序
count()、group_by()、order_by()

    from sqlalchemy import Column
    from sqlalchemy import Integer
    from sqlalchemy import String
    from sqlalchemy import create_engine
    from sqlalchemy.ext.declarative import declarative_base
    from sqlalchemy.orm import sessionmaker

    Base = declarative_base()
    class Student(Base):
        __tablename__ = 'student'
        id = Column(Integer, primary_key=True)
        name = Column(String(50))
        age = Column(Integer)
        address = Column(String(100))

    engine = create_engine('mysql+pymysql://fxq:123456@192.168.100.101/sqlalchemy')
    DBSession = sessionmaker(bind=engine)
    session = DBSession()

    print(session.query(Student).filter(Student.name.like("%feng%")).count())

    std_group_by = session.query(Student).group_by(Student.age)
    print(std_group_by)

    std_ord_desc = session.query(Student).filter(Student.name.like("%feng%")).order_by(Student.id.desc()).all()
    for i in std_ord_desc:
      print(i.id)

***

## 三、常用sqlalchemy字段类型.配置选项和关系选项

### 1 常用字段类型
  + Integer：对应python中int，普通整数，一般32位
  + SmallInteger：对应python中int，取值范围小的整数，一般16位
  + BigInteger：对应python中int或long，不限制精度的整数
  + Float：对应python中float，浮点数
  + Numeric：对应python中decimal.Decimal，普通整数，一般为32位
  + String：对应python中的string，字符串
  + Text：对应python中string，字符串，对较长或不限制长度的字符串做了优化
  + Boolean：对应python中的bool，布尔值
  + Date：对应python中的datetime.date,时间
  + Time：对应python中的datetime.datetime, 日期和时间

### 2 常用的SQLAlchemy列选项
  + primary_key：如果为True，代表表的主键
  + unique：如果为True，代表本列不允许出现重复值
  + index：如果为True，代表为本列创建索引
  + nullable：如果为True，代表本列允许有空值
  + default：为此列设置默认值
