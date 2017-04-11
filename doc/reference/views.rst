:banner: banners/views.jpg

.. highlight:: xml
.. _reference/views:

=====
视图
=====

.. _reference/views/structure:

共同结构
================

视图类型有多个字段，除非另有说明，否则它们都是选填的

``name`` (必填)
    只是用来在列表视图里面作为描述显示或者排序
``model``
   模型链接到视图（它不适用于QWeb视图）
``priority``
    客户端可以通过 ``id`` ， 或者 ``(model, type)`` 请求视图。 对于后者， 将会搜索出所有正确类型和模型的视图，并且将返回最小 ``priority`` 编号的视图 (它是“默认视图”)。

    ``priority`` 也定义了视图继承期间的应用顺序 :ref:`视图继承 <reference/views/inheritance>`
``arch``
    视图布局的描述
``groups_id``
    :class:`~odoo.fields.Many2many` 字段允许查看/使用当前视图的分组
``inherit_id``
    当前视图的父视图，详情可查阅 :ref:`reference/views/inheritance` ，默认情况下没有设置
``mode``
    继承模型， 详情可查阅 :ref:`reference/views/inheritance`。如果
    ``inherit_id`` 未设置，则 ``mode`` 只能为 ``primary``。 如果设置了
    ``inherit_id`` ， 默认情况下为 ``extension`` ，但可以设置为 ``primary``
``application``
    网站功能定义可切换的视图。默认情况下，始终应用该视图

.. _reference/views/inheritance:

继承
===========

视图匹配
-------------

* 如果一个视图被 ``(model, type)`` 请求， 则视图会有正确的模型和类型，并且 ``mode=primary`` 的最低优先级是匹配的
* 当一个视图被 ``id`` 请求时， 如果它的模型不是 ``primary`` ，则其最相近的父视图 ``primary`` 是匹配的

视图分解
---------------

分解为匹配的 ``primary`` 生成最终的 ``arch`` 视图:

#. 如果视图具有父视图，则父级完全解析，然后应用当前视图的继承说明
#. 如果视图没有父视图，则其 ``arch`` 按原样使用
#. 将查找当前视图具有模型 ``extension`` 扩展的子项，并且深度优先应用它们的继承说明（应用子视图，应用其子代，应用其兄弟)

应用子视图的结果是产生最终的 ``arch``

继承说明
-----------------

继承由元素定位器组成，以匹配父视图中的继承元素，以及用于修改所继承元素的子元素

有三种类型的元素定位符用于匹配目标元素:

