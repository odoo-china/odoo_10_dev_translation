:banner: banners/odoo_guideline.jpg

.. highlight:: python

===============
Odoo 指南
===============

本页介绍了新的Odoo编码指南。 这些旨在提高代码的质量（例如源代码更好的可读性）和Odoo应用程序。 事实上，正确的代码简化了维护，帮助调试，降低复杂性，提高可靠性。

这些指南应适用于每个新模块和新开发。 这些指南 *仅* 在代码重构（迁移到新的API，大重构，...）将应用于旧模块

.. warning::

    这些指南是用新模块和新文件编写的。 修改现有文件时，文件的原始样式将严格取代任何其他样式指南。 换句话说，不要修改现有文件以应用这些准则，以避免中断每行的修订历史记录。 有关更多详细信息，
    请参阅我们的 `pull request guide <https://odoo.com/submit-pr>`_ 。

模块结构
================

目录
-----------
模块被组织在重要的目录中。那些包含业务逻辑; 看看它们应该明白模块的目的。

- *data/* : 演示和数据xml
- *models/* : 模型定义
- *controllers/* : 包含控制器（HTTP路由）
- *views/* : 包含视图和模板
- *static/* : 包含web资源，放置在 *css/, js/, img/, lib/, ...*

其他可选目录组成模块。

- *wizard/* : 重新分组短暂的模型（以前的 *osv_memory*）及其视图。
- *report/* : 包含报表（RML报表 **[已弃用]** ，基于SQL视图的模型（用于报表）和其他复杂报表）。 Python目录和XML视图包含在此目录中。
- *tests/* : 包含Python/YML测试


文件命名
-----------
对于 *views* 声明，在2个不同的文件中从（前端）模板拆分后端视图

对于 *models* ，将业务逻辑按模型集拆分，在每个集合中选择一个主模型，此模型将其名称赋给集合。 如果只有一个模型，其名称与模块名称相同。 对于每个名为<main_model>的集可以创建以下文件:

- :file:`models/{<main_model>}.py`
- :file:`models/{<inherited_main_model>}.py`
- :file:`views/{<main_model>}_templates.xml`
- :file:`views/{<main_model>}_views.xml`

例如， *sale* 模块引入  ``sale_order`` 和 ``sale_order_line`` ， ``sale_order``  是主导的。
所以  ``<main_model>``  文件将命名为 :file:`models / sale_order.py` 和 :file:`views / sale_order_views.py` 。

对于 *data* ，按目的分割：演示或数据。 文件名将为
main_model名称，后缀为 *_demo.xml* 或 *_data.xml* 。


对于 *controllers* ，唯一的文件应该命名为 *main.py* 。否则，如果您需要从另一个模块继承现有控制器，
则其名称将为 *<module_name>.py* 。 与 *models* 不同，每个控制器类应该包含在一个单独的文件中。

对于 *static文件* ，由于资源可以在不同的上下文（前端，后端，两者）中使用，它们将只包含在一个bundle中。因此，
CSS/Less，JavaScript和XML文件应该使用bundle类型的名称作为后缀。即：“assets_common”包的 *im_chat_common.css* ，
*im_chat_common.js* ，“assets_backend”包的 *im_chat_backend.css* ， *im_chat_backend.js*  。
如果模块只有一个文件，那么约定将是 *<module_name>.ext* （即：*project.js* ）。
不要链接Odoo之外的数据（图像，库）：不要使用图像的URL，而是将其复制到我们的代码库中。

关于 *data* ，按目的分割：数据或演示。 文件名将为 *main_model* 名称，后缀为 *_data.xml* 或 *_demo.xml* 。

关于 *wizards* ，命名约定是：

- :file:`{<main_transient>}.py`
- :file:`{<main_transient>}_views.xml`

Where *<main_transient>* is the name of the dominant transient model, just like for *models*. <main_transient>.py can contains the models 'model.action' and 'model.action.line'.

其中 *<main_transient>* 是主导短暂模型的名称，就像 *models* 一样。
<main_transient>.py可以包含模型'model.action'和'model.action.line'

对于 *统计报告* ，其名称应如下所示：

- :file:`{<report_name_A>}_report.py`
- :file:`{<report_name_A>}_report_views.py` (pivot和graph视图)

对于 *可打印的报告* ，你应该有 :

- :file:`{<print_report_name>}_reports.py` (报表动作，纸格格式定义， ...)
- :file:`{<print_report_name>}_templates.xml` (xml 报表模板)


完整的目录结构应该看起来像：

.. code-block:: text

    addons/<my_module_name>/
    |-- __init__.py
    |-- __manifest__.py
    |-- controllers/
    |   |-- __init__.py
    |   |-- <inherited_module_name>.py
    |   `-- main.py
    |-- data/
    |   |-- <main_model>_data.xml
    |   `-- <inherited_main_model>_demo.xml
    |-- models/
    |   |-- __init__.py
    |   |-- <main_model>.py
    |   `-- <inherited_main_model>.py
    |-- report/
    |   |-- __init__.py
    |   |-- <main_stat_report_model>.py
    |   |-- <main_stat_report_model>_views.xml
    |   |-- <main_print_report>_reports.xml
    |   `-- <main_print_report>_templates.xml
    |-- security/
    |   |-- ir.model.access.csv
    |   `-- <main_model>_security.xml
    |-- static/
    |   |-- img/
    |   |   |-- my_little_kitten.png
    |   |   `-- troll.jpg
    |   |-- lib/
    |   |   `-- external_lib/
    |   `-- src/
    |       |-- js/
    |       |   `-- <my_module_name>.js
    |       |-- css/
    |       |   `-- <my_module_name>.css
    |       |-- less/
    |       |   `-- <my_module_name>.less
    |       `-- xml/
    |           `-- <my_module_name>.xml
    |-- views/
    |   |-- <main_model>_templates.xml
    |   |-- <main_model>_views.xml
    |   |-- <inherited_main_model>_templates.xml
    |   `-- <inherited_main_model>_views.xml
    `-- wizard/
        |-- <main_transient_A>.py
        |-- <main_transient_A>_views.xml
        |-- <main_transient_B>.py
        `-- <main_transient_B>_views.xml

.. note:: 文件名只能包含  ``[a-z0-9_]`` （小写字母数字和 ``_``  ）

.. warning:: 使用正确的文件权限：文件夹755和文件644。

XML 文件
=========

格式
------
要以XML格式声明记录，建议使用 **record** 符号（使用 *<record>* ）：

- 在  ``model`` 之前放置 ``id``  属性
- 对于字段声明， ``name`` 属性是第一个。 然后将 *值* 放在 ``field`` 标签中，
  或者在 ``eval``  属性中，最后是其他按重要性排序的属性（widget，options，...）。

- 尝试按模型分组记录。 如果操作/菜单/视图之间存在依赖关系，则此约定可能不适用。
- 使用在下一点定义的命名约定
- 标签 *<data>* 仅用于设置不可更新的数据  ``noupdate=1``

.. code-block:: xml

    <record id="view_id" model="ir.ui.view">
        <field name="name">view.name</field>
        <field name="model">object_name</field>
        <field name="priority" eval="16"/>
        <field name="arch" type="xml">
            <tree>
                <field name="my_field_1"/>
                <field name="my_field_2" string="My Label" widget="statusbar" statusbar_visible="draft,sent,progress,done" />
            </tree>
        </field>
    </record>

Odoo支持充当语法糖的自定义标签：

- menuitem: 使用它作为一个快捷方式来声明一个  ``ir.ui.menu``
- workflow: <workflow>标签会向现有工作流发送信号。
- template: 使用它来声明一个QWeb视图只需要 ``arch`` 部分的视图。
- report: 使用声明 :ref:`report action <reference/actions/report>`
- act_window: 使用它，如果record符号不能做你想要的

这4个第一标签优先于 *record* 符号。

命名 xml_id
-------------

安全，视图和动作
~~~~~~~~~~~~~~~~~~~~~~~~~

使用以下模式 :

* 菜单: :samp:`{<model_name>}_menu`
* 视图: :samp:`{<model_name>}_view_{<view_type>}` ， *view_type* 是
   ``kanban``, ``form``, ``tree``, ``search``  ， ...
* 动作: 主要动作方面 :samp:`{<model_name>}_action`.
  其他有后缀 :samp:`_{<detail>}`,其中 *detail* 是简短解释动作的小写字符串。
  仅当为模型声明多个动作时，才使用此选项.
* 组: :samp:`{<model_name>}_group_{<group_name>}` 其中
  *group_name* 是组的名称，通常是'user'，'manager'
* 规则: :samp:`{<model_name>}_rule_{<concerned_group>}` 其中 *concerned_group*
  是相关组的简称（'model_name_group_user'的'user'，公共用户的'public'，多公司规则的'company'...）

.. code-block:: xml

    <!-- views and menus -->
    <record id="model_name_view_form" model="ir.ui.view">
        ...
    </record>

    <record id="model_name_view_kanban" model="ir.ui.view">
        ...
    </record>

    <menuitem
        id="model_name_menu_root"
        name="Main Menu"
        sequence="5"
    />
    <menuitem
        id="model_name_menu_action"
        name="Sub Menu 1"
        parent="module_name.module_name_menu_root"
        action="model_name_action"
        sequence="10"
    />

    <!-- actions -->
    <record id="model_name_action" model="ir.actions.act_window">
        ...
    </record>

    <record id="model_name_action_child_list" model="ir.actions.act_window">
        ...
    </record>

    <!-- security -->
    <record id="module_name_group_user" model="res.groups">
        ...
    </record>

    <record id="model_name_rule_public" model="ir.rule">
        ...
    </record>

    <record id="model_name_rule_company" model="ir.rule">
        ...
    </record>



.. note:: 视图名称使用点符号  ``my.model.view_type`` 或 ``my.model.view_type.inherit``
          代替 *“这是我的模型的表单视图”* 。


继承 XML
~~~~~~~~~~~~~

继承视图的命名模式是 :samp:`{<base_view>} _ inherit _ {<current_module_name>}` 。
模块只能扩展一次视图。 后缀名为 :samp:`_inherit _ {<current_module_name>}` 其中
*current_module_name* 是扩展视图的模块的技术名称。


.. code-block:: xml

    <record id="inherited_model_view_form_inherit_my_module" model="ir.ui.view">
        ...
    </record>


Python
======

PEP8 选项
------------

使用linter可以帮助显示语法和语义警告或错误。 Odoo源代码试图尊照Python标准，但其中一些可以忽略。

- E501: 行太长
- E301: 期待1个空行，发现0个
- E302: 期待2个空行，发现1个
- E126: continuation line over-indented for hanging indent
- E123: 闭括号与开括号的行缩进不匹配
- E127: continuation line over-indented for visual indent
- E128: continuation line under-indented for visual indent
- E265: 块注释应以“＃ ”开头

Imports
-------
import 排序为

#. 外部库（每行一个，在python stdlib中排序和拆分）
#. 导入  ``odoo``
#. 从Odoo模块导入（很少，只有在必要时）

在这3组中，导入的行按字母顺序排序。

.. code-block:: python

    # 1 : imports of python lib
    import base64
    import re
    import time
    from datetime import datetime
    # 2 :  imports of odoo
    import odoo
    from odoo import api, fields, models # alphabetically ordered
    from odoo.tools.safe_eval import safe_eval as eval
    from odoo.tools.translate import _
    # 3 :  imports from odoo modules
    from odoo.addons.website.models.website import slug
    from odoo.addons.web.controllers.main import login_redirect


Python编程习惯
-----------------------------

- 每个python文件应该有 ``# -*- coding：utf-8 -*-``  作为第一行。
- Always favor *readability* over *conciseness* or using the language features or idioms.
- 总是支持*可读性*超过*简洁性*或使用语言特性或惯用语法。
- 不要使用  ``.clone()``

.. code-block:: python

    # bad
    new_dict = my_dict.clone()
    new_list = old_list.clone()
    # good
    new_dict = dict(my_dict)
    new_list = list(old_list)

- Python字典：创建和更新

.. code-block:: python

    # -- creation empty dict
    my_dict = {}
    my_dict2 = dict()

    # -- creation with values
    # bad
    my_dict = {}
    my_dict['foo'] = 3
    my_dict['bar'] = 4
    # good
    my_dict = {'foo': 3, 'bar': 4}

    # -- update dict
    # bad
    my_dict['foo'] = 3
    my_dict['bar'] = 4
    my_dict['baz'] = 5
    # good
    my_dict.update(foo=3, bar=4, baz=5)
    my_dict = dict(my_dict, **my_dict2)

- 使用有意义的变量/类/方法名称
- 无用变量：临时变量可以通过为对象赋予名称来使代码更清晰，
  但这并不意味着您应该始终创建临时变量：

.. code-block:: python

    # pointless
    schema = kw['schema']
    params = {'schema': schema}
    # simpler
    params = {'schema': kw['schema']}

- 多个返回点是可以的，当它们更简单

.. code-block:: python

    # a bit complex and with a redundant temp variable
    def axes(self, axis):
            axes = []
            if type(axis) == type([]):
                    axes.extend(axis)
            else:
                    axes.append(axis)
            return axes

     # clearer
    def axes(self, axis):
            if type(axis) == type([]):
                    return list(axis) # clone the axis
            else:
                    return [axis] # single-element list

- 了解内建函数：你至少应该对所有的Python内建有一个基本的了解（http://docs.python.org/library/functions.html）

.. code-block:: python

    value = my_dict.get('key', None) # very very redundant
    value= my_dict.get('key') # good

同时， ``if 'key' in my_dict`` 和 ``if my_dict.get('key')`` 具有非常不同的意义，
要确保你使用的是正确的。

- 学习列表解析：使用列表解析，字典解析，和基本的操作使用  ``map`` ， ``filter`` ， ``sum``  ，
  他们使代码更容易阅读。

.. code-block:: python

    # not very good
    cube = []
    for i in res:
            cube.append((i['id'],i['name']))
    # better
    cube = [(i['id'], i['name']) for i in res]

- 集合也是布尔值：在python中，许多对象在布尔上下文（例如if）中求值时具有“boolean-ish”值。
  其中当集合为空时是“false” 和 包含项目时的“truthy”的集合（列表，字典，集合...）

.. code-block:: python

    bool([]) is False
    bool([1]) is True
    bool([False]) is True

因此，你可以写 ``if some_collection:`` 替代 ``if len(some_collection):``  .


- 迭代

.. code-block:: python

    # creates a temporary list and looks bar
    for key in my_dict.keys():
            "do something..."
    # better
    for key in my_dict:
            "do something..."
    # creates a temporary list
    for key, value in my_dict.items():
            "do something..."
    # only iterates
    for key, value in my_dict.iteritems():
            "do something..."

- 使用 dict.setdefault

.. code-block:: python

    # longer.. harder to read
    values = {}
    for element in iterable:
        if element not in values:
            values[element] = []
        values[element].append(other_value)

    # better.. use dict.setdefault method
    values = {}
    for element in iterable:
        values.setdefault(element, []).append(other_value)

- 作为一个好的开发人员，记录你的代码（文档字符串的方法，简单注释棘手的代码部分）
- 除了这些指南，您还可以找到以下有趣链接：
  http://python.net/~goodger/projects/pycon/2007/idiomatic/handout.html
  （有点过时，但相当相关）

在Odoo中编程
-------------------

- 避免创建生成器和装饰器：只使用Odoo API提供的。
- 在python中，使用  ``filtered``，``mapped``，``sorted``  ,...方法来简化代码读取和性能 。


使您的方法批量工作
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

添加函数时，请确保它可以处理多个记录。 通常，这样的方法是用 ``api.multi`` 装饰器装饰的
（或者如果在旧的api中写的话，就是一个 *id* 的列表）。 然后你必须在 ``self`` 上进行迭代来处理每个记录。

.. code-block:: python

    @api.multi
    def my_method(self)
        for record in self:
            record.do_cool_stuff()

避免使用 ``api.one`` 装饰器：这可能不会做你想要的，扩展一个这样的方法不如 *api.multi* 方法那么容易，因为它返回一个结果列表（按记录集排序ids）。

对于性能问题，当开发“stat按钮”（例如）时，不要在  ``api.multi`` 方法中循环执行 ``search`` 或 ``search_count`` 。
建议使用 ``read_group`` 方法，只计算一个请求中的所有值。

.. code-block:: python

    @api.multi
    def _compute_equipment_count(self):
    """ Count the number of equipement per category """
        equipment_data = self.env['hr.equipment'].read_group([('category_id', 'in', self.ids)], ['category_id'], ['category_id'])
        mapped_data = dict([(m['category_id'][0], m['category_id_count']) for m in equipment_data])
        for category in self:
            category.equipment_count = mapped_data.get(category.id, 0)


传播上下文
~~~~~~~~~~~~~~~~~~~~~

在新的API中，上下文是一个 ``frozendict`` ，不能被修改。 要调用具有不同上下文的方法，
应该使用 ``with_context`` 方法:

.. code-block:: python

    records.with_context(new_context).do_stuff() # all the context is replaced
    records.with_context(**additionnal_context).do_other_stuff() # additionnal_context values override native context ones

在上下文中传递参数可能具有危险的副作用。 由于值是自动传播的，因此可能会出现一些行为。
在上下文中调用具有 *default_my_field* 键的模型的  ``create()``  方法将为相关模型 *my_field* 字段设置默认值。 但如果固化此创建，其他对象（如sale.order.line，在sale.order上创建）具有字段名称 *my_field* ，它们的默认值也将设置。

如果您需要创建影响某个对象行为的关键上下文，请选择一个好的名称，最后使用模块名称作为前缀，
以隔离其影响。 一个很好的例子是 ``mail`` 模块的键 *mail_create_nosubscribe* ， *mail_notrack* ，
*mail_notify_user_signature* ...：


不要绕过ORM
~~~~~~~~~~~~~~~~~~~~~
当ORM可以做同样的事情时，你不应该直接使用数据库游标！ 通过这样做，您将绕过所有的ORM功能，可能的事务，访问权限等。

And chances are that you are also making the code harder to read and probably
less secure.
并且造成使代码更难读，可能不安全的可能。

.. code-block:: python

    # very very wrong
    self.env.cr.execute('SELECT id FROM auction_lots WHERE auction_id in (' + ','.join(map(str, ids))+') AND state=%s AND obj_price > 0', ('draft',))
    auction_lots_ids = [x[0] for x in self.env.cr.fetchall()]

    # no injection, but still wrong
    self.env.cr.execute('SELECT id FROM auction_lots WHERE auction_id in %s '\
               'AND state=%s AND obj_price > 0', (tuple(ids), 'draft',))
    auction_lots_ids = [x[0] for x in self.env.cr.fetchall()]

    # better
    auction_lots_ids = self.search([('auction_id','in',ids), ('state','=','draft'), ('obj_price','>',0)])


No SQL 注入!
~~~~~~~~~~~~~~~~~~~~~~~~~~~
当使用手动SQL查询时，必须注意不要引入SQL注入漏洞。 当用户输入未正确过滤或引用不当时，会出现漏洞，允许攻击者向SQL查询引入不合意的子句
（例如绕过过滤器或执行UPDATE或DELETE命令）。

最安全的方法是永远不要使用Python字符串连接（+）或字符串参数插值（％）将变量传递给SQL查询字符串。

第二个原因，几乎同样重要，是数据库抽象层（psycopg2）的工作决定如何格式化查询参数，而不是你的工作！ 例如psycopg2知道当你传递一个值的列表，它需要将它们格式化为逗号分隔的列表，括在括号中！

.. code-block:: python

    # the following is very bad:
    #   - it's a SQL injection vulnerability
    #   - it's unreadable
    #   - it's not your job to format the list of ids
    self.env.cr.execute('SELECT distinct child_id FROM account_account_consol_rel ' +
               'WHERE parent_id IN ('+','.join(map(str, ids))+')')

    # better
    self.env.cr.execute('SELECT DISTINCT child_id '\
               'FROM account_account_consol_rel '\
               'WHERE parent_id IN %s',
               (tuple(ids),))

这是非常重要的，所以在重构时请小心，最重要的是不要复制这些模式！

这里是一个难忘的例子，帮助你记住问题是什么（但不要复制代码）。
继续之前，请务必阅读pyscopg2的在线文档，以了解如何正确使用它：

- 查询参数的问题 (http://initd.org/psycopg/docs/usage.html#the-problem-with-the-query-parameters)
- 如何使用psycopg2传递参数 (http://initd.org/psycopg/docs/usage.html#passing-parameters-to-sql-queries)
- 高级参数类型 (http://initd.org/psycopg/docs/usage.html#adaptation-of-python-values-to-sql-types)


尽可能保持您的方法简短/简单
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
函数和方法不应该包含太多的逻辑：拥有很多小而简单的方法比使用少量大而复杂的方法更可取。
一个好的经验法则是尽快拆分方法：
- 它有不止一个作用 (参阅 http://en.wikipedia.org/wiki/Single_responsibility_principle)
- 它太大，不适合在一个屏幕上。

此外，相应地命名函数：小的和正确命名的函数是可读/可维护代码和更严格的文档的起点。

此建议也与类，文件，模块和包相关。(参阅 http://en.wikipedia.org/wiki/Cyclomatic_complexity)


不要提交事务
~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Odoo框架负责为所有RPC调用提供事务上下文。 原理是在每个RPC调用开始时打开一个新的数据库游标，并在调用返回时提交，就在将响应发送到RPC客户端之前，
大致如下：

.. code-block:: python

    def execute(self, db_name, uid, obj, method, *args, **kw):
        db, pool = pooler.get_db_and_pool(db_name)
        # create transaction cursor
        cr = db.cursor()
        try:
            res = pool.execute_cr(cr, uid, obj, method, *args, **kw)
            cr.commit() # all good, we commit
        except Exception:
            cr.rollback() # error, rollback everything atomically
            raise
        finally:
            cr.close() # always close cursor opened manually
        return res

如果在执行RPC调用期间发生任何错误，则会以原子方式回滚事务，从而保留系统的状态。

类似地，系统还在测试套件执行期间提供专用事务，因此可以根据服务器启动选项进行回滚或不回滚。

结果是，如果你手动调用  ``cr.commit()``  在任何地方有很高的可能，你会以各种方式打破系统，
因为你会导致部分提交，因此部分和不完整的回滚，导致 其他：

#. 不一致的业务数据，通常是数据丢失
#. 工作流去同步，文档永久停滞
#. 无法完全回滚的测试，并且将开始污染数据库，并触发错误（即使在事务期间没有发生错误，也是如此）

这里是非常简单的规则：
    你应该 *永远不要* 自己调用  ``cr.commit()``  ， **除非** 你已经创建了你自己的数据库游标！
    而你需要做的那些情况是例外！
    顺便说一下，如果你创建了自己的游标，那么你需要处理错误情况和适当的回滚，
    以及在你完成后正确关闭光标。

与人们的信念相反，你甚至不需要在下面的情况下调用 ``cr.commit()`` ：
- 在 *models.Model* 对象的  ``_auto_init()``  方法中：
这是由插件初始化方法或创建自定义模型时的ORM事务处理的
- 在报表中： ``commit（）`` 也是由框架处理的，所以你甚至在报表中可以更新数据库。
- 在 *models.Transient* 方法中：这些方法在一个事务中类似于普通 *models.Model* 那样被调用，并在最后对应  ``cr.commit()/rollback()``
- 其他. (看到上述规则如果你有疑问！)

所有  ``cr.commit()``  从现在开始在服务器框架之外调用必须有一个 **显式注释** 解释为什么它们是绝对必要的，为什么它们确实是正确的，以及为什么他们不打破事务。 否则他们可以和将被删除！


正确使用翻译方法
~~~~~~~~~~~~~~~~~~~~~

Odoo使用名为“下划线”  ``_()``  的类似GetText方法表示代码中使用的静态字符串需要在运行时
使用上下文的语言进行翻译。 这个伪方法在代码中通过导入访问，如下所示：

.. code-block:: python

    from odoo.tools.translate import _

在使用它时，必须遵循一些非常重要的规则，以便它工作，并避免用无用的垃圾填充翻译。

基本上，此方法只应用于在代码中手动编写的静态字符串，它不会用于翻译字段值，例如产品名称等。
这必须使用在使用translate标志的相应字段上。

规则很简单：对下划线方法的调用应该始终是  ``_('literal string')``  的形式，没有其他的：

.. code-block:: python

    # good: plain strings
    error = _('This record is locked!')

    # good: strings with formatting patterns included
    error = _('Record %s cannot be modified!') % record

    # ok too: multi-line literal strings
    error = _("""This is a bad multiline example
                 about record %s!""") % record
    error = _('Record %s cannot be modified' \
              'after being validated!') % record

    # bad: tries to translate after string formatting
    #      (pay attention to brackets!)
    # This does NOT work and messes up the translations!
    error = _('Record %s cannot be modified!' % record)

    # bad: dynamic string, string concatenation, etc are forbidden!
    # This does NOT work and messes up the translations!
    error = _("'" + que_rec['question'] + "' \n")

    # bad: field values are automatically translated by the framework
    # This is useless and will not work the way you think:
    error = _("Product %s is out of stock!") % _(product.name)
    # and the following will of course not work as already explained:
    error = _("Product %s is out of stock!" % product.name)

    # bad: field values are automatically translated by the framework
    # This is useless and will not work the way you think:
    error = _("Product %s is not available!") % _(product.name)
    # and the following will of course not work as already explained:
    error = _("Product %s is not available!" % product.name)

    # Instead you can do the following and everything will be translated,
    # including the product name if its field definition has the
    # translate flag properly set:
    error = _("Product %s is not available!") % product.name


另外，请记住，翻译者必须使用传递给下划线函数的文字值，因此请尽量让它们易于理解，
并将伪字符和格式设置为最小。 翻译者必须注意格式化模式（如％s或％d，换行符等）需要保留，
但重要的是以明智的方式使用这些模式：

.. code-block:: python

    # Bad: makes the translations hard to work with
    error = "'" + question + _("' \nPlease enter an integer value ")

    # Better (pay attention to position of the brackets too!)
    error = _("Answer to question %s is not valid.\n" \
              "Please enter an integer value.") % question

一般在Odoo中，当操作字符串时，喜欢 ``%`` 大于 ``.format()`` （当在字符串中只有一个要替换的变量），
并且喜欢 ``％(varname)``  当多个变量必须被替换时）。 这使得翻译更容易为社区翻译。


符号和约定
-----------------------

- 模型名称（使用点符号，前缀由模块名称）:
    - 定义Odoo模型时：使用名称的单数形式（ *res.partner* 和 *sale.order* 而不是
      *res.partnerS* 和 *saleS.orderS* ）
    - 当定义一个Odoo Transient（向导）：使用 ``<related_base_model>.<action>`` 其中 *related_base_model*
      是与瞬态相关的基本模型（在 *models/* 中定义），*action* 的瞬态做什么。 例如：  ``account.invoice.make``  ，
      ``project.task.delegate.batch`` ，...
    - 当定义 *report* 模块（SQL视图）：使用  ``<related_base_model>.report.<action>``  ，基于瞬态约定。

- Odoo Python类：在api v8（面向对象样式）中使用camelcase代码，为旧api（SQL样式）使用下划线小写符号。


.. code-block:: python

    class AccountInvoice(models.Model):
        ...

    class account_invoice(osv.osv):
        ...

- 变量名 :
    - 使用驼峰模型变量
    - 对公共变量使用下划线小写符号。
    - 因为新API与记录或记录集而不是id列表一起使用，如果变量名不包含id或id列表，
      请不要使用 *_id* 或 *_ids* 后缀变量名。

.. code-block:: python

    ResPartner = self.env['res.partner']
    partners = ResPartner.browse(ids)
    partner_id = partners[0].id

-  ``One2Many`` 和 ``Many2Many``  字段应该总是有 *_ids* 作为后缀（例如：sale_order_line_ids）

-  ``Many2One``  字段应该有 *_id* 作为后缀（例如：partner_id，user_id，...）

- 方法约定
    - 计算字段 : 计算方法模式为 *_compute_<field_name>*
    - 搜索方法 : 搜索方法模式为 *_search_<field_name>*
    - 默认方法 : 默认方法模式为 *_default_<field_name>*
    - Onchange方法 : onchange方法模式为 *_onchange_<field_name>*
    - 约束方法 : 约束方法模式为 *_check_<constraint_name>*
    - 操作方法 : 对象操作方法是带 *action_* 的前缀。 它的装饰是 ``@api.multi`` ，但由于它只使用一个记录，添加 ``self.ensure_one()`` 在方法的开始。


- 在一个Model属性顺序应该是
    #. 私有属性（  ``_name`` ， ``_description`` ， ``_inherit``  ，...）
    #. 默认方法和  ``_default_get``
    #. 字段声明
    #. 以与字段声明相同的顺序计算和搜索方法
    #. 约束方法（  ``@api.constrains`` ）和onchange方法（ ``@api.onchange``  ）
    #. CRUD方法（ORM覆盖）
    #. 动作方法
    #. 最后，其他的业务方法。

.. code-block:: python

    class Event(models.Model):
        # Private attributes
        _name = 'event.event'
        _description = 'Event'

        # Default methods
        def _default_name(self):
            ...

        # Fields declaration
        name = fields.Char(string='Name', default=_default_name)
        seats_reserved = fields.Integer(oldname='register_current', string='Reserved Seats',
            store=True, readonly=True, compute='_compute_seats')
        seats_available = fields.Integer(oldname='register_avail', string='Available Seats',
            store=True, readonly=True, compute='_compute_seats')
        price = fields.Integer(string='Price')

        # compute and search fields, in the same order of fields declaration
        @api.multi
        @api.depends('seats_max', 'registration_ids.state', 'registration_ids.nb_register')
        def _compute_seats(self):
            ...

        # Constraints and onchanges
        @api.constrains('seats_max', 'seats_available')
        def _check_seats_limit(self):
            ...

        @api.onchange('date_begin')
        def _onchange_date_begin(self):
            ...

        # CRUD methods (and name_get, name_search, ...) overrides
        def create(self, values):
            ...

        # Action methods
        @api.multi
        def action_validate(self):
            self.ensure_one()
            ...

        # Business methods
        def mail_user_confirm(self):
            ...


Javascript 和 CSS
==================
**对于javascript :**

-  ``use strict;``  推荐用于所有的javascript文件
- 使用linter（jshint，...）
- 不要添加缩小的Javascript库
- 使用camelcase进行类声明
- 除非你的代码应该在每一页上运行，使用网站模块的  ``if_dom_contains``  函数来定位特定的页面。
  定位特定于您的代码需要使用JQuery运行的页面的元素。

.. code-block:: javascript

    odoo.website.if_dom_contains('.jquery_class_selector', function () {
        /*your code here*/
    });


