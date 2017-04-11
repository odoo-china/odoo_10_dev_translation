:banner: banners/actions.jpg

.. _reference/actions:

=======
动作
=======

动作定义 系统响应用户动作的行为: 登录,
动作按钮, 选择发票, ......

动作可以存储在数据库中或直接作为字典返回
例如 按钮方法. 所有动作共享的两个必要属性:

``type``
    当前动作的类别，确定可以使用哪些字段以及如何解释动作

``name``
    简要用户可读描述的动作, 可能会显示在客户端的界面中客户端可以采用以下4种形式获取动作:

``False``
    如果当前有打开任何的动作对话框，请将其关闭
字符串
    如果 :ref:`客户端动作 <reference/actions/client>` 匹配，解释为客户端动作的标签，否则视为数字
数字
    从数据库中读取相应的动作记录,可以是数据库标识符或者一个 :term:`external id`
字典
    视为客户端动作描述符并执行

.. _reference/actions/window:

窗口动作 (``ir.actions.act_window``)
==========================================

最常见的动作类型，用于通过视图呈现模型的可视化
:ref:`视图 <reference/views>`: 窗口动作定义模型
(以及可能的模型的特定记录) 的一组视图类型 (以及可能的特定视图)。

字段有:

``res_model``
    模型来呈现视图
