:banner: banners/javascript.jpg

.. highlight:: javascript

.. default-domain:: js

==========
Javascript
==========

控件(Widgets)
=======

.. class:: Widget

    来自于 ``web.Widget``, 所有可视组件的基类.
    它与MVC视图一致,并且提供一些服务来简化一个页面的区域的处理:

    * 处理控件的父/子关系
    * 提供有安全特性的扩展生命周期管理 (比如自动在一个父控件被销毁的时候销毁子控件)
    * 使用 :ref:`qweb <reference/qweb>` 自动渲染
    * 主干兼容的快捷方式

DOM源(DOM Root)
--------

:class:`~Widget` 对由控件的DOM源有形化的页面的部分负责.

一个控件的DOM源可以通过两个属性来使用:

.. attribute:: Widget.el

    作为控件的源而设置的原始的DOM元素

.. attribute:: Widget.$el

    jQuery对 :attr:`~Widget.el` 的封装

有两个主要的方法来定义和生成这个DOM源:

.. attribute:: Widget.template

    应该被设置为 :ref:`QWeb template <reference/qweb>` 的名字.
    如果设置了, 模板会在控件初始化之后开始之前被渲染. 模板生成的源元素会被设置为控件的DOM源.

.. attribute:: Widget.tagName

    如果控件没有模板定义那么就会被使用. 默认是 ``div``,
    会被作为标签名来生成DOM元素从而设置为控件的DOM源.
    也可使用如下的属性来进一步自定义这个生成的DOM源:

    .. attribute:: Widget.id

        被用于在DOM源上生成一个 ``id`` 属性.

    .. attribute:: Widget.className

        被用于在DOM源上生成一个 ``class`` 属性.

    .. attribute:: Widget.attributes

        把属性名映射(对象)到属性值上Mapping. 每个键:值对会被设置为生成的DOM源的DOM属性.

    在模板在控件上被指定的情况下,以上都不会被使用.

DOM源也可以通过复写来程序性的定义

.. function:: Widget.renderElement

    渲染并设置控件的DOM源. 默认的实现会渲染一个模板集合或者生成一个上面描述的元素,
    并且会在结束时调用 :func:`~Widget.setElement` .

    任何对 :func:`~Widget.renderElement` 的复写且没有调用它的 ``_super``
    **必须** 调用 :func:`~Widget.setElement` ,不管它生成的是什么,不然控件的行为就是未定义的.

    .. note::

        默认的 :func:`~Widget.renderElement` 可以被重复的调用,
        它会 *替换* 之前的DOM源(使用 ``replaceWith``). 然而,
        这要求控件正确的设置和解除设置它的事件(和子控件).
        一般来说, :func:`~Widget.renderElement` 不要被重复调用除非控件需要的这特性.

使用控件(Using a widget)
''''''''''''''

一个控件的生命周期有3个主要阶段:

* 控件实例的生成和初始化

  .. function:: Widget.init(parent)

       控件的初始化方法,同步的,可以被复写为从控件的生成者/父接收更多参数

       :param parent: 新的控件的父,被用于处理自动的销毁和事件的传播. 对于没有父的控件,其可以是 ``null``.
       :type parent: :class:`~Widget`

* DOM注入和启动, 这是通过调用下面中的任一个完成的、:

  .. function:: Widget.appendTo(element)

    渲染控件并把它作为目标的最后一个子插入,使用 `.appendTo()`_

  .. function:: Widget.prependTo(element)

    渲染控件并把它作为目标的第一个子插入,使用 `.prependTo()`_

  .. function:: Widget.insertAfter(element)

    渲染控件并把它作为目标的前项插入, 使用 `.insertAfter()`_

  .. function:: Widget.insertBefore(element)

    渲染控件并把它作为目标的后项插入,使用 `.insertBefore()`_

  所有的这些方法接收来相应的jQuery可以接收的方法
  (CSS 选择器, DOM 节点或者 jQuery 对象). 他们都返回一个 deferred_ 并且有如下3个任务:

  * 使用 :func:`~Widget.renderElement` 渲染控件的根元素
  * 使用匹配的jQuery方法插入控件的根元素
  * 启动控件,并且返回启动的结果

    .. function:: Widget.start()

        在控件被注入到DOM后,控件的异步启动,
        一般用于执行异步的RPC调用来提取用于控件完成其工作的远程数据.

        必须返回一个 deferred_ 来指示其工作何时完成了.

        一个控件在 :func:`~Widget.start` 方法已经被完成之前 *不保证* 正确的工作.
        控件的父/创建者必须等待一个控件完全的启动才能与其交互

        :returns: deferred_ 对象

* 控件的销毁和清除

  .. function:: Widget.destroy()

    销毁控件的子, 解除它的时间并且从DOM移除它的源. 在控件的父被销毁时自动调用,
    如果控件没有父或者控件的父保留而控件本身被移除的情况下,必须被明确地调用.

    控件的销毁自动地从其父解除链接.

关联至控件的销毁是一个重要的实用方法:

.. function:: Widget.alive(deferred[, reject=false])

    RPC和销毁的一个明显的问题是在控件被销毁或者在其销毁之后一个RPC调用可能需要消耗很长的时间来执行并返回,
    这是试着执行控件之上的操作处于损坏/无效状态.

    这是错误和奇怪行为的一个常见根源、.

    :func:`~Widget.alive` 可以被用于封装RPC调用,
    保证了在调用结束应该被执行的操作只会在这个控件依然存在的情况下才会被执行::

        this.alive(this.model.query().all()).then(function (records) {
            // would break if executed after the widget is destroyed, wrapping
            // rpc in alive() prevents execution
            _.each(records, function (record) {
                self.$el.append(self.format(record));
            });
        });

    :param deferred: 一个需要封装的 deferred_ 对象
    :param reject: 默认情况下, 如果RPC的调用在控件被销毁的情况下返回,
                   返回的 deferred_ 处于丢弃状态(既不解析也不拒绝).
                   如果 ``拒绝`` 被设置为 ``真``, 那么 deferred_ 会被拒绝.
    :returns: deferred_ 对象