**对于 CSS :**

- 所有类前缀用 *o_<module_name>* 其中 *module_name*
  是模块的技术名称（'sale'，'im_chat'，...）或模块保留的主要路由（对于网站模块主要，
  'o_forum' 针对 *website_forum* 模块）。 这个规则的唯一例外是webclient：它只使用 *o_* 前缀。
- 避免使用id
- 使用Bootstrap本地类
- 使用下划线小写符号命名类

Git
===

提交信息
--------------

你的提交前缀

- **[IMP]** 改进
- **[FIX]** 修复错误
- **[REF]** 重构
- **[ADD]** 用于添加新资源
- **[REM]** 用于移除资源
- **[MOV]** 移动文件（不要改变移动文件的内容，否则Git会松动轨迹，历史会丢失！），或者简单地将代码从一个文件移动到另一个文件。
- **[MERGE]** 用于合并提交（仅用于前向/后端口）
- **[CLA]** 用于签署Odoo个人贡献者许可证

然后，在消息本身中，指定受更改影响的代码部分（模块名称，库，横向对象，...）和更改的描述。

- 始终包含有意义的提交消息：它应该是自解释（足够长），包括已更改的模块的名称和更改的原因。
  不要使用单词，如“bugfix”或“改善”。
- 避免同时影响多个模块的提交。 尝试分裂到不同的提交，其中受影响的模块是不同的（如果我们需要单独恢复模块将是有帮助的）。

.. code-block:: text

    [FIX] website, website_mail: remove unused alert div, fixes look of input-group-btn

    Bootstrap's CSS depends on the input-group-btn
    element being the first/last child of its parent.
    This was not the case because of the invisible
    and useless alert.

    [IMP] fields: reduce memory footprint of list/set field attributes

    [REF] web: add module system to the web client

    This commit introduces a new module system for the javascript code.
    Instead of using global ...


.. note:: 使用长描述来解释 *why* 而不是 *what*，*what* 可以在diff中看到
