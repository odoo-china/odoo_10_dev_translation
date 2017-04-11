:banner: banners/security.jpg

.. _reference/security:

================
Odoo中的安全性
================

除了使用自定义的代码手动管理访问之外，Odoo还提供了两个主要的数据驱动机制来管理或限制对数据的访问。

这两种机制通过 *分组* 链接到特定的用户：用户属于任何数量的组，并且安全机制与组相关联，因此向用户应用安全机制。

.. _reference/security/acl:

访问控制
==============

由 ``ir.model.access`` 记录管理，定义对整个模型的访问。

每个访问控制都有一个授予权限的模型，授予的权限以及可选的组

访问控制是累加的，对于给定的模型，用户具有访问对其任何组授予的所有权限：如果用户在允许写入的组中，并且另一个允许删除，则他们可以写入和删除。

如果未指定组，则访问控制适用于所有用户，否则仅适用于给定组的成员。

可用的权限是创建(``perm_create``)，搜索和阅读(``perm_read``)，更新现有记录(``perm_write``)删除现有记录(``perm_unlink``)

.. _reference/security/rules:

记录规则
============

记录规则是记录允许操作（创建，读取，更新或删除）必须满足的条件。在应用访问控制之后，它将跟随记录应用。

记录规则具有：

* 适用的模型
* 它应用的权限(例如，如果设置了 ``perm_read`` ，只有在读取记录时 才会检查规则)
* 规则应用于用户组，如果未指定组，规则为 *global*
*  :ref:`domain <reference/orm/domains>` 用于检查给定的记录匹配规则（可访问）或不匹配（不可访问）。上下文中使用两个变量计算域: ``user`` 是当前用户的记录，``time`` 是 `time module`_

全局规则和组规则（限于特定组的规则与应用于所有用户的组）的使用方式大不相同:

* 全局规则是减法，他们 *必须全部* 匹配以获得可访问的记录
* 组规则是累加的，如果它们 *任意* 匹配（并且所有全局规则匹配），则记录是可访问的

这意味着第一个 *组规则* 限制访问，但任意 *组规则* 进一步扩展它，而 *全局规则* 只能限制访问（或没有效果）。

.. 警告:: 记录规则不适用于管理员用户，
    :class: aphorism

    尽管访问规则

.. _reference/security/fields:

字段访问
============

.. versionadded:: 7.0

ORM :class:`~odoo.fields.Field` 可以有一个 ``groups`` 属性提供组列表（以逗号分隔的字符串 :term:`external identifiers`)。

如果当前用户不在列出的组中，他将无权访问该字段

* 受限字段将从请求的视图中自动删除
* 限制字段从 :meth:`~odoo.models.Model.fields_get` 响应中删除
* 尝试（显示）读取或写入受限字段会导致访问错误

.. todo::

    field access groups apply to administrator in fields_get but not in
    read/write...

工作流转移规则
=========================

工作流转换可以限制到特定组。群组外的用户不能触发转换

.. _foo: http://google.com
.. _time module: https://docs.python.org/2/library/time.html