.. function:: Widget.isDestroyed()

    :returns: ``真`` 如果控件正在被或者已经被销毁了, 其他情况下为 ``假``

访问DOM内容(Accessing DOM content)
'''''''''''''''''''''

因为一个控制只对其下面的DOM源负责,这里有一个选择控件的DOM的次区域的快捷方式:

.. function:: Widget.$(selector)

    对DOM源应用作为其特定参数的CSS选择器::

        this.$(selector);

    功能上一致于::

        this.$el.find(selector);

    :param String selector: CSS 选择器
    :returns: jQuery 对象

    .. note:: 这个助手方法相似于 ``Backbone.View.$``

重设DOM源(Resetting the DOM root)
''''''''''''''''''''''

.. function:: Widget.setElement(element)

    重设控件的DOM源至给定的元素, 也会处理重设各种DOM源的别名,或者解除设置和重设委托事件.

    :param Element element: 设置为控件的DOM源的一个DOM元素或者jQuery对象

    .. note:: 应该与  `Backbone's setElement`_ 保持最大的兼容

DOM事件处理(DOM events handling)
-------------------

一个控件一般需要响应用户在页面的相应区域的的动作. 这需要DOM元素的绑定事件.

在结束时, :class:`~Widget` 提供了快捷方式:

.. attribute:: Widget.events

    事件是一个事件选择器至一个回调的映射(由一个空格隔开的一个事件名称和一个可选的CSS选择器).
    回调可以是控件方法或者一个函数对象的名字. 在这两种情况中, ``this`` 会被设置到控件中::

        events: {
            'click p.oe_some_class a': 'some_method',
            'change input': function (e) {
                e.stopPropagation();
            }
        },

    选择器用于jQuery的事件委派( `event delegation`_ ), 回调只会在DOM源的后裔和选择器
    ( elector\ [#eventsdelegation]_ )匹配的时候被触发. 如果选择器被忽视了
    (只指定了一个事件名称), 事件会被直接设置在控件的DOM源.

.. function:: Widget.delegateEvents

    这个方法管理 :attr:`~Widget.events` 和DOM的绑定. 它会在设置了控件的源之后自动的被调用.

    可以复写来设置更多相对于 :attr:`~Widget.events` 映射所允许的复杂的事件,
    但是其父应该一直被调用(否则 :attr:`~Widget.events` 不会被正确的处理).

.. function:: Widget.undelegateEvents

    当控件被销毁或者DOM源被重设时,这个方法管理 :attr:`~Widget.events` 从DOM源的解绑定,
    来避免 "幽灵(phantom)" 事件.

    在复写 :func:`~Widget.delegateEvents` 时,其应该被复写来解除设置任意事件.

.. note:: 这个行为应该与 `Backbone's delegateEvents`_ 兼容, 除了不接收任何参数之外.

子类控件(Subclassing Widget)
------------------

:class:`~Widget` 以标准方式被分为子集 (通过 :func:`~Class.extend` 方法),
并且提供了一些抽象的属性和具体的方法(你可能也可能不想复写). 创建一个子类看起来是这样::

    var MyWidget = Widget.extend({
        // 在渲染对象时使用的 QWeb 模板
        template: "MyQWebTemplate",
        events: {
            // 事件绑定示例
            'click .my-button': 'handle_click',
        },

        init: function(parent) {
            this._super(parent);
            // 插入在渲染之前需要执行的代码,
            // 用于对象的初始化
        },
        start: function() {
            var sup = this._super();
            // 渲染发布的初始化代码,这种情况下

            // 允许多路复用的延迟对象
            return $.when(
                // 从父类传输异步信号
                sup,
                // 返回自己的异步信号
                this.rpc(/* … */))
        }
    });

这个新的类可以以如下的方式被使用::

    // 创建实例
    var my_widget = new MyWidget(this);
    // 渲染和插入DOM
    my_widget.appendTo(".some-div");

在这两行被执行之后(并且 :func:`~Widget.appendTo` 返回的任何约定都已经被解析了,如果需要的话),
控件就已经准备好被使用了.

.. note:: 插入方法会启动控件本身,并且会返回 :func:`~Widget.start()` 的结果.

          如果由于某些原因你不嫌调用这些方法,你会不得不先先调用控件上的 :func:`~Widget.render()` ,
          然后把它插入到你的DOM中并且启动它.

如果控件不再被需要了(因为它是瞬态的), 只需要简单地结束它即可::

    my_widget.destroy();

会解绑定所有的DOM事件, 从DOM移除控件的内容并且销毁所有控件的数据.

开发指南(Development Guidelines)
----------------------

* 标识 (``id`` 属性) 应该被避免. 在通用应用和模块中, ``id`` 限制了组件的复用性并且让代码更脆弱.
  大部分情况下, 可以用空, 类或者一个DOM节点或jQuery元素的参考来代替他们.

  如果一个 ``id`` 是必须的 (因为第三方的库需要一个),
  那么这个id应该使用 ``_.uniqueId()`` 部分地生成,比如::

      this.id = _.uniqueId('my-widget-')
* 避免可预见的/常见的CSS类名. 像 "content" 或者 "navigation" 的类名可能匹配想要的意思/语法,
  但是很可能其它开发者有同样的需求, 创建一个冲突的名字和意想不到的行为.
  常见的通用类名应该有前缀,比如他们属于的组件的名称
  (创建 "非正式的(informal)" 命名空间, 就像在C语言中或者Objective-C中).
* 应该避免全局选择器. 因为一个组件可能在一个页面中被使用几次(一个例子是Odoo中的dashboards),
  查询应该被限制到一个给定组件的范围. 未被过滤的选择器,比如
  ``$(selector)`` 或者 ``document.querySelectorAll(selector)``
  一般会导致无意的或者不正确的行为.  Odoo Web的 :class:`~Widget` 有一个属性提供了它的DOM源的
  (:attr:`~Widget.$el`), 和一个至选择节点 (:func:`~Widget.$`) 的快捷方式.
* 更常见的, 永远不要假设你的组件拥有或者控制任何超过其自身的 :attr:`~Widget.$el`
* html 模板/渲染应该使用QWeb除非完全没必要.
* 所有的交互组件(显示信息到屏幕或者截取DOM事件的组件)必须继承自 :class:`~Widget` 并且被正确的实现,
  使用其自己的API和生存周期.

.. _.appendTo():
    http://api.jquery.com/appendTo/

.. _.prependTo():
    http://api.jquery.com/prependTo/

.. _.insertAfter():
    http://api.jquery.com/insertAfter/

.. _.insertBefore():
    http://api.jquery.com/insertBefore/

.. _event delegation:
    http://api.jquery.com/delegate/

.. _Backbone's setElement:
    http://backbonejs.org/#View-setElement

.. _Backbone's delegateEvents:
    http://backbonejs.org/#View-delegateEvents

.. _deferred: http://api.jquery.com/category/deferred-object/

远程过程调用协议(RPC)
=====

为了显示和交互数据,调用Odoo服务是很必要的. 这是使用 :abbr:`RPC <Remote Procedure Call>` 来调用的.

Odoo 网络服务提供2个主要的API来处理这些: 一个低层次的JSON-RPC,其基于与Odoo网络的Python部分的API通信
(而且还有你的模块, 如果你有Python部分地话)和一个高层次的API,在那之上允许你的代码直接与高层次的Odoo模型沟通.

所有的网络API都是异步的( :ref:`asynchronous <reference/async>` ). 因此,
他们都会返回 Deferred_ 对象 (不论他们是否解析了这些值). 理解这些是如何工作的对接下来的事情很重要.

高层次的 API: 调用Odoo 模型
-------------------------------------------

访问Odoo对象方法(通过XML-RPC使得从服务器端可用)是通过 :class:`Model`.
它通过两个主要的方法映射至Odoo服务器对象, :func:`~Model.call` (来自于 ``web.Model``) 和 :func:`~Model.query`
(来自于 ``web.DataModel``, 只在后端客户端可用).

:func:`~Model.call` 是一个至相应Odoo服务对象方法的直接映射. 它的使用类似于Odoo Model API,
但有3点不同:

* 接口是异步的( :ref:`asynchronous <reference/async>` ), 所以不是直接返回结果,
  RPC方法调用会返回 Deferred_ 实例,它们会自解析至匹配的RPC调用的结果.

* 因为 ECMAScript 3/Javascript 1.5 并没有任何类似于 ``__getattr__`` 或者 ``method_missing`` 的特性,
  因此需要一个明确的方法来分派RPC方法.

* 没有池的概念, 模型的代理协议在需要的地方被实例化, 而不是取自其他(或者可以说全局)的对象 ::

    var Users = new Model('res.users');

    Users.call('change_password', ['oldpassword', 'newpassword'],
                      {context: some_context}).then(function (result) {
        // 对 change_password 结果进行处理
    });

:func:`~Model.query` 是一个至查找的构建者类型的接口(``search`` + ``read`` 在 Odoo RPC 术语中).
它返回一个 :class:`~odoo.web.Query` 对象,其是不可变的但是允许从第一个来构建新的 :class:`~odoo.web.Query`
实例, 添加新的属性或者编辑父对象的属性 ::

    Users.query(['name', 'login', 'user_email', 'signature'])
         .filter([['active', '=', true], ['company_id', '=', main_company]])
         .limit(15)
         .all().then(function (users) {
        // 对用户的记录进行操作
    });

查询只会在在调用查询序列化方法时被真正的执行了, :func:`~odoo.web.Query.all` 和
:func:`~odoo.web.Query.first`. 这些方法会在每次调用的时候执行一个新的RPC调用.

因为这个原因, 很可能保持 "中介的(intermediate)" 查询并不同地使用它们/添加新的特性.

.. class:: Model(name)

    .. attribute:: Model.name

        这个对象绑定的模型的名字

    .. function:: Model.call(method[, args][, kwargs])

         使用提供的位置和关键字参数,调用当前模型的 ``method`` 方法.

         :param String method: 使用rpc调用 :attr:`~Model.name` 的方法
         :param Array<> args: 可选的传递给方法的位置参数
         :param Object<> kwargs: 可选的传递给方法的关键字参数
         :rtype: Deferred<>

    .. function:: Model.query(fields)

         :param Array<String> fields: 在查找时需要提取的字段的列表
         :returns: 一个代表了执行查找的 :class:`~odoo.web.Query` 对象

.. class:: odoo.web.Query(fields)

    第一个方法的集合是"提取(fetching)"的方法. 他们使用他们调用的对象的内部数据来执行RPC查询.

    .. function:: odoo.web.Query.all()

        提取当前 :class:`~odoo.web.Query` 对象查找的结果.

        :rtype: Deferred<Array<>>

    .. function:: odoo.web.Query.first()

       提取当前 :class:`~odoo.web.Query` 的 **第一个** 结果, 或者 ``空`` 如果当前的
       :class:`~odoo.web.Query` 没有任何结果的话.

       :rtype: Deferred<Object | null>

    .. function:: odoo.web.Query.count()

       提取当前 :class:`~odoo.web.Query` 会检索的记录的数量.

       :rtype: Deferred<Number>

    .. function:: odoo.web.Query.group_by(grouping...)

       使用第一个特定的组参数来提取查询的组

       :param Array<String> grouping: 列出服务器需要的组的级别. 分组可以是一个阵列或者变量.
       :rtype: Deferred<Array<odoo.web.QueryGroup>> | null

    第二个方法的集合是 "存取器(mutator)" 方法, 他们使用相关的参数化或者替换了的方法生成一个
    **新的** :class:`~odoo.web.Query` 对象.

    .. function:: odoo.web.Query.context(ctx)

       添加提供的 ``ctx`` 至查询, 高于任何存在的内容

    .. function:: odoo.web.Query.filter(domain)

       给查询添加提供的域,这个域是和现存的查询域进行与运算的.

    .. function:: opeenrp.web.Query.offset(offset)

       给查询设置指定的偏移. 新的偏移会 *替换* 旧的.

    .. function:: odoo.web.Query.limit(limit)

       给查询设置提供的限制. 新的限制会 *替换* 旧的.

    .. function:: odoo.web.Query.order_by(fields…)

       使用提供的字段说明来复写模型的自然顺序. 与Django的 :py:meth:`QuerySet.order_by
       <django.db.models.query.QuerySet.order_by>` 表现很像:

       * 接收 1..n 字段名, 用重要性反向排序(第一个字段是第一个排序的键). 字段被作为字符串提供出来.

       * 一个字段指定了一个升序排列,除非它有减号 "``-``" 作为前缀,这种情况下字段被用于将序排列

       与Django的排序不同,包含有:缺少一个随机排序(``?`` 字段) 并且不能为排序 "向下钻取(drill down)" 进数据关系.

聚合(分组)(Aggregation (grouping))
''''''''''''''''''''''

Odoo有一个强有力的分组能力, 就是在他们递归时有点不一样, 并且级别 n+1 依赖于分组在级别n直接提供的数据,
并且 :py:meth:`odoo.models.Model.read_group` 工作的不像一个直觉容易想到的API.

Odoo 网络避免直接调用 :py:meth:`~odoo.models.Model.read_group` 来帮助调用 :class:`~odoo.web.Query` 的一个方法,
:py:meth:`它很像在SQLAlchemy中的一个方法 <sqlalchemy.orm.query.Query.group_by>`
[#terminal]_::

    some_query.group_by(['field1', 'field2']).then(function (groups) {
        // 对提取的组进行一些处理
    });

这个方法在提供了1..n字段(来分组)作为参数时是异步工作的,
但是它也可以不用任何字段而被调用(空的字段集合或者什么也没有). 在这种情况下, 它会返回一个 ``null``
而不是一个延迟对象(Deferred object).

当分组的标准来自于一个第三方并且可能是或者不是字段列表时(比如可能是个空列表),
这提供了两种方式来测试真正的子组的存在性(对比于执行一个记录常规查询的需求):

* 检查 ``group_by`` 的结果和两个完全分隔的代码路径::

      var groups;
      if (groups = some_query.group_by(gby)) {
          groups.then(function (gs) {
              // 组
          });
      }
      // 没有组

* 或者一个使用 :func:`when` 的强制插入值进延迟的能力的更一致的代码路径::

      $.when(some_query.group_by(gby)).then(function (groups) {
          if (!groups) {
              // 没有分组
          } else {
              // 分组, 即使没有组(组本身可以是一个空的阵列)
          }
      });

一个 (成功的) :func:`~odoo.web.Query.group_by` 结果是一个 :class:`~odoo.web.QueryGroup` 的阵列:

.. class:: odoo.web.QueryGroup

    .. function:: odoo.web.QueryGroup.get(key)

        返回组的属性 ``键``. 已知的属性有:

        ``grouped_on``
            其分组这个组的字段
        ``value``
            这个组的 ``grouped_on`` 的值
        ``length``
            组中记录的数量
        ``aggregates``
            组的一个 {字段: 值} 的聚合映射

    .. function:: odoo.web.QueryGroup.query([fields...])

        等价于 :func:`Model.query` 但是被预先过滤为值包含这个组中的记录.
        返回一个 :class:`~odoo.web.Query` , 其可以被进一步的进行所有的操作、.

    .. function:: odoo.web.QueryGroup.subgroups()

        返回一个延迟至低于这个 :class:`~odoo.web.QueryGroup` 的一个阵列

低级别API: Python端RPC调用 (Low-level API: RPC calls to Python side)
---------------------------------------

上面部分对于调用核心的Odoo代码(模型代码)很有用, 但是它对于调用Odoo网络的python部分就不行了.

针对这点, 一个低级别的 :class:`~Session` 对象的API (这个类来自于 ``web.Session``,
但是通过 ``web.session`` ,其一个实例通常都可用):  ``rpc`` 方法.

这个方法只是接受一个绝对路径(调用的JSON的绝对URL :ref:`route <reference/http/routing>` )
和一个到属性到值的映射(作为关键字参数传递给Python方法). 这个功能提取Python方法的返回值,并转换为JSON.

比如, 为了调用 :class:`~web.controllers.main.DataSet` 控制器的 ``resequence`` ::

    session.rpc('/web/dataset/resequence', {
        model: some_model,
        ids: array_of_ids,
        offset: 42
    }).then(function (result) {
        // 重新排列不会出错
    }, function () {
        // 在调用的时候发生了一个错误
    });

.. _reference/javascript/client:

网络客户端(Web Client)
==========

Javascript 模块系统综述
---------------------------------

一个新的模块系统 (灵感来自于requirejs) 已经被部署了. 它给Odoo版本8系统带来了很多的优势.

* 载入顺序: 依赖被保证了首先载入, 即使文件在文件包里面没有以正确的顺序被载入.
* 更容易把文件分割为更小的逻辑单元.
* 没有全局变量: 很容易理解.
* 能够实现检查每个依赖和从属. 这让重构变革更简单,有更低的风险.

它也有一些不足:

* 如果想要与odoo交互,文件被要求使用模块系统, 因为不同的对象只在模块系统中可用,而不是在全局变量中
* 不支持循环依赖.这是有道理的,但是它意味着使用者必须很小心.

这很明显是一个很大的改变并且会需要每个人采用新的习惯.比如,可变的odoo就不再存在了.
做事情的新方法就是来明确的导入你需要的模块,并且明确的声明你输出的对象.下面是个简单的例子::

    odoo.define('addon_name.service', function (require) {
        var utils = require('web.utils');
        var Model = require('web.Model');

        // do things with utils and Model
        var something_useful = 15;
        return  {
            something_useful: something_useful,
        };
    });

这个代码段展示了一个叫 ``addon_name.service`` 的模块.  它使用 ``odoo.define`` 函数定义.
``odoo.define`` 接收一个名字和一个函数作为参数:

* 名字是addon定义的名字和描述其目的的名字的连接.
* 函数是 javascript 模块被真正定义的地方. 它接收一个函数 ``require`` 作为第一个参数, 并且返回一些东西
  (或者不返回, 取决于它是否需要输出一些东西). ``require`` 函数被用于获得依赖的处理函数.
  在这种情况下, 它给出两个javascript模块,
  来自于 ``web`` 的称为 ``web.utils`` and ``web.Model`` 的addon,的一个处理函数.

想法是这样的,你定义你需要导入的(通过使用 ``require`` 函数) 和声明你需要输出的(通过返回一些东西).
网络客户端接下来会确保你的代码被正确的载入.

模块包含在一个文件中,但是一个文件可以定义几个模块(虽然最好是把他们放到单独的文件中).

每个模块都可以返回一个延迟(deferred). 在这种情况下, 模块被标记为只在延迟被解析时才载入,
并且它的值等同于解析的值. 模块可以被拒绝(解除载入). 这会在控制台被作为信息记录.

* ``缺失依赖(Missing dependencies)``:
  这些模块没有在页面中显示. 很可能JavaScript文件没有在页面中或者模块名是错误的
* ``失效的模块(Failed modules)``:
  检测到一个javascript错误
* ``拒绝的模块(Rejected modules)``:
  模块返回一个拒绝的延迟. 它 (和它的依赖模块) 没有被载入.
* ``拒绝的链接模块(Rejected linked modules)``:
  依赖于一个被拒绝模块的模块
* ``非载入的模块(Non loaded modules)``:
  依赖于一个缺失或者失效模块的模块


网络客户端结构(Web client structure)
--------------------

网络客户端文件被重构成了更小和更简单的文件.下面是当前文件结构的描述:

* ``framework/`` 文件夹包含所有的基本低级别模块. 这里的模块被认为是通用的. 他们中包含:

  * ``web.ajax`` 实现rpc调用
  * ``web.core`` 是一个通用模块. 它输出不同的有用的对象和函数,比如 ``qweb``, ``_t`` 或者总线.
  * ``web.Widget`` 包含控件类
  * ``web.Model`` 是一个 ``web.ajax`` 的抽象,来实现调用服务器的模型方法
  * ``web.session`` 是之前的 ``odoo.session``
  * ``web.utils`` 有用的代码段
  * ``web.time`` 每个时间关联的通用函数
* ``views/`` 文件夹包含所有的视图定义
* ``widgets/`` 是独立的控件

``js/`` 文件夹也包含一些重要的文件:

* ``action_manager.js`` 是ActionManager类
* ``boot.js`` 是真正实现模块系统的文件
* ``menu.js`` 是顶层菜单的定义
* ``web_client.js`` 用于根控件WebClient
* ``view_manager.js`` 包含ViewManager

另外两个文件是用于巡回的 ``tour.js`` 和 ``compatibility.js``.后一个文件是桥接老系统至新模块系统的兼容层.
在这里每一个模块名被处处至全局变量 ``odoo``. 在理论上,我们的addon应该不使用变量 ``odoo`` 来工作,
并且兼容模块可以安全的禁用掉.

Javascript约定(conventions)
----------------------

下面是一些javascript代码的基本约定:

* 在模块头部声明所有依赖.同样的, 他们应该按模块名字的字母顺序排序. 这让理解你的模块变得容易.
* 在底部声明所有的输出.
* 在每个模块的开始添加 ``use strict`` 声明
* 要给你的模块起合适的名字: ``addon_name.description``.
* 对类名使用首字母大写 (比如, ``ActionManager`` 定义在模块 ``web.ActionManager`` 中),
  其他的都小写(比如, ``ajax`` 定义在 ``web.ajax``).
* 在一个文件声明一个模块





Odoo网络客户端的测试(Testing in Odoo Web Client)
==========================

Javascript单元测试(Unit Testing)
-----------------------

Odoo网络包含同等重要的核心Odoo网络代码和你自己的javascript模块的单元测试. 在javascript方面,
单元测试是基于 QUnit_ 并且有很多助手和扩展来提供和Odoo更好的集成.

为了便于查看运行的样子, 找到 (或者启动) 一个网络客户端启用了的Odoo服务器, 并且导向 ``/web/tests``
这会展示运行选择器, 其胡列出所有的javascript单元测试的模块, 并且允许启动他们中的任何一个
(或者一次执行所有模块的所有javascript测试).

.. image:: ./images/runner.png
    :align: center

点击任何一个运行按钮会启动相应的绑定的 QUnit_ 运行测试:

.. image:: ./images/tests.png
    :align: center

写一个测试(Writing a test case)
-------------------

第一步是列出测试文件. 这是通过Odoo的显示的 ``test`` 键, 通过把javascript文件加入进去:

.. code-block:: python

    {
        'name': "Demonstration of web/javascript tests",
        'category': 'Hidden',
        'depends': ['web'],
        'test': ['static/test/demo.js'],
    }

并且生成一个相应的测试文件

.. note::

    不存在的测试文件会被忽略, 如果一个模块的所有的测试文件都被忽略了(找不到),
    测试运行程序会认为这个模块没有javascript测试.

在那之后, 刷新运行程序选择器会显示新的模块并允许运行其所有的测试(目前是0):

.. image:: ./images/runner2.png
    :align: center

接下来是生成一个测试用例::

    odoo.testing.section('basic section', function (test) {
        test('my first test', function () {
            ok(false, "this test has run");
        });
    });

在所有的测试助手和结构住进 ``odoo.testing`` 模块之后. Odoo 测试就会住进 :func:`~odoo.testing.section`,
其本身是一个模块的一部分. 区域的第一个参数是这个区域的名字, 第二个是这个区域的本体.

:func:`test <odoo.testing.case>`, 由 :func:`~odoo.testing.section` 提供至回调,
用于注册一个给定的测试情形,其会在测试运行程序真正的工作的时候运行. Odoo 网络测试情形在其内部使用标准的 `QUnit
assertions`_ .

启动测试运行程序在这种情况下回运行测试并显示相应的声明信息, 使用红色来指示运行失败的情况:

.. image:: ./images/tests2.png
    :align: center

修复测试(通过在声明中替换 ``false`` 为 ``true`` )
会让其通过测试:

.. image:: ./images/tests3.png
    :align: center

声明(Assertions)
----------

就像上面说明的, Odoo的网络测试使用 `qunit assertions`_. 他们在全局都可用
(所以他们可以不同参考任何东西就可被调用). 下面的列表是可用的:

.. function:: ok(state[, message])

    检查 ``state`` 是真(在javascript语境中)

.. function:: strictEqual(actual, expected[, message])

    检查真正的 (由一个测试的方法生成) 和预期的值是相同的(主要等同于 ``ok(actual
    === expected, message)`` )

.. function:: notStrictEqual(actual, expected[, message])

    检查真实的和预期的值是*不* 一直的(基本上等同于 ``ok(actual !== expected, message)`` )

.. function:: deepEqual(actual, expected[, message])

    深入比较真实值和预期值: 在容器(对象和阵列)中递归来确保他们有同样的键值/元素数量和值匹配.

.. function:: notDeepEqual(actual, expected[, message])

    :func:`deepEqual` 的反向操作

.. function:: throws(block[, expected][, message])

    检查, 在被调用时, ``block`` 扔出一个错误. 选择性的证实错误是否符合 ``预期`` .

    :param Function block:
    :param expected: 如果是一个正则表达式,检查抛出的错误消息匹配这个表达式.
                     如果是一个错误类型,检查抛出的错误是否是其中一类.
    :type expected: Error | RegExp

.. function:: equal(actual, expected[, message])

    使用 ``==`` 操作符和它的强制规则,检查 ``actual`` 和 ``expected`` 是稀疏等价的.

.. function:: notEqual(actual, expected[, message])

    :func:`equal` 的反向操作

获得Odoo实例(Getting an Odoo instance)
------------------------

Odoo 实例是大部分Odoo网络模块行为(函数,对象, …)被访问的基础. 因此, 测试框架自动构建一个,
并且载入被测试的模块和所有它的依赖.这个新的实例被作为第一个位置参数提供给你的测试情形.
让我们通过添加javascript代码(不是测试代码)至测试模块来观察一下:

.. code-block:: python

    {
        'name': "Demonstration of web/javascript tests",
        'category': 'Hidden',
        'depends': ['web'],
        'js': ['static/src/js/demo.js'],
        'test': ['static/test/demo.js'],
    }

::

    // src/js/demo.js
    odoo.web_tests_demo = function (instance) {
        instance.web_tests_demo = {
            value_true: true,
            SomeType: instance.web.Class.extend({
                init: function (value) {
                    this.value = value;
                }
            })
        };
    };

接下来添加一个新的测试情形, 其只是简单的检查 ``instance`` 包含我们在模块中创建的所有的预期成员::

    // test/demo.js
    test('module content', function (instance) {
        ok(instance.web_tests_demo.value_true, "should have a true value");
        var type_instance = new instance.web_tests_demo.SomeType(42);
        strictEqual(type_instance.value, 42, "should have provided value");
    });

DOM 简记(Scratchpad)
--------------

就像在更宽广的客户端中, 任意访问文档内容在测试时是不被鼓励的. 但是DOM访问仍然需要,
比如在测试他们之前初始化 :class:`widgets <~odoo.Widget>` .

因此, 一个测试情形获得一个DOM简记作为它的位置参数, 在一个jQuery实例中.
那个简记在每个测试前会被完全清除, 并且只要不是在简记之外,你的代码就可以做任何它想做的::

    // test/demo.js
    test('DOM content', function (instance, $scratchpad) {
        $scratchpad.html('<div><span class="foo bar">ok</span></div>');
        ok($scratchpad.find('span').hasClass('foo'),
           "should have provided class");
    });
    test('clean scratchpad', function (instance, $scratchpad) {
        ok(!$scratchpad.children().length, "should have no content");
        ok(!$scratchpad.text(), "should have no text");
    });

.. note::

    简记的顶层元素没有被清除,测试情形可以添加文本或者DOM的子但是不应该替换 ``$scratchpad`` 本身.

载入模板(Loading templates)
-----------------

为了避免相应的处理花销, 默认情况下模板是不会被载入到QWeb中的. 如果你需要渲染比如需要QWeb模板的控件,
你可以请求他们通过 :attr:`~TestOptions.templates` 选项载入至 :func:`test case
function <odoo.testing.case>`.

这会在运行测试情形时自动的载入所有的相关模板进入实例的QWeb:

.. code-block:: python

    {
        'name': "Demonstration of web/javascript tests",
        'category': 'Hidden',
        'depends': ['web'],
        'js': ['static/src/js/demo.js'],
        'test': ['static/test/demo.js'],
        'qweb': ['static/src/xml/demo.xml'],
    }

.. code-block:: xml

    <!-- src/xml/demo.xml -->
    <templates id="template" xml:space="preserve">
        <t t-name="DemoTemplate">
            <t t-foreach="5" t-as="value">
                <p><t t-esc="value"/></p>
            </t>
        </t>
    </templates>

::

    // test/demo.js
    test('templates', {templates: true}, function (instance) {
        var s = instance.web.qweb.render('DemoTemplate');
        var texts = $(s).find('p').map(function () {
            return $(this).text();
        }).get();

        deepEqual(texts, ['0', '1', '2', '3', '4']);
    });

异步情形(Asynchronous cases)
------------------

测试情形的例子目前都是同步的, 他们从第一行处理至最后一行, 一旦最后一行处理完成了,那么测试也就完成了.
但是web客户端非常多的 :ref:`asynchronous code
<reference/async>`, 因此测试情形需要是懂得处理异步的.

这通过从测试情形的回调返回一个 :class:`deferred <Deferred>` 来实现::

    // test/demo.js
    test('asynchronous', {
        asserts: 1
    }, function () {
        var d = $.Deferred();
        setTimeout(function () {
            ok(true);
            d.resolve();
        }, 100);
        return d;
    });

这个例子也使用了选项参数( :class:`options parameter <TestOptions>` )
来指定测试情形应该预期的声明的数量, 如果更少或者更多的声明被指定了,测试情形会被记为失败.

异步测试情形 *必须* 指定他们会运行的声明的数目. 这允许更容易的捕获特定情形,
比如测试架构没有对异步操作进行警告.

.. note::

    异步测试情形同样有2秒的超时限制: 如果测试在2秒内没有完成,它会被认为是失效的.
    这个就意味着测试不会被解析. 这个超时 *只* 对测试本身有效, 对设置和拆解程序无效.

.. note::

    如果返回的延迟被拒绝了, 测试会失效,除非 :attr:`~TestOptions.fail_on_rejection` 被设置为 ``false``.

远程过程调用协议(RPC)
------

异步测试情形的一个重要的子集就是需要执行RPC调用的测试情形.

.. note::

    因为他们是异步情形的一个子集, RPC情形必须提供一个有效的 :attr:`assertions count
    <TestOptions.asserts>`.

为了启用模拟的RPC, 设置 :attr:`rpc option <TestOptions.rpc>` 为 ``mock``.
这会给测试情形回调添加第三个参数:

.. function:: mock(rpc_spec, handler)

    可以被用于两个不同的方式,取决于第一个参数的具体值:

    * 如果它匹配模式 ``model:method`` (本质上说,如果它包含一个冒号)
      调用会,比如通过 :func:`odoo.web.Model.call` 运行,直接给Odoo服务器设置虚拟的RPC调用(通过 XMLRPC) .

      在这种情形中, ``处理函数(handler)`` 应该是一个接收两个参数 ``args`` 和 ``kwargs`` 的函数,
      匹配相应的服务器端的参数并且应该只是返回值,就像他们是被Python XMLRPC处理器函数处理的样子::

          test('XML-RPC', {rpc: 'mock', asserts: 3}, function (instance, $s, mock) {
              // set up mocking
              mock('people.famous:name_search', function (args, kwargs) {
                  strictEqual(kwargs.name, 'bob');
                  return [
                      [1, "Microsoft Bob"],
                      [2, "Bob the Builder"],
                      [3, "Silent Bob"]
                  ];
              });

              // actual test code
              return new instance.web.Model('people.famous')
                  .call('name_search', {name: 'bob'}).then(function (result) {
                      strictEqual(result.length, 3, "shoud return 3 people");
                      strictEqual(result[0][1], "Microsoft Bob",
                          "the most famous bob should be Microsoft Bob");
                  });
          });

    * 另一方面, 如果它匹配一个绝对路径(比如 ``/a/b/c``) 他会虚拟一个 JSON-RPC 调用至网络客户端控制器,
      比如 ``/web/webclient/translations``. 在这种情况中,
      处理函数接收一个单独的包含所有的提供给JSON-RPC的参数的 ``params`` 参数.

      就像之前的,这个处理函数应该只是返回结果的值就像被原生的JSON-RPC处理函数返回的一样::

          test('JSON-RPC', {rpc: 'mock', asserts: 3, templates: true}, function (instance, $s, mock) {
              var fetched_dbs = false, fetched_langs = false;
              mock('/web/database/get_list', function () {
                  fetched_dbs = true;
                  return ['foo', 'bar', 'baz'];
              });
              mock('/web/session/get_lang_list', function () {
                  fetched_langs = true;
                  return [['vo_IS', 'Hopelandic / Vonlenska']];
              });

              // 控件需要它不然它会爆炸
              instance.webclient = {toggle_bars: odoo.testing.noop};
              var dbm = new instance.web.DatabaseManager({});
              return dbm.appendTo($s).then(function () {
                  ok(fetched_dbs, "should have fetched databases");
                  ok(fetched_langs, "should have fetched languages");
                  deepEqual(dbm.db_list, ['foo', 'bar', 'baz']);
              });
          });

.. note::

    虚拟处理函数可以包含声明, 这些生命应该是生命计数的一部分 (并且如果多个调用被用于一个包含声明的处理函数,
    它会乘上有效的声明的数目).

测试(Testing API)
-----------

.. function:: odoo.testing.section(name[, options], body)

    一个测试区域, 作为一个相关测试的共享的命名空间(对于只设置一侧的常量和值).
    他们 ``本体(body)`` 函数应该包含测试其本身.

    注意测试运行的顺序本质上是未定义的, *不要* 依赖它.

    :param String name:
    :param TestOptions options:
    :param body:
    :type body: Function<:func:`~odoo.testing.case`, void>

.. function:: odoo.testing.case(name[, options], callback)

    寄存一个在测试运行程序中的测试情形的回调, 这个回调只会在运行函数启动时启动一次 (或者根本不运行,
    如果测试被过滤了的话).

    :param String name:
    :param TestOptions options:
    :param callback:
    :type callback: Function<instance, $, Function<String, Function, void>>

.. class:: TestOptions

    可以被传递给 :func:`~odoo.testing.section` 或者 :func:`~odoo.testing.case` 的各种选项.
    除了 :attr:`~TestOptions.setup` 和 :attr:`~TestOptions.teardown`, 一个
    :func:`~odoo.testing.case` 的选项会复写相应的 :func:`~odoo.testing.section` 选项,所以
    比如 :attr:`~TestOptions.rpc` 可以被设为一个 :func:`~odoo.testing.section`
    并且接下来区别的设置 :func:`~odoo.testing.section` 的 :func:`~odoo.testing.case`


    .. attribute:: TestOptions.asserts

        一个整数, 应该在一个测试的正常处理中运行的声明的数量. 对于异步测试是必须的.

    .. attribute:: TestOptions.setup

        测试情形测试, 在每个测试情形之前运行. 如果两者都指定了的话,一个区域的 :func:`~TestOptions.setup`
        会在测试情形本身之前被运行.

    .. attribute:: TestOptions.teardown

        测试情形拆解, 如果二者都有的话,一个情形的 :func:`~TestOptions.teardown` 会在相应的区域之前被运行.

    .. attribute:: TestOptions.fail_on_rejection

        如果测试是异步的并且它导致的约定被拒绝了, 就使测试失效. 默认为 ``true``,
        在拒绝情况下不要使这个测试失败,就把它设置为 ``false`` ::

            // test/demo.js
            test('unfail rejection', {
                asserts: 1,
                fail_on_rejection: false
            }, function () {
                var d = $.Deferred();
                setTimeout(function () {
                    ok(true);
                    d.reject();
                }, 100);
                return d;
            });

    .. attribute:: TestOptions.rpc

        测试时使用的RPC方法,  ``"mock"`` 或 ``"rpc"`` 之一. 任何其它的值会对测试禁用RPC
        (如果他们被实例的成员启用了的话).

    .. attribute:: TestOptions.templates

        当前的模块 (和它的依赖)的模板是否应该在测试开始是载入到QWeb中. 一个布尔值, 默认为 ``false``.

测试运行程序也可以使用两个直接在 ``window`` 对象上的全局的设置值:

* ``oe_all_dependencies`` 是一个有web组件的所有的模块的 ``阵列(Array)`` ,
  由依赖性来排序 (在阵列中,一个 ``A`` ,其依赖的模块是 ``A'``, 那么 ``A'`` 的任何模块必须在 ``A`` 之前)

使用Python来运行(Running through Python)
----------------------

网络客户端包含同等重要的一下东西:在命令行中运行这些测试 (或者在一个CI系统中), 同时真正的运行他们也很简单,
但是设置之前的准备环境还是有点点复杂的.

#. 安装 PhantomJS_. 它是一个简易的浏览器,其允许自动运行和测试网络页面.
   QUnitSuite_ 使用它真正的运行 qunit_ 测试套件.

   PhantomJS_ 网站在某些平台上提供一个预先编译好的二进制文件,
   并且你的操作系统的包管理器很可能也会提供它.

   如果你从源码编译 PhantomJS_ , 我简易准备一些休息时间因为这不是很快
   (它需要同时编译 `Qt <http://qt-project.org/>`_ 和 `Webkit
   <http://www.webkit.org/>`_, 两个都是很大的项目).

   .. note::

       因为 PhantomJS_ 是基于webkit的, 如果Firefox, Opera or Internet Explorer
       可以正确的运行这个套件,也不能够进行测试(并且其对于 Safari 和 Chrome 也只是一个近似).
       因此推荐 *要* 在真实的浏览器中运行这个套件一次.

   .. note::

       这里使用的 PhantomJS_ 版本是编译的1.7, 之前的版本 *应该* 可以工作但是不是真正的被支持
       (并且在事情在 PhantomJS_ 本身不顺利的时候,倾向于是一段代码错误,所以调试起来会很痛苦).

#. 使用所有相关的模块安装一个新的数据库(有至少一个网络组件的所有的模块), 然后启动服务器

   .. note::

       对于一些测试, 一个源数据库需要被复制. 这个操作需要被复制的数据库没有任何连接,
       但是目前Odoo不会断开已经存在/未完成的连接, 所以重启服务器是确保所有事情都在正确状态里的最简单的方式.

#. 使用正确的指定的addons路径启动 ``oe run-tests -d $DATABASE -mweb``
   (并且使用你在上一步中创建的源数据库替换 ``$DATABASE`` )

   .. note::

       If you leave out ``-mweb``, the runner will attempt to run all
       the tests in all the modules, which may or may not work.

如果一切顺利的话, 你应该可以看到一列有 ``ok`` (很可能是这样)挨着他们名字的的测试,
以运行的测试数量和花费的时间的报告作为结尾:

.. literalinclude:: test-report.txt
    :language: text

恭喜你, 你刚刚执行完一个成功的 "离线(offline)" Odoo网络测试套件的运行.

.. note::

    请注意这个是对 ``web`` 模块运行所有的Python测试, 而不是Odoo的所有的web测试. 这可能会让人意外.

.. _qunit: http://qunitjs.com/

.. _qunit assertions: http://api.qunitjs.com/category/assert/

.. _QUnitSuite: http://pypi.python.org/pypi/QUnitSuite/

.. _PhantomJS: http://phantomjs.org/

.. [#eventsdelegation] 并不是所有的 DOM 事件都与事件委托兼容

.. [#terminal]
    可能有点别扭: :py:meth:`sqlalchemy.orm.query.Query.group_by` 不是终点,
    它返回一个仍然可以被替换的查询.