* 具有 ``expr`` 属性的 ``xpath`` 元素。 ``expr`` 是一个应用于当前 ``arch`` 的 XPath_  表达式\ [#hasclass]_ ，第一个节点是匹配的
* 具有 ``name`` 属性的 ``field`` 元素， 匹配具有相同 ``name`` 的第一个 ``field`` 。在匹配期间将忽略其它所有的属性
* 任何其它元素：具有相同名称和第一个元素相同的属性(忽略 ``position`` 和 ``version`` 属性)是匹配的，继承可以有可选的 ``position`` 属性，指定如何更改匹配的节点:

``inside`` (默认)
    继承的内容将添加到所匹配的节点
``replace``
    继承的内容将替换所匹配的节点。在规范的内容中包含 `$0` 的任何文本节点将被所匹配节点的副本替换，  	从而有效地包装匹配的节点
``after``
    继承的内容将添加到所匹配节点的父节点之后
``before``
    继承的内容将添加到匹配节点的父节点之前
``attributes``
    继承的内容应该具有 ``name`` 属性和选填的 ``attribute`` 元素：

    * 如果 ``attribute`` 元素是主要的， 则在所匹配的节点上创建以其 ``name`` 命名的新属性，将
      ``attribute`` 元素的文本作为值
    * 如果 ``attribute`` 元素不是主要的, 则需从所匹配的节点中删除以其 ``name`` 命名的属性。如果不存在这样的属性将产生错误
视图的说明是按顺序应用的。

.. _reference/views/list:

列表
=====

列表视图的根元素是 ``<tree>``\ [#treehistory]_ 。列表视图的根元素可以有以下属性

``editable``
    默认情况下, 选择列表视图的行打开对应的 :ref:`form view <reference/views/form>` 。 ``editable`` 属性可以让列表视图在原文进行编辑。

    验证值是 ``top`` 和 ``bottom`` ，使创建的新纪录分别出现在列表的顶部或底部.

    内联 :ref:`form view <reference/views/form>` 的体系结构派生自列表视图。在 :ref:`form view <reference/views/form>` 的字段和按钮上大多数有效的属性因此被列表视图所接受，如果列表视图是不可编辑的，它们可能没有任何意义

``default_order``
    覆盖视图的顺序，替换模型的默认顺序。该值是以逗号最为分割，列表后缀以 ``desc`` 反向排序

    .. code-block:: xml

        <tree default_order="sequence,name desc">
``colors``
    .. deprecated:: 9.0被 ``decoration-{$name}`` 替换
``fonts``
    .. deprecated:: 9.0被 ``decoration-{$name}`` 替换
``decoration-{$name}``
    允许基于记录的属性更改行文本的样式

    其值是Python的表达式。对于每个记录，计算表达式将所记录的属性作为上下文值，如果为 ``true``, 相应的样式将应用于该行。其他上下文值为 ``uid`` (当前用户的id) 和 ``current_date`` (当前日期为 ``yyyy-MM-dd`` 形式的字符串)。

    ``{$name}`` 可以是 ``bf`` (``font-weight: bold``)， ``it``
    (``font-style: italic``), 或任何 `引导上下文颜色
    <http://getbootstrap.com/components/#available-variations>`_ (``danger``,
    ``info``, ``muted``, ``primary``, ``success`` 或 ``warning``)。
``create``, ``edit``, ``delete``
    通过将相应的属性设置为 ``false`` 来 *禁用* 视图中的相应操作
``on_write``
    只有在 ``editable`` 列表中才有意义。列表模型上方法的名称。在创建或编辑该记录（在数据库中）后，将使用记录的 ``id`` 来调用该方法。

    该方法应返回其他记录的id列表以方便加载或更新
``string``
    视图的替代可翻译标签

    .. deprecated:: 8.0

        不再显示

.. toolbar attribute is for tree-tree views

列表视图的可能子元素是:

.. _reference/views/list/button:

``button``
    在列表单元格中显示按钮

    ``icon``
        图标，用于显示按钮
    ``string``
        * 如果没有 ``icon`` ， 那么就是按钮的文本
        * 如果有 ``icon`` ，则为 ``alt`` 的替代文本
    ``type``
        按钮的类型，提示如何点击它影响Odoo：

        ``workflow`` (默认)
            向工作流发送信号。按钮的 ``name`` 是工作流信号，行的记录作为参数传递给信号

        ``object``
            调用列表模型上的方法。按钮的 ``name`` 是方法，它使用当前记录的id以及在当前上下文中调用。

            .. web client also supports a @args, which allows providing
               additional arguments as JSON. Should that be documented? Does
               not seem to be used anywhere

        ``action``
            加载执行一个 ``ir.actions`` ，按钮的 ``name`` 动作的数据库id。上下文中使用的列表模型(作为 ``active_model``)扩展，当前行的记录(``active_id``)和列表中当前加载的所有记录			(``active_ids``，可能只是一个子集数据库记录匹配当前的搜索)
    ``name``
        详情请查阅 ``type``
    ``args``
        详情请查阅 ``type``
    ``attrs``
        基于记录值的动态属性

        属性到域, 域的映射在当前记录的上下文中说明，如果为 ``True`` ，则在单元上设置相应的属性

        可能的属性是 ``invisible`` (隐藏按钮)和 ``readonly`` (禁用按钮，但扔显示)
    ``states``
       ``invisible`` ``attrs`` 的简写: 状态列表，逗号分隔，要求模型具有 ``state`` 字段，并且它在视图中

       如果记录 *不是* 在所列出的状态当中，使按钮 ``invisible``
    ``context``
       在执行按钮的Odoo调用时合并到视图的上下文中
    ``confirm``
       确认消息以在执行按钮的Odoo调用之前显示 (并且供用户接受)

    .. declared but unused: help

``field``
    定义一个列，其中应为每个记录显示相应的字段。可以使用以下属性：

    ``name``
        要在当前模型中显示的字段名称。给定的名称对每个视图只能使用一次
    ``string``
        字段列的标题 (默认情况下, 使用模型字段的 ``string``)
    ``invisible``
        提取和存储字段，但不显示列中。对于不应该显示但是需使用的字段是非常有必要的。例如		``@colors``
    ``groups``
        列出应该能够看到字段的分组
    ``widget``
        字段显示的表示，可能的列表视图值包括:

        ``progressbar``
            将 ``float`` 字段显示为进度条。
        ``many2onebutton``
            如果字段是填充的，则通过复选标记替换m2o字段的值
        ``handle``
            对于 ``sequence`` 字段，而不是所显示字段的值只显示一个dra&drop图标
    ``sum``, ``avg``
        在列的底部显示相应的总数。总数仅在 *当前显示的* 记录上计算。总数操作必须与相应字段
        的 ``group_operator`` 匹配
    ``attrs``
        基于记录集的动态属性。只影响当前字段，``invisible`` 将会隐藏字段，但使其他记录的相同字段可见，它不会隐藏列本身

    .. note:: 如果列表视图是 ``editable`` ，在 :ref:`form view <reference/views/form>` 中的任意字段的属性都是有效的，并且将会设置在内关联的表单视图中
.. _reference/views/form:

表单
=====

表单视图用于显示单条记录的数据。根元素是 ``<form>``，由 HTML_ 及其它结构和语义组成

结构组件
---------------------

结构组件提供少逻辑的结构或"视觉"特征。它们用作窗体视图中的元素或元素集。

``notebook``
  定义标题部分。每个选项卡通过一个 ``page`` 子元素定义。页面可以具有以下属性：

  ``string`` (必填)
    标签的标题
  ``accesskey``
    HTML accesskey_
  ``attrs``
    基于记录值的标准动态属性

``group``
  用在表单中定义列。默认情况下，组定义两列并且组的最直接子项采用单列。 ``field`` 组的直接子元素默认显示一个标题，标题和字段本身的列为1。

  ``group`` 中的列数可以用 ``col`` 属性来计算，元素所用的列数可以用 ``colspan`` 属性来计算。

  子元素的横向布局（试着在改变行之前填充下一列）

  分组可以有一个 ``string`` 属性，它显示为分组的标题

``newline``
  只用于 ``group`` 元素，提前结束当前行并立即切换到新行 (没有提前填充任何剩下的列)
``separator``
  小的横向间距，带有 ``string`` 属性的为一个节标题
``sheet``
  可以作为一个直接的子 ``form``，显示为一个美观的表单形式
``header``
  与 ``sheet`` 结合，在工作表上方预留着宽的位置，通常用于显示工作流的按钮和状态的小部件

语义组件
-------------------

语义组件涉及并允许与Odoo系统的交互。可用的语义组件有:

``button``
  调用进去Odoo系统，类似于 :ref:`list view buttons <reference/views/list/button>`
``field``
  渲染 (并允许编辑)当前记录的单个字段。可能的属性包括

  ``name`` (必填)
    要呈现的字段的名称
  ``widget``
    字段具有基于其类型的默认呈现(
    例如 :class:`~odoo.fields.Char` ，:class:`~odoo.fields.Many2one`)。 ``widget`` 属性允许使用不同的渲染方法和上下文

    .. todo:: list of widgets

       & options & specific attributes (e.g. widget=statusbar
       statusbar_visible clickable)
  ``options``
    JSON 对象指定字段窗口小部件的配置(包括默认的窗口小部件)
  ``class``
    HTML类在生成的元素上设置，公共字段有：

    ``oe_inline``
      避免换行符后面的字段
    ``oe_left``, ``oe_right``
      floats_ 字段到相应的地方
    ``oe_read_only``, ``oe_edit_only``
      只显示相应表单模式中的字段
    ``oe_no_button``
      避免显示导航按钮在 :class:`~odoo.fields.Many2one`
    ``oe_avatar``
      对于图像字段，将图像显示为"头像" (正方形,最大尺寸为90x90，某些图像装饰)
  ``groups``
    仅显示特定用户的字段
  ``on_change``
    在编辑此字段的值时调用指定的方法，可以为用户生成更新的其他字段或显示警告

    .. deprecated:: 8.0

       在模型上使用 :func:`odoo.api.onchange`

  ``attrs``
    基于记录值的动态元参数
  ``domain``
    仅用于关系字段；显示现有记录以供选择应用的过滤器
  ``context``
    仅用于关系字段，上下文在获取可能的值时传递
  ``readonly``
    在只读和编辑模式下显示字段，但永远不可编辑
  ``required``
    如果字段没有值，则会生成错误并阻止保存记录
  ``nolabel``
    不自动显示字段的标签，只有当字段是 ``group`` 元素的直接子元素时才有意义
  ``placeholder``
    帮助消息显示在 *空* 字段中。可以替换复杂表单中的的字段标签。*不应该* 是数据，因为用户可能会将占	位符文本与填充字段混淆
  ``mode``
     :class:`~odoo.fields.One2many`，显示模式（视图类型）用于字段的链接记录。 一个 ``tree`` ， 	``form`` ， ``kanban`` 或 ``graph`` 。默认是 ``tree`` (一个列表显示)
  ``help``
    悬停在字段或标签时为用户显示说明提示
  ``filename``
    对于二进制字段，提供文件相关字段的名称
  ``password``
    表示 :class:`~odoo.fields.Char` 字段存储密码，并且不应显示其数据

.. todo:: classes for forms

.. todo:: widgets?

业务视图指南
-------------------------

.. sectionauthor:: Aline Preillon, Raphael Collet

业务视图针对的是普通用户，而不是高级用户。例如包括：机会，产品，合作伙伴，任务，项目等等

.. image:: forms/oppreadonly.png
   :class: img-responsive

一般来说，业务视图由以下几点组成

1. 顶部的状态栏（具有技术或业务流程）
2. 在中间的纸（形式本身），
3. 底部有历史和注释。

从技术上讲，新的表单视图在XML中的结构如下::

    <form>
        <header> ... content of the status bar  ... </header>
        <sheet>  ... content of the sheet       ... </sheet>
        <div class="oe_chatter"> ... content of the bottom part ... </div>
    </form>

状态栏
''''''''''''''

状态栏的目的是显示当前记录和动作按钮的状态。

.. image:: forms/status.png
   :class: img-responsive

按钮
...........

按钮的顺序遵循业务流程。例如，在销售订单中，逻辑步骤是：

1. 发送报价单
2. 确认报价
3. 创建最终发票
4. 发送货物

突出显示的按钮（默认为红色）强调逻辑下一步，以帮助用户。它通常是第一个活动按钮。另一方面，:guilabel:`cancel` 按钮 *必须是* 保持灰色 (正常)。例如，在发票中，按钮 :guilabel:`Refund` 绝不能是红色的。

技术上，按钮通过添加类 "oe_highlight" 突出显示::

    <button class="oe_highlight" name="..." type="..." states="..."/>

状态
..........

使用 ``statusbar`` 窗口小部件，并且以红色显示当前状态。所有流程共有的国家(例如，销售订单以报价开头，然后我们发送，然后成为完整的销售订单，最后完成)应该始终可见，但是根据特定子流程的异常或状态应该仅在当前可见。

.. image:: forms/status1.png
   :class: img-responsive

.. image:: forms/status2.png
   :class: img-responsive

状态按照字段中使用的顺序显示 (选择字段中的列表等)。始终可见的状态使用属性 ``statusbar_visible`` 指定。

::

    <field name="state" widget="statusbar"
        statusbar_visible="draft,sent,progress,invoiced,done" />

工作表
'''''''''

所有业务视图应类似于打印纸纸张的大小:

.. image:: forms/sheet.png
   :class: img-responsive

1.  ``<form>`` 或 ``<page>`` 中的元素不定义组，元素在其内部按照正常的HTML规则布局。它们的内容可以	使用 ``<group>`` 或者 ``<div>`` 元素显示分组。.
2. 默认情况下，元素 ``<group>`` 在里面定义两列，除非属性 ``col="n"`` 被使用。列具有相同的宽度(1/n th 组的宽度)。使用 ``<group>`` 元素来产生一列字段。
3. 为了给一部分赋予标题，向 ``<group>`` 元素中添加 ``string`` 属性::

     <group string="Time-sensitive operations">

   这取代了以前使用 ``<separator string="XXX"/>`` 。
4. ``<field>`` 元素不会产生标签，除了 ``<group>`` element\ [#backwards-compatibility]_ 的直	接子元素。使用 :samp:`<label for="{field_name}>` 以产生字段的标签。

工作表表头
.............
某些工作表具有包含一个或多个字段的标题，并且这些字段的标签仅在编辑模式下显示

.. list-table::
   :header-rows: 1

   * - Edit mode
     - View mode
   * - .. image:: forms/header.png
          :class: img-responsive
     - .. image:: forms/header2.png
          :class: img-responsive

使用HTML文本， ``<div>``, ``<h1>``, ``<h2>``… 生成较好的标题，``<label>`` 使用
类 ``oe_edit_only`` 仅在编辑模式下显示字段的标签。类 ``oe_inline`` 将使字段内联(而不是分段)：字段后面的内容将显示在同一行而不是下一行中。上面的表单由以下XML生成::

    <label for="name" class="oe_edit_only"/>
    <h1><field name="name"/></h1>

    <label for="planned_revenue" class="oe_edit_only"/>
    <h2>
        <field name="planned_revenue" class="oe_inline"/>
        <field name="company_currency" class="oe_inline oe_edit_only"/> at
        <field name="probability" class="oe_inline"/> % success rate
    </h2>

按钮箱
..........

许多相关动作或链接可以在表单中显示。例如，在表单中，动作“安排呼叫”和“安排会议”在使用CRM时具有重要的地位。而不是将它们放在“更多”菜单中，将它们直接放在工作表中作为按钮（在顶部），使它们更加明显，更容易访问。

.. image:: forms/header3.png
   :class: img-responsive

从技术上来说，这些按钮放在一个 ``<div>`` 内，以便将它们分组为表格顶部的块

::

    <div class="oe_button_box" name="button_box">
        <button string="Schedule/Log Call" name="..." type="action"/>
        <button string="Schedule Meeting" name="action_makeMeeting" type="object"/>
    </div>

组和标题
.................

现在使用一个 ``<group>`` 元素生成一列字段，并带有一个可选的标题。

.. image:: forms/screenshot-03.png
   :class: img-responsive

::

    <group string="Payment Options">
        <field name="writeoff_amount"/>
        <field name="payment_option"/>
    </group>

建议表单上有两列字段。为此，只需将包含字段的 ``<group>`` 元素放在顶层的 ``<group>`` 元素中即可

要使 :ref:`view extension <reference/views/inheritance>` 更简单, 建议在 ``<group>`` 元素上放置一个 ``name`` 属性，可以很容易在正确的地方添加 。

特殊情况: 小计
~~~~~~~~~~~~~~~~~~~~~~~

一些类被定义为在发票形式中呈现小计：

.. image:: forms/screenshot-00.png
   :class: img-responsive

::

    <group class="oe_subtotal_footer">
        <field name="amount_untaxed"/>
        <field name="amount_tax"/>
        <field name="amount_total" class="oe_subtotal_footer_separator"/>
        <field name="residual" style="margin-top: 10px"/>
    </group>

占位符和内联字段
..............................

有时字段标签使表单太复杂。可以省略字段标签，在字段中放置一个占位符。仅当字段为空时，占位符文本才可见。占位符应该告诉在字段中放置什么，它 *必须不能* 是一个例子，因为它们常常与填充数据混淆。

还可以通过一个明确的块元素，如 ``<div>`` 中呈现它们“内联”来将字段组合在一起。这允许对语义相关的字段进行分组，就好像它们是单个（复合）字段。

以下示例取自 *Leads* 表单, 同时显示占位符和内联字段 (邮编和城市).

.. list-table::
   :header-rows: 1

   * - Edit mode
     - View mode
   * - .. image:: forms/placeholder.png
          :class: img-responsive
     - .. image:: forms/screenshot-01.png
          :class: img-responsive

::

    <group>
        <label for="street" string="Address"/>
        <div>
            <field name="street" placeholder="Street..."/>
            <field name="street2"/>
            <div>
                <field name="zip" class="oe_inline" placeholder="ZIP"/>
                <field name="city" class="oe_inline" placeholder="City"/>
            </div>
            <field name="state_id" placeholder="State"/>
            <field name="country_id" placeholder="Country"/>
        </div>
    </group>

图片
......

图像，如头像，应显示在工作表的右侧。产品形式如下：

.. image:: forms/screenshot-02.png
   :class: img-responsive

上面的表单包含以sheet开头的<sheet>元素：

::

    <field name="product_image" widget="image" class="oe_avatar oe_right"/>

标签
....

最多 :class:`~odoo.fields.Many2many` 字段，类似，比较好呈现为标签列表。使用窗口
小部件 ``many2many_tags`` ：

.. image:: forms/screenshot-04.png
   :class: img-responsive

::

    <field name="category_id" widget="many2many_tags"/>

配置表单指南
------------------------------

配置形式的示例：阶段，离开类型等等。这涉及每个应用程序配置下的所有菜单项（如销售/配置）

.. image:: forms/nosheet.png
   :class: img-responsive

1. 没有标题（因为没有状态，没有工作流，没有按钮）
2. 没有表单

对话框表单指南
-----------------------

例如：从时机中“调度呼叫”

.. image:: forms/wizard-popup.png
   :class: img-responsive

1. 避免分隔符（标题已经在弹出的标题栏中，所以另外一个分隔符不相关）
2. 避免取消按钮（用户一般关闭弹出窗口得到相同的影响）
3. 动作按钮必须突出显示（红色）
4. 当有文本区域时，使用占位符，而不是标签或分隔符
5. 像在常规窗体视图中，在<header>元素中放置按钮

配置向导指南
--------------------------------

例如：设置/配置/销售

1. 总是在线（没有弹出）
2. 没有表单
3. 保持取消按钮（用户不能关闭窗口）
4. 按钮“应用”必须是红色


.. _reference/views/graph:

图表
======

图形视图用于可视化多个记录或记录组上的总数。根元素是 ``<graph>`` ，可以具有以下属性：

``type``
  ``bar`` (默认), ``pie`` 和 ``line`` 使用的图形类型
``stacked``
  只用于 ``bar`` 图表。如果存在并设置为 ``True``，则堆栈条在一个组内

图形视图中唯一允许的元素是 ``field`` ，它可以有以下属性：

``name`` (必填)
  要在图形视图中使用的字段名称。用于分组（而不是总和）

``type``
  指示该字段是否应用作分组标准或组内的总和值。可能的值为：

  ``row`` (默认)
    组按指定字段。所有图形的类型支持至少是一个级别的分组，一些可能支持更多。对于数据透视图，每个组都有自	己的行
  ``col``
    仅由数据透视表使用，按列组创建
  ``measure``
    字段在组内聚合

``interval``
  日期和日期时间字段，按指定的时间间隔分组(``day`` ，``week`` ， ``month`` ， ``quarter``
  或 ``year`` )而不是按特定的datetime上分组（固定二次决议）或日期（固定日决议）

.. warning::

   图形视图聚合对数据库内容执行，非存储函数字段不能再图形视图中使用

枢轴
------

枢轴视图用于将聚合可视化为 `pivot table`_。它的根元素是 ``<pivot>``，它可以具有以下属性。

``disable_linking``
  设置为 ``True`` 以将表单元格的链接删除到列表视图。
``display_quantity``
  默认情况下，设置为 ``true`` 以显示列数

在枢轴视图中允许的元素与图形视图中相同

.. _reference/views/kanban:

看板
======

看板视图是一个可视化的 `kanban board`_ ：它将记录显示为"卡片"，位于 :ref:`list view <reference/views/list>` 和一个不可编辑的 :ref:`form view <reference/views/form>`。记录可以按列分组以用于工作流可视化或操纵（例如，任务或工作进度管理）或未分组（仅用于可视化记录）

看板视图的根元素是 ``<kanban>`` ，可以使用以下属性：

``default_group_by``
  如果通过动作或当前搜索未指定分组，是否应该将看板视图分组。应该是要分组的字段的名称，否则不指定分组
``default_order``
  按顺序排序，如果用户尚未对记录进行排序（通过列表视图）
``class``
  将HTML类添加到“看板”视图的根HTML元素中
``quick_create``
  是否可以创建记录而不切换到表单视图。默认情况下，``quick_create`` 在看板视图分组时启用，
  否则禁用。

  如果设置为 ``true`` 则始终启用，否则为 ``false`` 则始终禁用。

视图元素的可能子元素为：

``field``
  声明要聚合或在看板 *逻辑* 中使用的字段。如果字段只显示在看板视图中，则不需要预先声明。

  可能的属性包括：

  ``name`` (必填)
    要提取的字段的名称
  ``sum``, ``avg``, ``min``, ``max``, ``count``
    显示相应的总数在看板列的顶部，该字段的值是总（字符串）的标签。仅支持每个字段的总动作。

``templates``
  定义了一个列表 :ref:`reference/qweb` 模板。为了能看的更清楚，卡片定义可以分割为多个模板，
  但是看版式图 *必须* 至少定义一个根模板 ``kanban-box``，它将为每个记录渲染一次。

  看板视图主要的使用标准 :ref:`javascript qweb <reference/qweb/javascript>` 并提供以下的上下文
  变量：

  ``instance``
    当前的 :ref:`reference/javascript/client` 实例
  ``widget``
    当前 :js:class:`KanbanRecord`，可以用来获取一些元信息。这些方法也可以直接在模板上下文中使用，
    不需要通过 ``widget`` 来访问
  ``record``
    具有所有请求字段作为其属性的对象。每个字段有两个属性 ``value`` 和 ``raw_value``，前者根据当前的
    用户参数进行格式化，后者是从a :meth:`~odoo.models.Model.read` (对于根据用户的区域设置格式化的
    日期和日期时间字段 `formatted according to user's locale
    <https://github.com/odoo/odoo/blob/a678bd4e/addons/web_kanban/static/src/js/
    kanban_record.js#L102>`_)
  ``formats``
    :js:class:`web.formats` 模块来操作和转换值
  ``read_only_mode``
    不言自明


    .. rubric:: 按钮和字段

    虽然大多数看板的模板是标准的 :ref:`reference/qweb`，特别是看板进程的 ``field`` ， ``button``
    和 ``a`` 元素：

    * 默认字段由其格式化的值替换，除非它们匹配特定的看板视图的窗口小部件

      .. todo:: list widgets?

    * 按钮和具有 ``type`` 属性的链接会执行Odoo相关的操作，而不是它们的标准HTML函数。可能的类型有：

      ``action``, ``object``
        标准行为 :ref:`Odoo buttons <reference/views/list/button>`，可以使用与标准Odoo按钮相
        关的大多数属性
      ``open``
        以只读模式在表单视图中打开卡的记录
      ``edit``
        在可编辑模式下以表单视图打开卡的记录
      ``delete``
        删除卡的记录并删除卡

    .. todo::

       * kanban-specific CSS
       * kanban structures/widgets (vignette, details, ...)

Javascript API
--------------

.. js:class:: KanbanRecord

   :js:class:`Widget` 处理单个记录到卡的渲染。在它自己的渲染模式上下文中作为 ``widget`` 可见。

   .. js:function:: kanban_color(raw_value)

      将颜色分割值转换为看板颜色类 :samp:`oe_kanban_color_{color_index}`。内置的CSS提供了一个
      ``color_index`` 为9的类。

   .. js:function:: kanban_getcolor(raw_value)

      将颜色分割值转换为颜色索引（默认情况下介于0到9之间）。颜色分割值可以是数字或字符串。

   .. js:function:: kanban_image(model, field, id[, cache][, options])

      生成指向指定字段的URL作为图片访问。

      :param String model: 模型托管图像
      :param String field: 保存图像数据的字段的名称
      :param id: 包含要显示的图像的记录的标识符
      :param Number cache: 应覆盖浏览器默认的缓存持续时间（以秒为单位）。 ``0`` 表示完全禁用缓存
      :returns: 图片网址

   .. js:function:: kanban_text_ellipsis(string[, size=160])

      剪辑超出指定大小的文本，并向其添加省略号。可以用于显示潜在非常长的字段（例如描述）的初始部分

.. _reference/views/calendar:

日历
========

日历视图将记录显示为每日，每周或每月日历中的事件。它们的根元素是 ``<calendar>`` 。日历视图上的可用属性包括：

``date_start`` (必填)
    保存事件的开始日期记录的字段名称
``date_stop``
    保存事件的结束日期记录字段的名称，如果提供了 ``date_stop`` 记录，则记录在日历中直接可移动（通
    过拖放）
``date_delay``
    替代 ``date_stop``，提供事件的持续事件，而不是结束日期

    .. todo:: what's the unit? Does it allow moving the record?

``color``
    用于 *颜色分割* 记录字段的名称。同一色段中的记录在日历中被分配相同的高亮颜色，颜色是随机地分配
``event_open_popup``
    在对话框中打开事件，而不是切换到表单视图，默认情况下禁用
``quick_add``
    在点击时启用快速事件创建：只询问用户一个 ``name``，并试图创建一个新的事件，只有那个和点击的事件时
    间。如果快速创建失败，则返回到完整表单对话框
``display``
    用于事件显示的格式字符串，字段名称应该在括号中 ``[`` and ``]``
``all_day``
    记录上的布尔字段的名称，其指示相应的事件是否被标记为day-long（并且持续时间是不相关的）
``mode``
    加载日历时默认显示模式。
    可能的属性有: ``day``, ``week``, ``month``


.. todo::

   what's the purpose of ``<field>`` inside a calendar view?

.. todo::

   calendar code is an unreadable mess, no idea what these things are:

   * ``attendee``
   * ``avatar_model``
   * ``use_contacts``

   calendar code also seems to refer to multiple additional attributes of
   unknown purpose

.. _reference/views/gantt:

甘特
=====

甘特视图适用于当地显示甘特图（用于调度）

甘特视图的根元素 ``<gantt/>``，它没有子元素，但是可以采取以下属性：

``date_start`` (必填)
  为每个记录提供事件的开始日期时间的字段名称。
``date_stop``
  为每个记录提供时间的结束持续时间的字段名称。可以替换为 ``date_delay`` 。必须提供 ``date_stop``
  和 ``date_delay`` 中的一个（且只有一个）

  如果记录的字段为 ``False`` ，则假定它是一个“点事件”，结束日期将设置为开始日期
``date_delay``
  提供时间持续时间的字段的名称
``duration_unit``
  其中之一 ``minute``, ``hour`` (默认), ``day``, ``week``, ``month``, ``year``

``default_group_by``
  要对任务进行分组的字段名
``type``
  ``gantt`` 经典甘特视图（默认）

  ``consolidate`` 第一个子项的值被合并在甘特的任务中

  ``planning`` 子项显示在甘特的任务中
``consolidation``
  字段名称以在记录单元中显示合并值
``consolidation_max``
  具有 "group by" 字段作为键的字典和在以红色显示单元格之间可以达到的最大合并值(例如 ``{"user_id":
  100}``)

  .. warning::
      字典定义必须使用双引号 ``{'user_id': 100}`` 不是有效的值
``string``
  要显示在合并值旁边的字符串，如果未指定，则将使用合并字段的标签
``fold_last_level``
  如果设置了值，则折叠最后一个分组级别
``round_dnd_dates``
  可以将任务的开始和结束日期四舍五入为最接近的刻度
``drag_resize``
  调整任务大小，默认为 ``true``

.. ``progress``
    name of a field providing the completion percentage for the record's event,
    between 0 and 100
.. consolidation_exclude
.. consolidation_color

.. _reference/views/diagram:

图表
=======

图表视图可用于显示记录的有向图。根元素是 ``<diagram>`` 不带任何属性。

图表视图的可能子项为：

``node`` (必填，1)
    定义图形的节点。其属性是：

    ``object``
      节点的Odoo模型
    ``shape``
      条件形状映射类似于颜色和字体 :ref:`the listview <reference/views/list>`。唯一有效的形状是
      ``矩形`` (默认形状是省略号)
    ``bgcolor``
      与 ``shape`` 相同，但有条件地为节点映射背景颜色。默认背景颜色是白色，唯一有效的替代是 ``grey`` 。

``arrow`` (必填，1)
    定义图像的有向边。其属性是：

    ``object`` (必填)
      边缘的Odoo模型
    ``source`` (必填)
      :class:`~odoo.fields.Many2one` 指向的边缘模型的字段的边缘的源节点记录
    ``destination`` (不可或缺)
      :class:`~odoo.fields.Many2one` 字段的边缘模型指向边缘的节点记录
    ``label``
      Python 属性列表 (引用字符串)。相应的属性的值将被连接并显示为边的标签

``label``
    图表的说明， ``string`` 属性定义了笔记的内容，每个 ``label`` 作为图表标题中的段落输出，容易看见，但没有特别强调

.. _reference/views/search:

搜索
======

搜索视图与以前的视图类型不同，它们不显示 *内容* ：虽然它们适用于特定的模型，它们用于过滤其它视图的内容(通常聚合视图，例如： ref:`reference/views/list` 或 :ref:`reference/views/graph` )。除了在用例中的差别之外，它们以相同的方式定义

搜索视图的根元素是 ``<search>`` 。它不需要属性。

.. @string is not displayed anywhere, should be removed

搜索视图的可能的子元素是：

``field``
    字段使用用户提供的值定义域或上下文。当生成搜索域时，字段域使用 **AND** 与另一个域和过滤器组合。

    字段可以具有以下属性：

    ``name``
        要过滤的字段的名称
    ``string``
        字段的标签
    ``operator``
        默认情况下，字段生成以下形式的域 :samp:`[({name}，{operator}, {provided_value})]` 其中
        ``name`` 是字段的名称，``provided_value`` 用户，可能被过滤或变换 (例如， 用户期望提供选择
        字段的值的 *label* ，而不是值本身).

        ``operator`` 属性允许覆盖默认的操作符，这取决于字段的类型(例如， ``=`` 用于浮动字段，
        ``ilike`` 用于char字段)
    ``filter_domain``
        完整的域用作字段的搜索域，可以使用 ``self`` 变量在自定义域中写入提供的值。可以用于生成比
        ``operator`` 本身更加灵活的域 (例如同时搜索多个字段)

        如果提供了 ``operator`` 和 ``filter_domain`` ，``filter_domain`` 优先。
    ``context``
        允许添加上下文键， 包括用户提供的值 (对于 ``domain`` 是可用的 ``self`` 变量)。默认情况下，
        字段不生成域

        .. note:: 域和上下文是包含性的，并且如果指定了 ``上下文``，则都会生成。要仅生成上下文值，请将
        ``filter_domain`` 设置为空列表：``filter_domain="[]"``
    ``groups``
        使该字段仅对特定用户可用
    ``widget``
        使用特定的搜索小部件的字段 (标准Odoo 8.0 中的唯一用例是一个 ``selection`` 小部件
        :class:`~odoo.fields.Many2one` fields)
    ``domain``
        如果字段可以提供自动完成(例如 :class:`~odoo.fields.Many2one`)，过滤可能的完成结果
``filter``
    过滤器是搜索视图中的预定义切换，它只能启用或禁用。它的主要目的是将数据添加到搜索上下文（传递到数据视
    图进行搜索/过滤的上下文），或者将新的部分添加到搜索过滤器

    过滤器可以具有以下属性：

    ``string`` (必填)
        过滤器的标签
    ``domain``
        一个 Odoo :ref:`domain <reference/orm/domains>` ，将作为搜索域的一部分附加到操作域中
    ``context``
        一个Python字典，合并到操作域中以生成搜索域
    ``name``
        过滤器的逻辑名，可 :ref:`默认情况下启动<reference/views/search/
        defaults>`，也可 :ref:`继承 <reference/views/inheritance>`
    ``help``
        过滤器的更长的说明文本，可以显示为工具提示
    ``groups``
        是过滤器仅适用于特定用户

    .. note::

       .. versionadded:: 7.0

       过滤器的顺序 (没有分离它们的非过滤器) 被视为包含合成的：它们将由 ``OR`` 构成，而不是通常的
       ``AND``。

       ::

          <filter domain="[('state', '=', 'draft')]"/>
          <filter domain="[('state', '=', 'done')]"/>

       如果选择了两个过滤器，将选择 ``state`` 是 ``draft`` 或 ``done`` 的记录，但是

       ::

          <filter domain="[('state', '=', 'draft')]"/>
          <separator/>
          <filter domain="[('delay', '<', 15)]"/>

       如果两个过滤器都被选择，将选择 ``state`` 是 ``draft`` **and** ``delay`` 在15以下的记录。

``separator``
    可用于在简单搜索视图中分离过滤器组
``group``
    可以用于分离过滤器组，在复杂搜索视图中比 ``separator`` 更易读

.. _reference/views/search/defaults:

搜索默认值
---------------

搜索字段和过滤器可以通过操作 ``context`` 使用 ： samp:`search_default_{name}` keys来配置。对于字段，值应该是在字段中设置的值，对于过滤器，它是一个布尔值。例如：假设 ``foo`` 是一个字段， ``bar`` 是一个动作上下文的过滤器：

.. code-block:: python

  {
    'search_default_foo': 'acro',
    'search_default_bar': 1
  }

将自动启用 ``bar`` 过滤器并搜索 *acro* 的 ``foo`` 字段


.. _reference/views/qweb:

QWeb
====

QWeb 视图是标准的 ： ref:`reference/qweb` 模板在视图的 ``arch`` 。它们没有特定的根元素。

QWeb视图只能包含一个模板\ [#template_inherit]_，而模板名称 *必须* 匹配视图的完整(包括模块名称)
:term:`external id`.

:ref:`reference/data/template` 应该用作定义QWeb视图的快捷方式。

.. [#backwards-compatibility] 用于向后兼容的原因
.. [#hasclass] 添加了一个扩展函数，用于在QWeb视图中进行更简单的匹配： ``hasclass(*classes)`` 如果
				上下文节点具有所有指定的类
.. [#treehistory] 由于历史原因，它的起源于树形视图，稍后被重新用于更多的表/列表类型显示
.. [#template_inherit] 或者没有模板，如果它是继承的视图，则 :ref:`它应该只包含xpath元素
                       <reference/views/inheritance>`

.. _accesskey: http://www.w3.org/TR/html5/editing.html#the-accesskey-attribute
.. _CSS color unit: http://www.w3.org/TR/css3-color/#colorunits
.. _floats: https://developer.mozilla.org/en-US/docs/Web/CSS/float
.. _HTML: http://en.wikipedia.org/wiki/HTML
.. _kanban board: http://en.wikipedia.org/wiki/Kanban_board
.. _pivot table: http://en.wikipedia.org/wiki/Pivot_table
.. _XPath: http://en.wikipedia.org/wiki/XPath
