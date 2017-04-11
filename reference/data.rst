:banner: banners/data_files.jpg

.. _reference/data:

==========
数据文件
==========

Odoo在很大程度上是数据驱动的, 并且很大一部分的模块定义就是它管理的不同的记录(record)的定义
: 用户界面 (菜单和视图), 安全 (访问权限和访问规则), 报表和基础数据的定义都是通过记录(record).

结构
=========

在Odoo中定义数据的主要方式是通过 XML 数据文件: 一个XML数据文件的大致结构如下:

* 在根元素 ``odoo`` 下有任意多的操作元素

.. code-block:: xml

    <!-- 数据文件的根元素 -->
    <odoo>
      <operation/>
      ...
    </odoo>

数据文件是被按顺序进行处理的, 进行的操作只能够指向到之前已经定义好的操作

核心操作
===============

.. _reference/data/record:

``记录(record)``
----------

``记录(record)`` 适当的定义或者更新一个数据库的记录, 它有如下的属性:

``模型(model)`` (必须的)
    增加(或更新)的模型(model)的名字
``id``
    记录(record)的 :term:`外部标识` . 强烈建议提供一个这样的标识

    * 在增加记录(record)时, 允许后续的定义来修改或者参考这个记录(record)
    * 在修改记录(record)是, 指向要修改的记录(record)
``内容(context)``
    在增加记录(record)时使用的 context
``强制更新(forcecreate)``
    在更新模式中, 如果记录(record)不存在的话是否创建它

    需要一个 :term:`外部id`, 默认为 ``True``.

