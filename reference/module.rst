:banner: banners/module.jpg

=======
模块
=======



.. _reference/module/manifest:

Manifest
========


manifest文件用于将一个python包声明为一个Odoo模块并且制定模块的元数据。

这个文件被命名为 ``__manifest__.py`` ，包含单个Python字典，其中每个键指定模块元数据。

::

    {
        'name': "A Module",
        'version': '1.0',
        'depends': ['base'],
        'author': "Author Name",
        'category': 'Category',
        'description': """
        Description text
        """,
        # 在安装时总是加载的数据文件
        'data': [
            'mymodule_view.xml',
        ],
        # 包含可选加载演示数据的数据文件
        'demo': [
            'demo_data.xml',
        ],
    }

可用的 manifest 字段:

``name`` ( ``str``, 必填)
    人类可阅读的模块的名字
``version`` ( ``str``)
    这个模块的版本，应该遵循 `semantic versioning`_ 规则
``description`` ( ``str``)
    使用reStructuredText形式描述该模块
``author`` ( ``str`` )
    模块作者名
``website`` ( ``str``)
    模块作者的网站地址
``license`` ( ``str``, defaults: ``LGPL-3``)
    模块的分发许可证
``category`` ( ``str``, default: ``Uncategorized``)
    Odoo中的分类类别，模块的大致业务领域。

    尽管推荐使用 `existing categories`_ , 该字段是自由形式的，而未知类别是即时创建的。
    类别层次结构可以使用分隔符 ``/`` 创建。 “Foo/Bar”将创建一个类别“Foo”，
    一个类别“Bar”作为“Foo”的子类别，并将“Bar”作为模块的类别。
``depends`` ( ``list(str)``)
    Odoo模块，必须在此模块之前加载，因为这个模块使用它们创建的特性，或者因为它改变了它们定义的内容。

    当安装一个模块时，它的所有依赖关系都在它之前安装。同样，在加载模块之前加载依赖关系。
``data`` ( ``list(str)``)
    数据文件列表始终被安装或更新模块时。 从模块根目录列出的路径列表
``demo`` ( ``list(str)``)
    仅在 *演示模式* 下安装或更新的数据文件列表
``auto_install`` (``bool``, default: ``False``)
    如果是 ``True`` ，如果所有的依赖都安装了，这个模块会自动安装。

    它通常用于实现两个其他独立模块之间的协同集成的“链接模块”。

    例如 ``sale_crm`` 依赖 ``sale`` 和 ``crm`` 并且被设置为 ``auto_install`` 。
    当 ``sale`` 和 ``crm`` 都安装后，它会自动向销售订单添加CRM活动跟踪，
    而 ``sale`` 或 ``crm`` 彼此意识不到。

.. _semantic versioning: http://semver.org
.. _existing categories:
     https://github.com/odoo/odoo/blob/master/odoo/addons/base/module/module_data.xml
