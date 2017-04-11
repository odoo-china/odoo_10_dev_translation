:orphan:

.. _reference/async:

异步操作
=======================

作为一门编程语言(和运行库), javascript在本质上是单线程的.
这意味着任何阻塞请求或者运算会阻塞整个页面(并且, 在老版本的浏览器中,
软件自身甚至会阻止用户切换至另一个标签页): 一个javascript环境可以被当成一个基于事件
的运行循环,在这个循环中应用的开发者无法控制循环本身.

因此, 不建议执行长时间运行的同步网络请求或者其他种类的复杂且开销巨大的访问,
取而代之的是异步API的应用.

这个指南的目标是提供一些处理异步系统的工具, 并且对系统问题和危害进行堤防.

延迟模块(Deferreds)
---------

延迟模块(Deferreds)是一种形式的约定 (`promises`_.) OpenERP Web 目前使用的是 `jQuery's deferred`_.

延迟模块(Deferreds)的核心思想是这样的: 潜在的异步方法会返回一个 :js:class:`Deferred` 对象
而不是一个任意值或者(更常见的)什么也不返回.

这个对象可通过增加回调(callbacks)从而被用于跟踪异步操作的结束,
不论是成功的回调(callbacks)还是错误的回调(callbacks).

延迟模块(Deferreds)相对于直接简单的传递回调函数给异步方法的一个巨大优势是提供了一种能力:组合( :ref:`compose them
<reference/async/composition>` ).

使用延迟模块(Deferreds)
~~~~~~~~~~~~~~~

延迟模块(Deferreds)最重要的方法是 :js:func:`Deferred.then`.
它被用于附加新的回调(callbacks)到延迟对象.

* 第一个参数附加了一个成功的回调, 在延迟对象被成功解析时调用并提供解析的值给异步操作.

* 第二个参数附加了一个失败的回调，在延迟对象被拒绝时调用并提供了拒绝的值(通常是一些错误消息).

附加至延迟模块(Deferreds)的回调(callbacks)绝对不会是"丢失(lost)":
如果一个回调被附加至一个已经解析或者拒绝的延迟, 这个回调会被立刻调用(或忽略).
一个延迟模块只会被解析或者拒绝一次, 其结果只会是已解析或已拒绝:
一个给定的延迟模块不可能调用一个单独的成功回调两次, 或者同时调用成功或者失败的回调(callbacks).

:js:func:`~Deferred.then` 会是你在与延迟对象(和异步API)交互时最常使用的方法.

构建延迟模块(Deferreds)
~~~~~~~~~~~~~~~~~~

在使用了异步API后, 紧接着就要构建它们: 用于虚拟(mocks_),
用一个复杂的方式从多种源组成延迟模块(Deferreds),
从而使当前的操作刷新当前屏幕或者给其他事件展现的时间, ...

这在使用jQuery的延迟对象时是很容易实现的.

.. 注意:: 这部分是jQuery的延迟对象的实现细节, 创建约定(promises)不是任何我知道的标准(或者假设)的一部分.
          如果你使用不属于jQuery的延迟对象, 他们的API可能(并且经常是)完全不同.

