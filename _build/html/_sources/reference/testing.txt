:banner: banners/testing_modules.jpg

.. _reference/testing:


===============
测试模块
===============

Odoo中使用unittest来支持模块的测试。

要编写测试，只需在模块中定义 ``tests`` 子模块，它将自动检查测试模块。测试模块应该是
以 ``test_`` 开头的名称，并且应该从 ``tests/__init__.py`` 文件中导入，例如：

.. code-block:: text

    your_module
    |-- ...
    `-- tests
        |-- __init__.py
        |-- test_bar.py
        `-- test_foo.py

``__init__.py`` 包含::

    from . import test_foo, test_bar

.. warning::

    不从``tests/__init__.py``导入的测试模块将不会运行

.. versionchanged:: 8.0

    以前，测试运行器只会运行添加到两个列表 ``fast_suite`` 和 ``checks`` 中的模块，并
    检入 ``tests/	__init__.py`` 文件。在8.0版本中它将运行所有导入的模块

测试运行器将简单地运行测试用例，如官方的 `unittest documentation`_ ，所述，但是Odoo提供了许多与测试Odoo内容（主要模块）相关的实用程序和帮助:

.. autoclass:: odoo.tests.common.TransactionCase
    :members: browse_ref, ref

.. autoclass:: odoo.tests.common.SingleTransactionCase
    :members: browse_ref, ref

默认情况下，在安装相应的模块后立即运行测试。测试用例也可以配置为在所有模块安装后运行，而不是在模块安装后立即执行

.. autofunction:: odoo.tests.common.at_install

.. autofunction:: odoo.tests.common.post_install

最常见的情况是使用 :class:`~odoo.tests.common.TransactionCase` 并在每个方法中测试模型的属性::

    class TestModelA(common.TransactionCase):
        def test_some_action(self):
            record = self.env['model.a'].create({'field': 'value'})
            record.some_action()
            self.assertEqual(
                record.field,
                expected_field_value)

        # other tests...

运行测试
-------------

在安装或更新模块时，将自动运行测试。启动Odoo服务器时启用 :option:`--test-enable <odoo-bin --test-enable>` 。

从Odoo 8开始，是不支持在安装/更新周期之外运行测试

.. _unittest documentation: https://docs.python.org/2/library/unittest.html
