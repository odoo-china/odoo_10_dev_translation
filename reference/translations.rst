:banner: banners/translate.jpg

.. _reference/translations:


===================
翻译模块
===================

导出可翻译术语
==============

由于您的模块中的许多术语都是“可隐式翻译”的，即使您没有对翻译进行任何具体工作，
也可以导出模块的可翻译术语，并找到要使用的内容。

.. todo:: needs technical features

翻译导出通过管理界面通过登录到后端接口并打开 :menuselection:`设置 ->翻译 ->导入/导出 ->导出翻译`

* 将语言保留为默认语言（新语言/空模板）
* 选择 `PO 文件`_ 格式
* 选择你的模块
* 点击 :guilabel:`导出` 并且下载文件

.. image:: translations/po-export.*
    :align: center
    :width: 75%

这给你一个文件叫做 :file:`{yourmodule}.pot` ，它应该被移动到 :file:`{yourmodule}/i18n/` 目录下。
该文件是一个 *PO模板* ，它只是列出可翻译字符串，从中可以创建实际的翻译（PO文件）。 可以使用 msginit_ ，
使用专用翻译工具（如 POEdit_）创建PO文件，或者通过将模板复制到名为 :file:`{language}.po` 的新文件中来创建。
翻译文件应放在 :file:`{yourmodule}/i18n/` ，旁边 :file:`{yourmodule}.pot` ，
在安装相应语言时由Odoo自动加载（通过 :menuselection:`设置 ->翻译 ->加载翻译` ）

.. note:: 在安装或更新模块时，也会安装或更新所有加载语言的翻译

隐性出口
================

Odoo自动从“data”类型的内容中导出可翻译字符串：

* 在非QWeb视图中，所有文本节点以及 ``string``，``help``，``sum``，``confirm`` 和 ``placeholder``
  属性的内容被导出
* QWeb模板（服务器端和客户端），除了 ``t-translation ="off"`` 块内部，
  ``title`` ，``alt`` ， ``label`` 和 ``placeholder`` 属性也被导出
* 对于 :class:`~odoo.fields.Field`, 除非该的模块被标记为
  ``_translate = False``:

  * 它的 ``string`` 和 ``help`` 属性被导出
  * 如果存在 ``selection`` 和一个列表（或元组），它就被导出
  * 如果它们的 ``translate`` 属性设置为 ``True`` ，它们的所有现有值（囊括所有记录）被导出

* 帮助/错误消息 :attr:`~odoo.models.Model._constraints` 和
  :attr:`~odoo.models.Model._sql_constraints` 被导出

明确出口
================

当涉及Python代码或Javascript代码中更多的“命令式”情况时，Odoo不能自动导出可翻译术语，
因此必须明确标记以便导出。 这通过在函数调用中包装文字字符串来完成。

在Python中，包装函数是 :func:`odoo._`::

    title = _("Bank Accounts")

在Python中，普遍的包装函数是 :js:func:`odoo.web._t`:

.. code-block:: javascript

    title = _t("Bank Accounts")

.. warning::

    只有文本字符串可以标记为导出，而不是表达式或变量。 对于字符串格式化的情况，
    这意味着必须标记格式字符串，而不是格式化的字符串::

        # 错误，提取可以工作，但它不会正确翻译文本
        _("Scheduled meeting with %s" % invitee.name)

        # 正确
        _("Scheduled meeting with %s") % invitee.name

.. _PO File: http://en.wikipedia.org/wiki/Gettext#Translating
.. _msginit: http://www.gnu.org/software/gettext/manual/gettext.html#Creating
.. _POEdit: http://poedit.net/
