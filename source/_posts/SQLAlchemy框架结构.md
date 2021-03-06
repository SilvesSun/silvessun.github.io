---
title: SQLAlchemy
date: 2018-01-29 18:04:01
tags: SQLAlchemy
categories:
 - Python
 - 开源框架
 - 翻译
---

这是本系列的第一章. 今天来读一读SQLAlchemy的结构
[参考原文](http://aosabook.org/en/sqlalchemy.html)

SQLAlchemy是Python编程语言下的一款开源软件.提供了SQL工具包及对象关系映射工具，使用MIT许可证发行. 最先发布于2005年.

 SQLAlchemy试图在Python处理数据库的过程中, 提供一个使用Python数据库API交互的端对端系统.  即使是在它早期的版本中, 它的性能也吸引了很多的关注. 主要特点有:  流畅的处理大量复杂的查询和映射工作, 以及"工作单元"的实现, 为将数据持久化到数据库中提供了 一个高度自动化的系统.

 # 数据库抽象面临的挑战
 "数据库抽象" 通常用来表示一个隐藏了数据存储和查询的细节的系统. 这个术语有时候会被人们极端化,  认为不仅数据库的细节应该被隐藏, 而且关系结构本身也应该被隐藏,  甚至数据库是否是关系型的也应该被隐藏.

 有关ORMs的大部分评论认为这种系统的主要目的正如上所述, 将关系数据库的使用隐藏起来, 接管构建数据库和与数据库交互的任务, 简化为具体的实现.这种方式的核心在于剥夺开发人员对关系结构进行设计和查询的能力，转交由不透明的库来处理

 经常和数据库打交道的人们知道这种方法时完全不现实的. 结构关系和SQL查询有很强的功能性, 且构成了应用设计的核心. 这些数据结构应该如何设计, 组织以及处理查询的结果不仅取决于需要查询的数据, 还取决于数据的结构. 如果这些都被完全的隐藏了, 那么在一开始使用关系型数据库的意义就很小了.

 既要寻求屏蔽关系数据库底层细节的方法，又面对着关系数据库需要详尽的说明的事实。这种矛盾通常被称为“对象-关系阻抗失配”(object-relational impedance mismatch)问题。SQLAlchemy采用了一种比较新颖的方法来解决这个问题。

 ## SQLAlchemy解决数据库抽象的方法
 SQLAlchemy认为开发人员必须考虑数据的关系结构. 一个预先定义并隐藏数据模式(schema)和查询方法的系统只是在忽视关系型数据库的意义，导致传统的阻抗失配问题。

 但与此同时, 这些决定的实现可以在而且应该在更高的层次上实现.  在对象模型和数据库scheme间建立联系, 并在查询过程中保持这种联系是一种高度重复的工作. 使用工具自动化执行这些任务，可以使应用开发更加简洁、高效。创建自动化工具的时间，远远少于手工实现这些操作的时间。

 基于此, SQLAlchemy将自己定义为一个工具包, 强调开发人员是所有关系机构及其应用联系的构造者和设计者, 而不是被动的接受第三方库的决定. SQLAlchemy采取“不完全抽象”理念，暴露关系概念，鼓励开发者在应用程序和关系数据库之间裁剪出一个自定义的、但又是完全自动化的交互层。SQLAlchemy的创新之处在于，它在不牺牲开发者对于关系数据库的控制的同时，实现了高度的自动化。

 # 核心层和ORM层
 为了达到提供一套工具的目的, SQLAlchemy提供了丰富的接口用于和数据库的每一次进行交互, 并分为核心层和ORM层.  核心层包括Python与数据库交互的API(DBAPI), 生成数据库能识别的SQL语句, 以及数据架构(scheme)管理. 这些功能都在APIs展现出来. ORM(对象关系映射), 则是构建在核心层上的一个特定的库.SQLAlchemy提供的ORM层只是可以构建在核心层上的众多对象抽象层的其中一个，很多开发者和组织是直接在核心层上构建自己的应用。
 ![SQLAlchemy层次图](http://p3euxxfa8.bkt.clouddn.com/sqlalchemy_layer.png)

 核心层和ORM的分离一直是SQLAlchemy的主要特点, 对此支持者和反对者都有. 核心层的显式存在导致：（一）ORM需要将映射到数据库的类属性关联到一个叫Table的结构上，而不是直接关联到数据库中表述的字符串属性名。（二）ORM需要使用一个叫select的结构来产生SELECT查询，而不是直接将对象属性拼接成一个字符串的语句。（三）ORM需要从ResultProxy接受结果行（ResultProxy自动将select映射到每个结果行），而不是直接操纵数据库游标(cursor)将数据转化成用户定义的对象

 在一个很简单的以orm为中心的应用中, 核心层几乎是不可见的. 然而, 当形式所需时为了使核心层和ORM流畅的转化; 或者在复杂的以ORM为中心的应用中, 更具体, 细致的处理数据库, 可以下潜一两个抽象层次, 核心层和ORM很好地结合在了一起. 随着SQLAlchemy的逐渐成熟, ORM提供了越来越多的全面而复杂的模式, 在日常使用中,已经很少使用 核心层的API了. 但是在早期ORM未成熟时, 可操控的核心层使许多看似不可能的任务得到完成, 这是SQLAlchemy能成功的因素之一.

 这种核心层/ORM架构的缺点是, 一个指令必须经过更多的步骤. 传统的使用C实现的Python, 运行时速度缓慢的主要原因是单独的函数调用. 解决这个问题的办法是通过重排或者内联减少调用链, 并将性能要求高的关键代码用C实现. SQLAlchemy多年来使用这两种种办法来提升性能. 然而随着PyPy解释器的逐渐发展, 得益于PyPy通过JIT(just_run_intime)内联减少了长调用,SQLAlchemy的性能问题可能得到改善而不需要用C代码实现.

 # 改良DBAPI
 SQLAlchemy是通过底层的DBAPI来实现与数据库交互. DBAPI本身并不是一个实际的库, 只是一个标准. 因此, 针对不同的数据库, DBAPI的实现也可能不同.

 DBAPI带来两大挑战, 一是围绕DBAPI的基本使用提供一个易用且功能全面的接口, 二是针对不同的数据库引擎提供特定的实现.

 ## the Dialect System
 DBAPI描述的接口及其简单. 它的核心组件包括: DBAPI自己, 一个连接, 一个游标("游标""在数据库中指一个语句（statement）和它相关的结果的上下文). 如下所示为一个简单的连接数据库并提供结果的例子.
 ```py
connection = dbapi.connect(user="user", pw="pw", host="host")
cursor = connection.cursor()
cursor.execute("select * from user_table where name=?", ("jack",))
print "Columns in result:", [desc[0] for desc in cursor.description]
for row in cursor.fetchall():
    print "Row:", row
cursor.close()
connection.close()
```
SQLAlchemy 在传统的DBAPI上进行封装. 通过调用create_engine, 连接数据库和配置信息也是在此配置的, 返回一个engine的实例. 通过这个对象来访问并未直接暴露的DBAPI.

engine提供"隐式执行"接口来执行简单语句. 获取和关闭DBAPI连接的过程都被隐藏了起来:
```py
engine = create_engine("postgresql://user:pw;host/dbname")
result = engine.execute("select * from table")
print result.fetchall
```

SQLAlchemy 0.2新增一个Connection对象, 从而实现了对DBAPI连接的显式维护.
```py
conn = engine.connect()
result = conn.execute("select * from table")
print result.fetchall()
conn.close()
```
Engine或者Connection的execute方法返回的结果为ResultProxy. ResultProxy提供了类似于DBAPI 游标但是比DBAPI功能更丰富的接口. Engine, Connection和ResultProxy分别对应于DBAPI, DBAPI连接实例, 和一个特定的DBAPI游标对象

从底层来看, Engine代表着Dialect这个抽象类. 这个抽象类对应于不同的数据库的时候, 都有不同的DBAPI和数据库连接实现.  一个Connection对象的建立是因为对于所有的请求, 最终Engine都会指向Dialect对象, 具体到不同的DBAPI或者数据库, 都有可能不同.

当Connection创建时, 会从连接池(pool)中获取并维护一个DBAPI连接, Pool对象主要负责创建新连接, 并在内存中维护一个连接池以供频繁调用.

在语句执行的过程中, Connection会创建一个ExecutionContext对象. 这个对象从语句开始执行直到ResultProxy销毁一直存在.  同时这个对象也可用于某些DBAPI和数据库组合的子类.

下图展示了这些对象之间, 以及和DBAPI组件之间的关系:
![组件关系](http://p3euxxfa8.bkt.clouddn.com/compents_relation.png)

## DBAPI多样性问题的解决
为了完成管理DBAPI多样性的任务, 首先来看看这个问题的领域. DBAPI规范(目前是第二版), 定义为一系列允许多样行为的接口以及许多未定义的接口. 从而使实际使用的DBAPI在不同的领域有很大的不同, 包括是否接受Python Unicode, 在执行Insert语句后如何获取自增主键, 参数生效的范围. 以及很多针对特殊类型时的行为, 包括处理二进制, 精确值, 日期, 布尔值, Unicode.

SQLAlchemy解决这个问题的方法是, 在Dialect 和 ExecutionContext上运行多级子类的多样性. 下图展示了当使用psycopg2时Dialect和ExecutionContext之间的关系. PGDialect提供针对Postgresql数据库包括ARRAY和catalogs的特定行为, PGDialect_psycopg2提供了特定于psycopg2 DBAPI的行为，包括Unicode处理和服务器游标的行为.
![简单的Dialect/ExecutionContext继承体系](http://p3euxxfa8.bkt.clouddn.com/simple_dialect_inherit.png)

在处理多数据任务时, 上述模式有一个变体. 比如当包括处理任意ODBC数据库的pyodbc, JDBC的Jython驱动zxjdbc, 通过使用一个继承自sqlalchemy.connectors(提供了不同后端共有的DBAPI行为)的混合类来完成上述关系.
![Common DBAPI behavior shared among dialect hierarchies](http://p3euxxfa8.bkt.clouddn.com/dbapi_behavior.png)

Dialect 和 ExecutionContext使得可以定义和数据库的每一次交互(数据库连接时的格式化, 如何处理语句执行时的古怪情况). 同时Dialect还是一个生成适用于特定数据库SQL语句,Python类型和数据库类型直接如何转化的工厂.

# 模式定义(scheme defination)

完成数据库连接和交互的任务后, 接下来需要提供创建和操作(数据库)的SQL语句. 首先, 需要定义如何表示数据库的表和(表中的)字段, 即数据库架构.  数据表和字段展示了数据时怎样被组合起来的, 所以大部分SQL语句也是指向这些结构的.

一个ORM或者数据接入层需要提供SQL语句在程序上的对应关系.(原文是programmatic access to the SQL language, 在程序级别上有指向SQL语句的表示, 我理解为对应的SQL语句需要有相应的ORM表述); 这正是基于一个表示表和字段的编程系统. SQLAlchemy提供Table和Column这两个独立于用户的模型类定义的结构, 使得核心层和ORM分离. 这样做的原理是, 定义关系型数据库的关系结构, 必要的时候包括平台细节, 可以被明确的设计, 而无需考虑关系对象, 这位分离ORM和核心层提供了基础. 独立于ORM组件也意味着, 模式定义同样适用于其他可能基于核心层构建的关系系统.

Table和Column属于metadata, 提供一个Metadata的集合来表示Table对象. 这个结构源自Martin Fowler在*Patterns of Enterprise Application Architecture*中提到的"Metadata Mapping" . 下图展示了**sqlalchemy.schema**的一些关键元素
![key elements of the sqlalchemy.schema](http://p3euxxfa8.bkt.clouddn.com/key_elements.png)

Table描述了实际目标表的名字和一些其他属性, 它所含 Column描述了每个表中字段名和字段类型, (还提供 )一系列描述描述constraint, index, sequence的对象. 其中有一些可作用于数据库引擎和SQL构建系统. 特别的, ForeignKeyConstraint决定了两个表怎样连接.

Table和Column相较其他模式包中的元素, 其独特在于双重继承的. 同时继承自sqlalchemy.schema和sqlalchemy.sql.expression. 不仅仅是scheme级别的组件, 同时也是SQL语句中的核心元素.
![](http://p3euxxfa8.bkt.clouddn.com/markdown-img-paste-20171121114457645.png)

我们可以看出，Table继承自SQL的FromClause，即“你能select的对象”；Column继承自SQL的ColumnElement，即“你能在SQL表达式中使用的东西”。

# SQL表达式

最开始, 如何生成SQL并不明确. 文本语言是可考虑的选择, 这是大部分像Hibernate的HQL的知名ORM框架所采用的方式. 对于Python来说, 可能有一个更好的方法, 使用Python对象和表达式来生成树状表达式. 甚至重新设计Python的操作符使其能表现SQL行为.

SQLAlchemy的表达式语言源自lan Bicking的SQLObject的SQLBuilder库的启发. 在这种方式中, Python对象作为一个SQL表达式的词法单元, 这些对象上的方法和操作符, 构成了词法结构. 最常见的对象是Column对象. SQL对象使用映射对象表示这些对象, 通过.q访问命令空间. SQLAlchemy将这个属性命令为.c, .c属性在核心层的selectable一直保留到现在, 在表示表和选择的元素上使用.

## 表达式树
一个SQLAlchemy SQL表达式正是解析SQL表达式生成的结构 ------一颗分析树. 区别在于开发者直接构造一颗分析树,而不是从字符串中解析. 分析树的核心节点类型是ClauseElement, 下图标明了ClauseElement和一些关键类的关系.
![ Basic expression hierarchy](http://p3euxxfa8.bkt.clouddn.com/basic_expression_hierarchy.png)

通过使用构造函数, 方法以及重载Python运算符的功能, 一个如下的语句结构:
```SQL
SELECT id FROM user WHERE name = ?
```
在Python中可以构造为
```Python
from sqlalchemy.sql import table, column, select
user = table('user', column('id'), column('name'))
stmt = select([user.c.id]).where(user.c.name=='ed')
```

下图为select的结构图
![Example expression tree](http://p3euxxfa8.bkt.clouddn.com/expression_tree.png)

注意'ed'在_BindParam的结构中, 使其在SQL语句中作为一个绑定参数的查询标记. 我们可以看到, 一次简单的自上向下的遍历就能生成一条SQL语句, 在语句编译部分我们将会看到更多的细节.

## Python操作符
在SQLAlchemy中, 语句
```SQL
column('a') == 2
```

的结果是一个表达式结构, 而不是TRUE或者FALSE. 达到这样的效果是重写Python中像
```py
__eq__, __ne__, __le__, __lt__, __add__, __mul__
```
 这些方法. 使用mixin中的ColumnOperators, 面向字段的表达式节点重载操作符. 从而使上式等价于
 ```py
from sqlalchemy.sql.expression import _BinaryExpression
from sqlalchemy.sql import column, bindparam
from sqlalchemy.operators import eq

_BinaryExpression(
    left=column('a'),
    right=bindparam('a', value=2, unique=True),
    operator=eq
)
 ```
 eq实际上是一个Python内置函数. 将操作符表示为一个对象（如operator.eq）而不是一个字符串（如=）使字符串表示可以在语句编译时，针对具体的数据库方言指定。

## 编译
负责将SQL表达式树渲染成文本SQL的核心类是Compiled, Compiled有两个重要的子类: SQLCompiler和DDLCompiler. SQLCompiler主要负责处理SELECT, INSERT, UPDATE,DELETE这些数据库查询语言(DQL)和数据库操作语言(DML)的SQL渲染, 而DDLCompiler处理CREATE和DROP这些数据库定义语言(DDL).  额外的有继承自TypeCompiler, 主要负责类型的字符串表示的类. 然后针对不同的数据库, 每个数据库继承自这三个编译类, 提供不同的子类.下图展示了postgresql的继承关系:
![ Compiler hierarchy, including PostgreSQL-specific implementation](http://p3euxxfa8.bkt.clouddn.com/compiler_hierarchy.png)

Compiled的子类提供了一系列的visit方法, 每个visit方法被ClauseElement的特定子类所引用. 通过遍历ClauseElement节点, 递归的连接每一个visit方法生成的字符串, 一个表达式就被构造出来了. 在这个过程中，Compiled对象维护关于匿名标识符名、bound parameter名，以及嵌套子查询的状态。这些都是为了产生出一个字符串形式的SQL语句，和带有默认值的bound parameter集合。图20.10展示了visit函数产生出字符串单元的过程
![Call hierarchy of a statement compilation](http://p3euxxfa8.bkt.clouddn.com/call_hierarchy.png)

一个完整的Complied结构包括完整的SQL字符串和绑定的默认值, 这些被ExecutionContext 强制转化为DBAPI的execute方法所能接受的格式.

# 使用ORM映射类
现在来看看ORM. 首先, 要能用自己定义的表的元类和用户定义的数据表之间联系起来; 其次, 要能定义类之间的关系.

SQLAlchemy将这个过程叫做"mapping", 出自Fowler's Patterns of Enterprise Architecture的数据映射模式( Data Mapper pattern). 总体来说, SQLAlchemy ORM从Fowler受益良多, 同时也借鉴了java的Hibernate 和lan Bicking的SQLObject

## 传统式和声明式的对比
术语*传统映射classical mapping* 指将对象-关系数据映射到用户类的系统. 这种方式将Table和用户类认为是两个独立的类, 通过mapper联系起来. 当一个mapper应用到用户类的时候, 这个类就获得了与数据库中的表相对应的字段的属性.
```py
class User(object):
    pass

mapper(User, user_table)

# now User has an ".id" attribute
User.id
```
mapper也能为这个类添加其他的属性, 包括和其他类的联系, 以及任意的SQL表示. 将属性添加到类的过程在Python中叫做"猴子补丁(monkeypatching)", 然而由于我们是基于数据驱动而不是任意的乱加属性, 更恰当的术语应该是*class instrumentation*

现在使用SQLAlchemy的大部分情况主要是使用声明扩展. 声明式扩展是一个如同其他的对象-关系工具使用的配置系统的系统. 在这种系统中, 用户在类中显示的定义属性,
每一个属性和需要映射的属性一一对应.  多数情况下, Table和Mapper不会显示提及, 只有类, Column 以及其他ORM相关的属性被指定.
```py
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
```
上面这段代码看起来就好像通过id = Column()直接操作类(the class instrumentation is being achieved directly, 存疑), 然而事实并非如此. 声明式扩展使用Python元类生成对应于定义的类的Table, 然后和类一起传送给Mapper函数. 然后mapper函数采用同样的方式, 将它的属性(这儿是id)附加到类上, 取代原来的值. 在元类的初始化完成时（即程序执行流离开User类描述的区块时），被id标记的Column对象已经移动到了一个新的Table中，并且User.id已经被一个特定于映射的新属性所取代。

SQLAlchemy总是有着快速, 声明式的表格配置. 然而, 为了保持对传统映射的支持, 声明式的创建被延迟了.早期, 一个叫做ActiveMapper的临时扩展，即后来的Elixir项目, 支持声明式映射. 它在一个较高的层级中重新定义了映射组件. 声明式映射的目标是通过保留SQLAlchemy的传统映射概念，重新组织了它们的使用方式, 使之更加简洁, 比传统映射更适应类级扩展来避免Elixir的高度抽象.

无论使用传统映射还是声明式映射, 映射的对象都能根据其属性表现出与之相应的新行为, 从而可以表达SQL结构. SQLAlchemy继承了SQLObject的行为，使用一个特殊的属性来获取SQL字段表达式，这个属性叫做.c，如下面的例子：
```py
result = session.query(User).filter(User.c.username == 'ed').all()
```
在版本0.4中，SQLAlchemy将这个功能移到映射到的属性自身：
```py
result = session.query(User).filter(User.username == 'ed').all()
```
这个属性访问上的变化被证明是一个巨大的进步，因为它允许在类上出现类似字段（而不是字段）的对象获得额外的特定于类的能力，这些能力并不直接源于底层的Table对象。它还允许不同类型的类属性的集成使用，比如指向直接表中字段的属性，指向从字段生出的SQL表达式的属性，还有指向相关类的属性。最终，它实现了映射类和映射类的实例的对称性，因为同样的属性在不同的类中会有不同的行为。类上的属性返回SQL表达式，而实例上的属性返回实际的数据。

## 映射剖析
User类的id属性是Python描述符对象的一种. 描述符是有 \_get_, \__set__和\__del__方法的对象. 在Python运行过程中, 所有有关这个属性的类的实例的操作都会描述符有 关.  接下来我们用另一个例子来展示Python的实现, 即InstrumentedAttribute. 我们从一个Table和用户定义的类开始，建立了一个只有一个字段的映射，和一个定义了到相关类的引用的relationship：
```py
user_table = Table("user", metadata,
    Column('id', Integer, primary_key=True),
)

class User(object):
    pass

mapper(User, user_table, properties={
    'related':relationship(Address)
})
```

当映射完成后, 和类有关的Objects细节如下:
![ Anatomy of a mapping](http://p3euxxfa8.bkt.clouddn.com/anatomy_of_mapping.png)

上图展示了SQLAlchemy的映射被定义为用户定义的类和这个类映射到的元类直接交互相分离的两层. 左半部分是 Class instrumentation, 右半部分是SQL和数据库相关的功能.
总体的模式为使用对象的组合来分离行为角色，使用对象继承区分一个角色下的行为差异。

在class instrumentation中, ClassManager和映射的类链接在一起, 同时, InstrumentedAttribute对象和映射的对象的属性链接到一起. InstrumentedAttribute也是公开的Python描述符对象的一种, 在基于类的表达式（如User.id==5)中使用时，产生SQL表达式.  当处理User的实例的时候, InstrumentedAttribute将属性的行为委托给一个AttributeImpl对象, 为所表示的数据定制的多个变体之一.

再看映射一侧, Mapper代表的是用户定义的类和数据库单元的联系, 通常来说是Table. Mapper维护了一组MapperProperty属性, 用于处理特定属性的SQL表示. 最常见的MapperProperty 变种是ColumnProperty和RelationshipProperty, 分别代表字段映射和其他Mapper的联系.

MapperProperty负责的是属性加载成LoaderStrategy的行为, 包括属性如何用SQL渲染和如何从获取的结果中生成(的行为). LoaderStrategy有不同的变体, 不同的LoaderStrategies随着加载行为是 deferred, eager和immediate 的不同而不同. 在映射时会有一个默认的配置, 并可在查询时指定. RelationshipProperty同样也代表DependencyProcessor, 用于决定映射器间的依赖关系和在数据刷新到(数据库)时属性同步. DependencyProcessor的决定基于父表和可选目标之间的联系.

Mapper/RelationshipProperty构成了一个有向图,其中 Mapper是节点, RelationshipProperty是有向边. 当一个应用的所有映射定义完成后,随后的 *configuration* 的初始化过程就开始进行. 主要用RelationshipProperty来确定父表和目标表之间的细节(包括AttributeImpl, DependencyProcessor ).  这个图是贯穿整个ORM操作的关键, 诸如定义操作如何沿着对象路径传播的cascade, 在查询过程中, 相关对象和集合的立即加载, 以及在持久化(数据到数据库)之前, 刷新数据时建立对象的依赖关系.

#  查询和加载行为
SQLAlchemy通过Query完成创建对象和数据加载的过程. Query对象的初始状态包括两个: 一个对象实体, 一个对Session的引用. 对象实体可以是一系列类的映射和/或者用于查询的SQL语句. Session表述的是和一个或者多个数据库的连接, 以及事务中的缓存数据.
```py
from sqlalchemy.orm import Session
session = Session(engine)
query = session.query(User)
```
在上面的代码中, 我们创建了一个Query对象, 这个对象将生成一个User对象, 并和刚刚创建的session关联起来. 如何前面讨论的select一样, Query提供了一个生成建造者模式. 一次方法调用会将额外的条件和修饰符关联到一个语句构件上. 当在Query上调用迭代操作时, 它生成一个表示select语句的结构, 然后送到数据库执行, 然后将返回的结果解析为和最开始请求的对象相关联的ORM对象.

Query在SQL渲染和数据加载过程中做了严格的区分. 前者代表的是SELECT语句,  后者指的是将查询到的结果转译为ORM对象. 事实上, 可以在没有SQL渲染的情况下加载数据, 因为Query可能会被要求根据用户的文本查询结果.

如果将每个字段或者有ColumnProperty的SQL看做叶子节点,每个包含在query中的 RelationshipProperty 看作是指向另一个mapper的边,  一系列的Mapper对象就构成了一个图. SQL渲染和数据加载都利用这个图的递归遍历方法. 在每个结点上执行的动作最终是和每个MapperProperty相关的LoaderStrategy的工作，它在SQL渲染阶段将字段和连接(join)添加到创建中的SELECT语句中，在数据加载阶段生成处理结果行的Python函数

每当获取到一个数据行, 在数据加载的过程中生成Python函数, 并在内存中生成可能的关于对象属性映射的结果. 这些函数的生成是基于检测到在结果集中接收到第一个即将到来的行以及在加载行为这些特定条件. 如果一个属性不需要加载, 也就不会生成可调用的函数.

下图展示了在立即加载过程中, 几种LoaderStrategy的遍历过程. 表明了在Query的_compile_context过程中, 连接到渲染的SQL表达式的过程. 同样也揭示了获取结果填充属性的函数生成过程, 这个过程发生在Query的instances方法中.
![Traversal of loader strategies including a joined eager load](http://p3euxxfa8.bkt.clouddn.com/traversal.png)

早期SQLAlchemy使用传统的遍历方法填充结果. 在0.5版本中引入的可调用加载系统使其性能得到极大的提升. 因为很多和行有关的决定只需要在最开始执行一次, 而不用针对每一个行做处理, 这样很多无用的函数调用就可以不去调用了.

# Session/Identity Map
在SQLAlchemy中, Session对象代表的是实际使用ORM过程中用到的公共接口, 即加载数据和持久化数据. 它提供了对指定数据库的查询和持久化数据到数据库的操作.

Session除开作为连接到数据库的接口外, 还维护着所有在内存中和当前Session相关的映射类的数据集的引用. 基于此, Session实现了Fowler定义的两个模式: 身份映射 和 工作单元. 对于特定的Session, 身份映射维护着一个在数据库中标识唯一的所有对象的映射, 从而消除了重复标识可能带来的问题.  基于身份映射的工作单元能最大可能有效的将所有更新自动的持续化到数据库. 实际的持久化步骤就是flush, 在SQLAlchemy中大部分情况下是自动的.

## 开发历史
最开始, Session是一个并不透明的系统, 负责着单个任务的刷新过程. flush过程牵涉到将SQL语句发送到数据库, 并将这些语句产生的结果的状态在内存中同步. flush操作一直是SQLAlchemy所做的最复杂的操作之一.

早期版本中, flush方法的调用是在commit之后, 而且这个方法是在objectstore这个当前线程中隐式呈现的. 在0.1版本中, 完全没有必要调用Session.add(), 也没有完全明确的Session概念. 用户所要做的步骤仅仅是创建映射对象, 在查询过程中更新已存在对象的状态(同时查询本身也是被各自映射的对象之间调用的), 然后通过objectstore.commit命令将所有的变更持久化到数据库. 操作集合的对象池是全局的, 在当前线程中起效.

objectstore.commit模型早期很快吸引到一批用户, 但这个死板的模型很快就碰壁了. 当前的SQLAlchemy新用户可能需要为Session定义一个工厂, 可能的注册过程, 以及需要将所有的对象组织到一个session中哀叹不已, 但是当前这种做法比早期整个系统完全不明确的情况更可取. 0.1版本中的便利的使用模式在当前的SQLAlchemy中依然普遍存在, 比如通常在当前线程中注册Session.

Session这个对象是在0.2版本中引入的, 模仿的是Hibernate中的Session对象. 这个版本的特点是开始支持事务控制, Session可以通过begin方法加入一个事务, 当调用commit方法的时候事务结束. objectstore.commit方法被重命名为objectstore.flush, 同时可以在任何时刻创建新的Session. Session本身从UnitWork中分离出来, UnitWork仍然是一个私有对象, 负责实际的刷新动作.

当刷新动作开始被用户显示调用, 0.4版中引入autoflush的概念, 这意味着flush在每次查询操作之前先刷新. 这样做的好处是查询操作能查询到内存中所要查询对象的准确状态, 因为所有的变更都已经发送过. 早期SQLAlchemy不含此特性, 因为那时常用模式就是刷新会永久的提交改变. 但是随着自动刷新的引入, 这个过程被另一个特性(事务型session)完成, 从而可以在事务中自动启动一个Session, 直到用户调用显式commit为止. 这个特性使得flush方法再也不用提交它刷新的数据, 并可以安全调用. 现在Session可以在内存状态和SQL查询状态间实时同步, 在显式调用commit之前, 不会持久化到数据库. 事实上, java中的hibernate也是这样做的, 然而SQLAlchemy是基于Python的Storm ORM, 在0.3中引入的.

0.5版本中引入了事务后过期(post-transaction expiration), 从而支持更多的事务操作; 当commit或rollback操作之后, 默认Session中的所有状态都将被清除. 在随后的SQL重新查询数据或者当上下文中的其他事务访问过期对象的属性的时候再重新生成. 起初, SQLAlchemy建立在不论什么时候, 查询操作都要尽可能少的发送到数据库. 也正是这个原因, 在提交时消除的行为变得缓慢。然而，它成功解决了Session中包含过期数据的问题，使它可以在事务后用一种简单的方式加载新的数据，而不需要重新构建所有已经加载的对象。早期, 这个问题似乎无法得到合理的解决. 因为什么时候Session认定当前数据时老旧数据的问题并不明确, 因此在下一次访问的时候会生成新的SELECT集合, 这样做的代价是昂贵的. 一旦将Session应用到一个"总在事务中"的模型时, 事务端的重点就自然成为了数据消除，因为高度隔离的事务的本质就是它直到提交或回滚都看不到新的数据. 当然, 不同的数据库有不同的事务隔离配置, 甚至完全没有事务. SQLAlchemy的消除模型完全可以接受这些使用模式，开发人员只需要清楚，低隔离层次可能在多个回话共享同一行时，在一个会话中暴露未隔离的改变。这和直接使用两个数据库连接时发生的情况没什么不同。

## Session概览
下图展示了一个Session和它处理的对象结果之间的关系:
![Session overview](http://p3euxxfa8.bkt.clouddn.com/session_overview.png)

面向外层的是Session和用户定义的类, 每个用户定义的对象都是一个映射类的实例. 这里我们可以看到每一个映射对象保持着对SQLAlchemy的InstanceState结构的引用, 这个对象记录了ORM的状态，包括即将发生的属性改变和属性消除状态。前面“映射剖析”章节讨论的属性instrumentation，InstanceState是在其中的实例级部分——与在类级的ClassManager相对应（前面讲过，映射类及其实例之间是对称的，行为有某种对应关系——译者注）。它代表和类关联的AttributeImpl对象，维护映射对象的字典的状态（即Python的__dict__属性）。

## State Tracking
IdentityMap是数据库ID和InstanceState对象之间的映射, 这些对象有着persistent的数据库ID. 默认的实现是, IdentityMap和InstanceState一起通过当指向一个实例的所有的强引用都删除后, 将这个实例也删除的方式管理自己的大小, 这和Python的WeakValueDictionary的工作方式是一样的. Session对所有标记为dirty或deleted的对象，以及标记为new的pending对象，通过创建到这些对象的强引用来保护这些对象免于垃圾回收。所有的强引用都会在刷新操作后丢弃.

InstanceState同时在维护一个特定的对象的属性"改变了什么"这一重要任务发挥着作用. 通过在新值传入之前, 使用一个 move-on-change系统, 将特定属性的"旧值"保存到committed_state字典中, 刷新时, 通过对比两个字典不同的值来确定哪些属性发生了改变.

对于集合, 一个和InstrumentedAttribute/InstanceState系统相匹配的单独的包来保留一组特定包的更改, 常见的Python类如set，list，dict都在使用前进行继承并根据历史跟踪的增变方法进行扩展。集合系统在0.4版本修订为可扩充的，可以在任何类似集合的对象上使用。

# 工作单元
Session提供的flush方法将它的工作移交给unitofwork这个分离的模块. 如前所说, 刷新动作可能是SQLAlchemy中最复杂的操作.

工作单元的任务是将一个特定Session中所有挂起状态移到数据库, 将session中new, dirty, deleted的数据集合清空. 一旦完成, session在内存中的状态个事务中的状态就一致了(*此处存疑*). 这个过程中主要的难点是如何确定持久化的步骤, 并以正确的顺序运行. 这包括一系列的插入, 更新和删除操作. 删除操作还要考虑递归删除相关记录的结果; 确保更新操作只更新那些确实有过修改的字段; 当有新的主键生成的时候, 将主键的状态同步到相关的外键;确保在保证速度的情况下, 让插入操作按照记录添加到session中的顺序插入; 以及确保更新和删除顺序操作以防止死锁.

## 历史
工作单元最开始被设计成为一种特别紊乱的系统, 开发过程可以类比为在一个森林中, 在没有地图的情况下找到一条路. 早期的bug和缺失的行为在后续的补丁中得到修复. 虽然在0.5版本中的一些重构改善了某些问题, 但是直到0.6版本整个工作单元才得以重构, 这个时候才成为有着稳定的, 易于理解且有着数以百计的测试用例. 经过数周的考虑, 一种基于一致数据结构的新方案提了出来. 在此基础上, 重写工作单元只花了几天. 当然, 已有版本提供的交叉验证对新实行的完成也很有帮助. 这也说明, 在版本更替中, 第一个版本不论多么糟糕, 只要它能工作, 就有价值, 同时还说明对子系统的完全重构常常是那些难于开发的系统开发工作的一部分.

## 拓扑排序
工作单元的设计关键是将每一步作为一个节点组成一个数据结构, 这种设计被称为命令模式. 在这个数据结构中的一系列命令依据拓扑排序组织着. 拓扑排序基于局部排序, 即只需要关键的点排序即可. 下图展示了拓扑排序.
![Topological sort](http://p3euxxfa8.bkt.clouddn.com/Topological_sort.png)

工作单元基于持久化命令间必须的先后关系构造偏序。这些命令然后经过拓扑排序后按顺序调用.  判断哪个命令优先级更高, 哪个命令是派生命令的基础是两个映射对象之间的relationship关系. 通常来说, 一个映射对象是基于另一个对象的, 在mapper中的表现就是有一个外键. 在多对多关系中是同样的规则, 但是在此主要考虑一对多和多对一的情况. 外键依赖主要是为了防止出现违反依赖约束的情况出现, 但是同样主要的是, 这个顺序在多数平台上, 只有当插入情况出现时才会生成对应的主键, 在删除过程中, 使用的是相反的顺序. 因为当引用不存在时, 外键也不会存在.

在所展现的依赖结构中, 拓扑排序在两个级别上发挥着作用, 第一个层次基于Mapper间的依赖将持久化步骤入栈, 第二个层次将这些栈分成更小的批, 来处理循环或者自引用的表. 下图展示了插入User对象后接着插入Address对象的情形. 其中一个中间的步骤将新生成的User主键值拷贝到每个Address对象的user_id外键列.
![ Organizing objects by mapper](http://p3euxxfa8.bkt.clouddn.com/markdown-img-paste-20180111152411796.png)

在每个映射器的排序情况中，可以对任意数量的用户和地址对象进行刷新，而不会对步骤的复杂性产生影响，或者需要考虑多少“依赖项”。

排序的第二个层次是在单个mapper的范围内基于对象间的直接依赖组织持久化步骤. 这种情况出现的最简单的一种情形就是, 有一个包含了到自身的外键依赖的表. 表中的特定行需要在同一个表中引用它的另一个行之前插入. 另一个例子是一组有循环引用(reference cycle)的表：表A引用表B，表B引用表C，表C又引用表A。一些A的对象必须要在其他对象之前插入，才能允许B和C的对象也插入进来。一个引用自身的表是循环引用的一个特例。

为了确定哪些操作在聚合操作依然可以保留, 对每个Mapper桶上，和Mapper桶分解成的对象的命令的庞大集合，在mapper间存在的依赖集上应用环路检测算法，相关算法可以在[Guido Van Rossum's blog](http://neopythonic.blogspot.com/2009/01/detecting-cycles-in-directed-graph.html) 找到想似的版本. 涉及到环路的桶就被分解成对象的操作，通过将新的依赖规则从每个对象的桶加入每个mapper的桶，将对象的操作混入mapper的操作的集合。

桶结构的原理是，它允许尽可能多的对共同的语句进行批处理，既减少了Python中需要的步骤数，又可以和DBAPI进行更多的有效交互。有时候用一个Python方法调用就可以执行上千条语句。只有当mappper间的循环引用存在时，才会使用更昂贵的单个对象依赖的模式，但也只是在对象图中需要的部分才使用。

# 结论
SQLAlchemy从诞生之初就有很高的目标，想成为功能最丰富、最通用的数据库工具。它做到了这一点，并且一直将关注点放在关系型数据库上，认识到用深度、透彻的方式支持关系数据库的实用性是一项大的事业。甚至在现在，我们还不断发现这个事业的范围比以前想象的要大。

为了从每个领域的功能中提取最有价值的东西，SQLAlchemy打算使用基于组件的方法，提供了很多不同的模块单元，应用程序可以单独使用或是组合起来使用。这个系统的创建、维护和交付都一直是很有挑战的。

SQLAlchemy打算缓慢发展，这是基于一个理论——系统地、有基础地构建稳定的功能最终会比没有基础地快速发布新功能更有价值。SQLAlchemy用了很长的时间构建出一个一致的、文档齐全的用户故事。但在这个过程中，底层的架构一直领先着一步，导致在一些情况下会出现“时间机器”效应，新功能可以几乎在用户需要它们之前添加进来。

Python语言是一个很好的宿主语言（如果有些挑剔的话，特别是在性能方面），语言的一致性和极大开放的运行时模型让SQLAlchemy可以比用其他语言写的类似产品有更好的用户体验。

SQLAlchemy项目希望Python在尽可能广的领域得到广泛深入接受，并且关系数据库的使用也一直生机勃勃。SQLAlchemy的目标是要展示关系数据库，Python，以及经过充分考虑的对象模型都是非常有价值的开发工具。