``字段(field)``
'''''''''

每个记录(record)都可以由 ``字段(field)`` 标签组成, 定义在创建记录(record)时设置的值.
一个 ``记录(record)`` 如果没有 ``字段(field)`` 的话会使用所有默认值(创建时)
或者设么也不做(更新时).

一个 ``字段(field)`` 有一个必须的 ``name`` 属性, 设置字段(field)的名字,
和各种方法来定义它自己:

空
    如果没有给这个字段(field)提供值, 一个隐藏的 ``False`` 会设置进去.
    可以用于清空一个字段(field), 或者避免使用这个字段(field)的默认值.
``查找(search)``
    用于 :ref:`关系字段(field) <reference/orm/fields/relational>`,
    在这个字段(field)的model上需要一个 :ref:`domain <reference/orm/domains>` .

    将会评估这个 domain, 查找使用这个字段(field)的模型(model)并且设置search的结果作为这个字段(field)的值.
    如果返回的结果是一个 :class:`~odoo.fields.Many2one` 则只会使用第一个结果
``引用(ref)``
    如果提供了一个 ``ref`` 属性, 他的值必须是有效值的 :term:`外部id`,
    其会被查找到并且设置为字段(field)的值.

    大部分都是为了 :class:`~odoo.fields.Many2one` 和
    :class:`~odoo.fields.Reference` 字段(field)
``类型(type)``
    如果提供了一个 ``type`` 属性, 他会被用于解释和转换字段(field)的内容.
    这个字段(field)的内容会提供给一个使用 ``file`` 属性的外部文件,
    或者提供给这个节点的内部.

    可用的 type 有:

    ``xml``, ``html``
        提取这个 ``字段(field)`` 的子类作为一个单独的文件, 评估任何指定为这种
        ``%(external_id)s`` 形式的 :term:`外部id` .
        ``%%`` 能够用于输出 *%* 标识.
    ``file``
        确保字段(field)的内容是在当前模型(model)下一个有效的文件路径,
        报春一对 :samp:`{module},{path}` 作为字段(field)的值
    ``char``
        设置这个字段(field)的内容不做修改的直接作为这个字段(field)的值
    ``base64``
        base64_-编码了这个字段(field)的内容, 结合 ``file``
        *属性* 来载入, 例如图片等, 数据到附件中
    ``int``
        转换这个字段(field)的内容为一个整数并设置它作为字段(field)的值
    ``float``
        转换这个字段(field)的内容为一个浮点数并设置它作为字段(field)的值
    ``list``, ``tuple``
        应该包含任意数量有相同性质的 ``值(value)`` 元素作为 ``字段(field)``,
        每个元素分解为一个生成的元组或者列表的元素, 并且生成的集合被设置为字段(field)的值.
``评估(eval)``
    在前面方法不合适的情况下, ``eval`` 属性简单的评估提供给他的python表达式并且设置
    其结果为字段(field)的值.

    这个评估的内容包含很多的的模块 (``time``, ``datetime``,
    ``timedelta``, ``relativedelta``), 一个解析 :term:`外部标识` (``ref``)
    的函数和当前字段(field)的模型(model) 对象如果其是可应用的 (``obj``)

``删除(delete)``
----------

``删除(delete)`` 标签可以移除任何数目之前定义的记录(record). 它有如下的属性:

``模型(model)`` (必须)
    需要删除的记录(record)所位于的模型(model)
``id``
    需要删除的记录(record)的 :term:`外部id`
``查找(search)``
    一个用于查找要删除的记录(record)的 :ref:`domain <reference/orm/domains>`

``id`` 和 ``查找(search)`` 是互斥的

``函数(function)``
------------

``函数(function)`` 标签用提供的参数来调用一个model的方法.
它有两个强制参数 ``模型(model)`` 和 ``name`` 用于指定调用的模型(model)和其名字.

参数可以用 ``eval`` 来提供(应该计算一系列参数来调用方法)
或者 ``值(value)`` 元素 (参照 ``list`` 的值).

``工作流(workflow)``
------------

``工作流(workflow)`` 标签给一个存在的工作流发送一个信号. 这个工作流可以通过一个 ``ref`` 属性来被指定
(一个存在的工作流的 :term:`外部id`) 或者一个 ``值(value)`` 标签来返回一个工作流的id.

这个标签有两个强制的属性 ``模型(model)`` (模型(model) 链接至的工作流) 和 ``action``
(发送至工作流的信号名字).

.. ignored assert

快捷方式(Shortcuts)
=========

因为一些Odoo的重要的结构性模型(模型(model))是复杂且关联的,
数据文件提供更短的替代名称使用 :ref:`记录(record)标签 <reference/data/record>` 来定义他们:

``菜单项(menuitem)``
------------

用默认值和备用值(fallbacks)定义一个 ``ir.ui.menu`` 记录(record):

父(Parent)菜单
    * 如果设置了一个 ``parent`` 属性, 它应该是另一个菜单项的 :term:`外部id`,
    作为这个新的项目的上级(parent)
    * 如果没有提供 ``parent`` , 系统会尝试解析 ``name`` 属性作为一个
    ``/``-不同的菜单名队列并且在这个菜单层级中中找到一个地方.
    在编译中, 中间菜单会被自动创建
    * 否则菜单会被定义为一个 "顶级" 的菜单项 (*并不是* 一个没有父的菜单项)
菜单名字
    如果没有 ``名字`` 属性, 如果有的话，会尝试从一个链接的动作(action)来获得这个菜单名字.
    否则则使用这个记录(record)的 ``id``
组(Groups)
    一个 ``组(groups)`` 属性会被编译成为一个逗号分隔的
    用于 ``res.groups`` 模型的 :term:`外部标识` . 如果一个
    :term:`外部标识` 有一个减号 (``-``) 作为前缀, 这个组会被从这个菜单组中 *移除*
``动作(action)``
    如果指定了, ``action`` 属性应该是在菜单打开时执行的action的 :term:`外部id`
``id``
    菜单项的 :term:`外部id`

.. _reference/data/template:

``模板(template)``
------------

创建一个 :ref:`QWeb 视图 <reference/views/qweb>` 只需要视图的 ``arch`` 部分,
并且语序少量 *可选* 属性:

``id``
    这个视图的 :term:`外部标识`
``名字(name)``, ``inherit_id``, ``优先级(priority)``
    跟 ``ir.ui.view`` 相应的字段意义相同 (注意: ``inherit_id``
    应该是一个 :term:`外部标识`)
``主项(primary)``
    如果设置为 ``True`` 并且结合了一个 ``inherit_id``, 就定义这个视图为主视图
``组(groups)``
    逗号分隔的组的 :term:`外部标识` 列表
``页面(page)``
    如果设置为 ``"True"``, 这个模板是会一个网页 (可链接, 可删除)
``可选(optional)``
    ``enabled`` 或者 ``disabled``, 决定了这个视图是不是可以被禁用 ( 在网站接口)
    和他的默认状态. 如果没设置的话, 视图总是启用的.

``报告(report)``
----------

创建一个有少量默认值的 ``ir.actions.report.xml`` 记录(record).

到部分只是 ``ir.actions.report.xml`` 相应字段的代理属性,
但会在报告(report)的 ``模型(model)`` 的 :guilabel:`更多(More)` 菜单中自动创建项目.

.. ignored url, act_window and ir_set

CSV 数据文件
==============

XML 数据文件是灵活且能自我描述的, 但在创建大量相同模型上的简单记录的时候就会显得冗长.

在这种情况中, 数据文件可以使用 csv_, 这是 :ref:`权限管理 <reference/security/acl>` 的常见情况:

* 文件名是 :file:`{model_name}.csv`
* 第一行列出了需要写的字段, 有指定的字段 ``id`` 作为 :term:`外部标识` (用于创建或更新)
* 以后的每行创建一个新的记录(record)

这是定义了美国州的数据文件的第一行 ``res.country.state.csv``

.. literalinclude:: ../../odoo/addons/base/res/res.country.state.csv
    :language: text
    :lines: 1-15

用一个更易读的格式渲染后的效果:

.. csv-table::
    :file: ../../odoo/addons/base/res/res.country.state.csv
    :header-rows: 1
    :class: table-striped table-hover table-condensed

对于每一行 (记录):

* 第一列是要创建或更新的记录的 :term:`外部id`
* 第二列是要链接道德国家对象的 :term:`外部id` (国家对象必须在之前定义好)
* 第三列是 ``res.country.state`` 的 ``name`` 字段
* 第四列是 ``res.country.state`` 的 ``code`` 字段

.. _base64: http://tools.ietf.org/html/rfc3548.html#section-3
.. _csv: http://en.wikipedia.org/wiki/Comma-separated_values