延迟模块(Deferreds)通过不使用任何参数调用它们的构造函数(constructor)创建出来 [#]_ .
这样就用如下的方法创建了一个 :js:class:`Deferred` 实例对象:

:js:func:`Deferred.resolve`

    就像他的名字指出的, 这个方法把延迟对象转换至"已解析"的状态.
    可以提供必要数目的参数给它, 这些参数会被提供给任何处于等待状态的成功回调.

:js:func:`Deferred.reject`

    类似于 :js:func:`~Deferred.resolve`, 但是它是把延迟对象转换至"已拒绝"的状态
    并且调用处于等待状态的失败处理函数.

:js:func:`Deferred.promise`

    创建一个延迟对象的只读视图. 一般来说, 返回一个延迟的约定视图来阻止调用器来解析或
    拒绝这个对你有用的延迟对象.

:js:func:`~Deferred.reject` 和 :js:func:`~Deferred.resolve` 被用于通知调用器
异步操作已经失效了 (或者成功了). 在异步操作已经结束了的时候应该简单的调用这些方法,
来通知任何对这个结果有兴趣的人.

.. _reference/async/composition:

组合延迟模块(Deferreds)
~~~~~~~~~~~~~~~~~~~

目前我们看到的东西都很好, 但大部分通过给其他函数来传递函数都是可以实现的
(即使增加函数实现起来可能比较繁琐... 但依然是可实现的).

延迟模块(Deferreds) 最大的亮点是当代码需要以某种方式来组合异步操作时,
因为他们可以被用于作为这样组合的一个基础.

延迟对象的组合方式主要有两种: 多路传输(multiplexing)和管线/级联(piping/cascading).

延迟对象的多路传输
`````````````````````

多路传输的最常见的原因就是简单的执行多个异步操作
并且想要他们都完成之后才继续执行后续操作(并且处理更多事情).

jQuery的约定(promises)的多路传输函数是 :js:func:`when`.

.. note:: jQuery的 :js:func:`when` 的多路传输的表现是一个
          (大部分情况下不相容的)定义于 `CommonJS Promises/B`_ 的扩展.

这个函数可以接收任何数目的约定 [#]_ 并且会返回一个约定.

这个返回的约定在 *所有的* 多路传输约定已经被解析完毕之后被解析,
并且会在任何一个多路传输的约定被拒绝之后被立即拒绝
(用约定对象来代替布尔值, 它的表现就像Python语言里的 ``all()`` 函数).

如果需要的话, 多个通过 :js:func:`when` 多路传输的约定的解析值被分配到 :js:func:`when`
的成功回调函数参数里. 在约定位于函数  :js:func:`when` 里时,
约定的解析值与回调的参数有相同的索引, 因此你会获得:

.. code-block:: javascript

    $.when(p0, p1, p2, p3).then(
            function (results0, results1, results2, results3) {
        // code
    });

.. 警告::

    在一个普通的分配中, 每个传递给回调的参数会是一个阵列(array):
    每个约定被使用一个有0..n值的阵列概念性的解析
    并且这些值被传递给 :js:func:`when` 的回调.
    但是jQuery看成延迟模块(Deferreds)在解析一个特殊的值, 并且把这个值 "打开" .

    例如, 在上面的代码块中, 如果每个约定的索引是它解析值的数目(0 to 3),
    ``results0`` 就是一个空阵列, ``results2`` 是一个2个(一对)元素的阵列,
    但 ``results1`` 是 ``p1`` 解析的真实的值, 而不是一个阵列.

延迟链(Deferred chaining)
`````````````````

第二个有用的组合是这样的, 把一个异步操作的结果作为另一异步操作的开始, 并且需要二者的结果:
使用目前已经描述过的工具, 就拿处理 OpenERP 的查找/读取队列来说,
需要如下的这样几行:

.. code-block:: javascript

    var result = $.Deferred();
    Model.search(condition).then(function (ids) {
        Model.read(ids, fields).then(function (records) {
            result.resolve(records);
        });
    });
    return result.promise();

这个代码片段看起来还是不错的, 不过马上它就会变得很笨重.

但是 :js:func:`~Deferred.then` 也会允许处理这类的链路:
它返回一个新的约定对象, 而不是它调用的那个, 并且返回回调(callbacks)的值,
这个行为来说很重要: 不论是调用了哪个回调函数.

* 如果没有设置回调(没有提供或者留空), 那解析或者拒绝值就会简单的递交给
  :js:func:`~Deferred.then` 的约定(它本质上是个空操作)

* 如果设置了回调并且不返回一个可观测的对象( 一个延迟或者一个约定),
  那么它的返回值 (如果不返回任何东西的话就为 ``未定义(undefined)`` ) 会代替给它的值.

  .. code-block:: javascript

      promise.then(function () {
          console.log('called');
      });

  会用这个单独值 ``未定义(undefined)`` 来解析.

* 如果设置了回调并返回了一个可观测的对象, 那么这个对象会成为这个管线的真实解析(和结果).
  这意味着一个来自失败回调的已经解析的约定会解析这个管线,
  并且一个来自成功回调的失败的约定会拒绝这个管线.

  这提供了一个简易的链路操作继承, 并且之前的代码片段可以被重写:

  .. code-block:: javascript

      return Model.search(condition).then(function (ids) {
          return Model.read(ids, fields);
      });

  如果 ``search`` 或者 ``read`` 失败, 整个表达的结果会编码失败(有正确的拒绝值),
  并且链路处理正确的话, 会被 ``read`` 的解析结果来解析.

在使用第三方的基于约定的API时, :js:func:`~Deferred.then` 同样是有用的,
用于过滤示例的解析值计数 (为了使用 :js:func:`when` 的对于单值约定的特殊的处理).

jQuery.Deferred API
~~~~~~~~~~~~~~~~~~~

.. js:function:: when(deferreds…)

    :param deferreds: 多路传输的延迟对象
    :returns: 被多路传输的延迟对象
    :rtype: :js:class:`Deferred`

.. js:class:: Deferred

    .. js:function:: Deferred.then(doneCallback[, failCallback])

        附加新的回调(callbacks)至解析或者拒绝的延迟对象.
        回调(callbacks) 以他们被附加至延迟的顺序被处理.

        为了只提供一个失败回调, 传递 ``空(null)`` 来作为 ``完成回调(doneCallback)``,
        为了只提供一个成功回调, 第二个参数可以被忽略(并且不传递).

        返回一个新的延迟, 其解析相应回调的结果, 如果回调返回一个延迟本身,
        那么新的回调会被用于链路的解析.

        :param doneCallback: 延迟解析完成调用的函数
        :param failCallback: 延迟拒绝之后调用的函数
        :returns: 调用它的延迟对象
        :rtype: :js:class:`Deferred`

    .. js:function:: Deferred.done(doneCallback)

        附加一个新的成功回调至延迟对象, ``deferred.then(doneCallback)`` 的快捷方式.

        .. 注意:: 一个不同在于 :js:func:`Deferred.done` 的结果是被忽略了而不是传递给了链路

        这是一个 `CommonJS Promises/A`_ 的jQuery扩展, 提供了很少的值来直接调用
        :js:func:`~Deferred.then` , 这应该被避免.

        :param doneCallback: 延迟被解析后调用的函数
        :type doneCallback: Function
        :returns: 调用它的延迟对象
        :rtype: :js:class:`Deferred`

    .. js:function:: Deferred.fail(failCallback)

        附加一个新的失败回调至延迟对象, ``deferred.then(null, failCallback)`` 的快捷方式.

        `Promises/A <CommonJS Promises/A>`_ 的第二个jQuery扩展.
        尽管它比 :js:func:`~Deferred.done` 提供了更多的值, 它仍然不够且应该被避免.

        :param failCallback: 延迟被拒绝时调用的函数
        :type failCallback: Function
        :returns: 调用它的延迟对象
        :rtype: :js:class:`Deferred`

    .. js:function:: Deferred.promise()

        返回一个延迟对象的只读视图, 移除了所有的设值方法(解析或者拒绝).

    .. js:function:: Deferred.resolve(value…)

        被调用来解析一个延迟, 任何提供的值会被传递给延迟对象的成功处理函数.

        解析一个已经被解析或者拒绝的延迟没有任何效果.

    .. js:function:: Deferred.reject(value…)

        被调用来拒绝(失败)一个延迟, 任何提供的值会被传递给延迟对象的失败处理函数.

        拒绝一个已经被解析或者拒绝的延迟没有任何效果.

.. [#] 或者只是调用 :js:class:`Deferred` 作为一个函数, 结果是一样的.

.. [#] 或者非约定(not-promises), `CommonJS Promises/B`_ 的 :js:func:`when`
       是一个可以一致的处理数值和约定的角色: :js:func:`when` 会直接传递约定,
       但是非约定的值和对象会被转换成一个已解析的约定 (用它们自己的值来解析它们).

       jQuery的 :js:func:`when` 保持这个特性, 使得延迟模块(Deferreds)
       更容易由"静态(static)" 值创建, 或者在约定被打包进 :js:func:`when` 时
       来允许防御性代码, 以防万一.

.. _promises: http://en.wikipedia.org/wiki/Promise_(programming)
.. _jQuery's deferred: http://api.jquery.com/category/deferred-object/
.. _CommonJS Promises/A: http://wiki.commonjs.org/wiki/Promises/A
.. _CommonJS Promises/B: http://wiki.commonjs.org/wiki/Promises/B
.. _mocks: http://en.wikipedia.org/wiki/Mock_object
