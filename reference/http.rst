:banner: banners/web_controllers.jpg

===============
网络控制器
===============

.. _reference/http/routing:

路由(Routing)
=======

.. autofunction:: odoo.http.route

.. _reference/http/request:

请求(Request)
=======

请求对象在请求开始时被自动设置于 :data:`odoo.http.request`

.. autoclass:: odoo.http.WebRequest
    :members:
    :member-order: bysource
.. autoclass:: odoo.http.HttpRequest
    :members:
.. autoclass:: odoo.http.JsonRequest
    :members:

响应(Response)
========

.. autoclass:: odoo.http.Response
    :members:
    :member-order: bysource

    .. maybe set this to document all the fine methods on Werkzeug's Response
       object? (it works)
       :inherited-members:

.. _reference/http/controllers:

控制器(Controllers)
===========

控制器需要提供扩展性, 很像模型( :class:`~odoo.models.Model` ), 但是不使用相同的机制,
因为其先决条件 (一个载入模块的数据库) 可能并没有准备好 (比如没有创建数据库, 或者没有选择数据库).

因此控制器提供了他们自己的扩展机制, 与模型(model)的是分开的:

控制器是通过继承( :ref:`inheriting <python:tut-inheritance>` )

.. autoclass:: odoo.http.Controller

来创建的, 并且定义了方法, 这个方法被如下函数装饰 :func:`~odoo.http.route`::

    class MyController(odoo.http.Controller):
        @route('/some_url', auth='public')
        def handler(self):
            return stuff()

为了 *复写(override)* 一个控制器, 继承( :ref:`inherit <python:tut-inheritance>` )
它的类并且复写相关的方法, 如果需要的话, 使他们重新曝光::

    class Extension(MyController):
        @route()
        def handler(self):
            do_before()
            return super(Extension, self).handler()

* 用 :func:`~odoo.http.route` 装饰, 这对于保证这个方法(和路由)可见是必须的:
  如果这个方法重新定义时没有被装饰, 它就会成为"未发布的(unpublished)"
* 所有方法的装饰器是链接的, 如果复写方法的装饰器没有参数, 所有之前的参数会被保留,
  任何提供的参数会覆盖之前已经定义的参数, 例如::

    class Restrict(MyController):
        @route(auth='user')
        def handler(self):
            return super(Restrict, self).handler()

  会改变 ``/some_url`` 的认证, 从公共变成用户(需要登录)