``views``
    成对的列表 ``(view_id, view_type)`` 。每对的第二个元素是视图的类别(tree, form, graph, ...) 第一个是可选数据库id (或 ``False``)。如果没有提供id, 客户端应该获取所请求模型的指定类型的默认视图(这是通过 :meth:`~odoo.models.Model.fields_view_get` 自动完成的）。列表的第一种类型是默认视图类型，并且将在执行动作时默认打开。每个视图类型在列表中最多只能出现一次
``res_id`` (选填)
    如果默认视图是 ``form``， 则指定要加载的记录 (否则应创建一个新纪录)
``search_view_id`` (选填)
    成对的 ``(id, name)`` ，``id`` 是要为动作加载特定的搜索视图的数据库标识符。默认为获取模型的默认搜索视图
``target`` (选填)
    是否应该在主内容区域打开视图 (``current``)，
    全局模式 (``fullscreen``) 或在一个对话框/弹出窗口 (``new``)中打开视图。 使用
    ``main`` 而不是 ``current`` 清除面包屑。默认值为
    ``current``。
``context`` (选填)
    附加上下文数据传递给视图
``domain`` (选填)
    域名过滤以隐式添加到所有的视图搜索查询中
``limit`` (选填)
    默认情况下，列表显示的记录数，在web客户端中默认为80
``auto_search`` (选填)
   是否应在加载默认视图后立即执行搜索视图。默认为 ``True``

例如，打开客户使用的 (与 ``customer`` 标志设置的合作伙伴) 列表和表单视图::

    {
        "type": "ir.actions.act_window",
        "res_model": "res.partner",
        "views": [[False, "tree"], [False, "form"]],
        "domain": [["customer", "=", true]],
    }

或者在新的对话框中打开特定产品（单独获得）的表单视图::

    {
        "type": "ir.actions.act_window",
        "res_model": "product.product",
        "views": [[False, "form"]],
        "res_id": a_product_id,
        "target": "new",
    }

数据库中窗口操作有一些不同的字段，客户端应该忽略，主要用于组合 ``views`` 列表：

``view_mode``
    逗号分隔的视图类型列表作为字符串。 所有这些类型将存在于生成的 ``views`` 列表中 (至少具有 ``False`` view_id)
``view_ids``
    M2M1查看对象, 定义 ``views`` 的初始内容
``view_id``
    特定视图添加到 ``views`` 列表中，如果它的类型是 ``view_mode`` 列表的一部分，
    并且尚未由 ``view_ids`` 中的某个视图填充

这些主要用于从 :ref:`reference/data` 定义操作时:

.. code-block:: xml

    <record model="ir.actions.act_window" id="test_action">
        <field name="name">A Test Action</field>
        <field name="res_model">some.model</field>
        <field name="view_mode">graph</field>
        <field name="view_id" ref="my_specific_view"/>
    </record>

将使用 "my_specific_view" 视图， 即使这不是模型的默认视图。

``views`` 序列的服务器端组成如下：

* ``view_ids`` 获取每个 ``(id, type)`` (按 ``sequence`` 排序)
* 如果 ``view_id`` 已定义并且其类型尚未填充，则加上 ``(id, type)``
* 对于 ``view_mode`` 中的每个未填充类型, 加上 ``(False, type)``

.. todo::

    * ``src_model``, ``multi`` seem linked to "sidebar" actions?
    * ``auto_refresh`` looks ignored/deprecated
    * ``usage``?
    * ``groups_id``?
    * ``filter``?

.. _reference/actions/url:

URL 动作 (``ir.actions.act_url``)
====================================

允许通过Odoo动作打开URL（网站/网页）。可以通过两个字段进行定制:

``url``
   激活动作时打开的地址
``target``
    在新窗口/页面中打开地址，如果是 ``new``， 则用页面替换当前内容 ``self``。 默认为 ``new``

::

    {
        "type": "ir.actions.act_url",
        "url": "http://odoo.com",
        "target": "self",
    }

将用Odoo主页替换当前内容部分。

.. _reference/actions/server:

服务器动作 (``ir.actions.server``)
======================================

允许从任何有效的动作位置触发复杂服务器代码。只有两个字段与客户端有关:

``id``
    要运行的服务器动作数据库中的标识符
``context`` (选填)
    运行服务器动作时要使用的上下文数据

数据库中的记录明显更丰富，并且可以根据其状态执行大量特定的或通用的动作 ``state`` . 一些字段（和相应的行为）在状态中共享：

``model_id``
    Odoo模型链接到动作，通过 :ref:`contexts 赋值<reference/actions/server/context>`
``condition`` (选填)
    使用服务器动作的 :ref:`context 赋值<reference/actions/server/context>` 为Python代码。 如果为 ``False`` ， 则阻止动作运行。默认为: ``True``

有效的动作类型 (``state`` 字段) 是可扩展的, 默认类型是:

``code``
--------

默认和最具灵活性的服务器动作类型，使用动作的 :ref:`context 赋值
<reference/actions/server/context>` 检测任意的Python代码。 仅使用一个特定类型的特定字段：

``code``
    一段Python代码在调用动作时执行

.. code-block:: xml

    <record model="ir.actions.server" id="print_instance">
        <field name="name">Res Partner Server Action</field>
        <field name="model_id" ref="model_res_partner"/>
        <field name="code">
            raise Warning(object.name)
        </field>
    </record>

.. note::

    代码段中可以定义一个名为“action”的变量，它将作为下一个要执行的动作返回给客户端：

    .. code-block:: xml

        <record model="ir.actions.server" id="print_instance">
            <field name="name">Res Partner Server Action</field>
            <field name="model_id" ref="model_res_partner"/>
            <field name="code">
                if object.some_condition():
                    action = {
                        "type": "ir.actions.act_window",
                        "view_mode": "form",
                        "res_model": object._name,
                        "res_id": object.id,
                    }
            </field>
        </record>

    将要求客户打开一个表单记录，如果它满足一些条件

这往往是在 :ref:`数据文件 <reference/data>` 创建的唯一操作类型, 除了 :ref:`reference/actions/server/multi` 外，其它类型比Python代码定义的更简单，从UI定义，而不是从 :ref:`数据文件 <reference/data>` 中定义。

.. _reference/actions/server/object_create:

``object_create``
-----------------

创建一个新纪录，从头开始 (通过 :meth:`~odoo.models.Model.create`)
或通过复制现有记录 (通过 :meth:`~odoo.models.Model.copy`)

``use_create``
    创建的方法，其中之一：

    ``new``
        在由 ``model_id`` 指定的模型中创建一个记录
    ``new_other``
        在由 ``crud_model_id`` 指定的模型中创建一个记录
    ``copy_current``
        复制调用操作的记录
    ``copy_other``
        复制其他记录，通过 ``ref_object`` 获取

``fields_lines``
    要在创建或者复制记录时覆盖的字段。:class:`~odoo.fields.One2many` 与字段:

    ``col1``
        ``ir.model.fields`` 在 ``use_create`` 隐含的模型中设置
    ``value``
        字段的值，通过 ``type`` 解释
    ``type``
        如果是 ``value`` , ``value`` 字段被解释为一个字面值（可能被转换）, 如果 ``equation`` 的 ``value`` 字段是解释为Python表达式并被进行求值
``crud_model_id``
    模型中创建一个新纪录, 如果 ``use_create`` 被设置为 ``new_other``
``ref_object``
    :class:`~odoo.fields.Reference` 要复制的任意记录, 如果 ``use_create``
    设置为 ``copy_other`` ，则使用
``link_new_record``
    booleac标志通过 ``link_field_id`` 指定的many2one字段将新创建的记录链接到当前记录, 默认为 ``False``
``link_field_id``
    many2one 到 ``ir.model.fields`` ，指定当前记录的m2o字段，应在其上设置新创建的记录（模型应该匹配）

``object_write``
----------------

与 :ref:`reference/actions/server/object_create` 类似，但会更改现有记录，而不是创建新纪录

``use_write``
    写的方法，以下之一:

    ``current``
        写入当前记录
    ``other``
        写入通过 ``crud_model_id`` 和 ``ref_object`` 选择的其他记录
    ``expression``
        写入通过 ``crud_model_id`` 选择其模型的其他记录，并通过创建 ``write_expression`` 选择其id
``write_expression``
    返回记录或对象标识的Python表达式，在 ``use_write`` 设置为 ``expression`` 使用， 以便决定应该修改哪个记录
``fields_lines``
    查阅 :ref:`reference/actions/server/object_create`
``crud_model_id``
    查阅 :ref:`reference/actions/server/object_create`
``ref_object``
    查阅 :ref:`reference/actions/server/object_create`

.. _reference/actions/server/multi:

``multi``
---------

一个接一个执行多个动作。要执行的动作通过 ``child_ids`` m2m定义. 如果子动作本身返回动作，最后一个将作为自己的下一个动作返回到客户端

``trigger``
-----------

向工作流发送信号

``wkf_transition_id``
    触发 :class:`~odoo.fields.Many2one` 到 ``workflow.transition``
``use_relational_model``
    if ``base`` (默认)， 则代表当前记录触发信号。 如果是 ``relational``，则代表通过 ``wkf_model_id`` 和 ``wkf_field_id`` 选择当前记录的字段触发信号

``client_action``
-----------------

用于直接返回使用 ``action_id`` 定义的其他的间接动作。将该动作返回给客户端执行

.. _reference/actions/server/context:

求值上下文
------------------

在服务器动作求值上下文中或周围有多个密钥可用：

``self``
    通过 ``model_id`` 链接到动作的模型对象
``object``, ``obj``
    仅在提供 ``active_model`` 和 ``active_id`` （通过上下文）时可用，否则为 ``None``。
    由 ``active_id`` 选择实际的记录
``pool``
    当前数据库注册表
``datetime``, ``dateutil``, ``time``
    对应的Python模块
``cr``
    当前位置
``user``
    当前用户记录
``context``
    执行上下文
``Warning``
     ``Warning`` 异常的构造函数

.. 忽略UID (通过可用的 ``user``), 工作流（可通过对模型的工作流程方法）

.. _reference/actions/report:

报表动作 (``ir.actions.report.xml``)
==========================================

触发报表的打印

``name`` (必填)
    用在列表视图中作为描述显示或者排序
``model`` (必填)
    您的报表所关注的模型
``report_type`` (必填)
    ``qweb-pdf`` 用于PDF报表或 ``qweb-html`` 用于HTML
``report_name``
    您的报表的名称（这将是PDF输出的名称）
``groups_id``
    :class:`~odoo.fields.Many2many` 字段允许查看/使用当前报表的分组
``paperformat_id``
    :class:`~odoo.fields.Many2one` 字段设置适合此报表的纸张格式（如果未指定，将使用默认格式）
``attachment_use``
    如果设置为 ``True`` ，报表只在第一次请求时生成，并且随后则为从存储的报表中直接打印，而不是每次都重新生成。

    可用于只能生成一次的报表 (例如出于法律的原因)

``attachment``
   定义报表名称的Python表达式；该记录可作为变量访问 ``object``

.. _reference/actions/client:

客户端动作 (``ir.actions.client``)
======================================

触发完全在客户端中实现的动作。

``tag``
    客户端动作的标识符，客户端应知道如何响应任意的字符串
``params`` (选填)
    发送给客户端附加数据的Python字典，以及客户端动作标签
``target`` (选填)
    客户端动作应在内容区域(``current``)， 或在全局模式下 (``fullscreen``) 对话框/弹出窗口
    (``new``)中打开。使用 ``main`` 而不是 ``current`` 清除面包屑。默认为 ``current``.

::

    {
        "type": "ir.actions.client",
        "tag": "pos.ui"
    }

告诉客户端启动Point of Sale接口，服务器是不知道POS接口是如何工作的。

.. [#notquitem2m] 技术上不是M2M：添加序列字段，并且可以仅由视图类型组成，而没有视图id。
