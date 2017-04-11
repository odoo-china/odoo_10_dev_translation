:banner: banners/cmdline.jpg

.. _reference/cmdline:

===============================
命令行界面: odoo-bin
===============================

.. _reference/cmdline/server:

启动server
==================

.. program:: odoo-bin

.. option:: -d <数据库名>, --database <数据库名>

    当安装或更新模块是使用数据库。

.. option:: -i <模块>, --init <模块>

    在运行服务之前用逗号分隔一系列安装的模块
    (要求 :option:`-d`).

.. option:: -u <模块>, --update <模块>

    在运行服务之前逗号分隔一系列模块来更新
    (要求 :option:`-d` ).

.. option:: --addons-path <目录>

    逗号分隔存储模块的目录。扫描这些目录中的模块

.. option:: --workers <数目>

    如果 ``计数`` 不是0（默认），使多处理器和设置指定数量的HTTP角色（子流程处理HTTP和RPC请求）

    .. note:: 多处理器模式仅在基于Unix系统下可用

    一系列选项允许限制和循环角色:

    .. option:: --limit-request <限制数目>

        在回收和重启之前，一个角色要处理的请求数。

        默认是 8196.

    .. option:: --limit-memory-soft <限制内存>

        每个角色允许最大限度的虚拟内存。如果限制被超出，这个角色将被停止并且在当前请求的最后被再回收。

        默认是 640MB.

    .. option:: --limit-memory-hard <限制内存>

        强制限制虚拟内存，并且任意角色超出限制将被立刻停止而不必等待当前请求进程的结束处理。

        默认是 768MB.

    .. option:: --limit-time-cpu <限制秒数>

        防止该角色使用每个请求超过限制CPU的秒数。如果超出限额，角色将被停止。


        默认是 60.

    .. option:: --limit-time-real <限制时间>

        防止角色花费超过限制秒数的处理请求。如果超出限额，角色将被停止。

        不同于 :option:`--limit-time-cpu` ，这是一个"硬性时间"限制包括例如 SQL查询。

        默认是 120.

.. option:: --max-cron-threads <count>

    一系列角色专用于计划任务。默认是2。这些角色在多线程模式下是线程，在多进程模式下是进程。

    针对多进程模式，这被加入到HTTP角色进程中。

.. option:: -c <配置文件>, --config <配置文件>

    通过一个转换的配置文件

.. option:: -s, --save

    保存服务器配置到当前的配置文件中(默认是 :file:`{$HOME}/.odoorc` ，并且使用
    :option: `-c` 被覆盖)

.. option:: --proxy-mode

    enables the use of ``X-Forwarded-*`` headers through `Werkzeug's proxy
    support`_.

    .. warning:: proxy mode *must not* be enabled outside of a reverse proxy
                 scenario

.. option:: --test-enable

    安装模块后运行测试

.. option:: --dev <feature,feature,...,feature>

    * ``all``: 以下的所有功能特性被激活

    * ``xml``: 从xml文件直接读取qweb模板，而不是从数据库中。一旦一个模板在数据库中被改动，它将不会从
      xml文件中被读取到直至下次更新/初始化。

    * ``reload``: 当python文件被更新重启服务(可能无法检测取决于使用的文本编辑器)

    * ``qweb``: 当一个节点包含 ``t-debug='debugger'`` 时相关的qweb模板被中断。

    * ``(i)p(u)db``:当一个意外的错误出现而日志返回这个错误之前启动一个python的调试器。

.. _reference/cmdline/server/database:

database
--------

.. option:: -r <数据库用户>, --db_user <数据库用户>

    用户连接PostgreSQL的数据库用户。

.. option:: -w <数据库密码>, --db_password <数据库密码>

    如果使用 `密码验证` ，提供数据库密码。

.. option:: --db_host <主机地址>

    数据库服务器地址

    * ``localhost`` on Windows
    * UNIX socket otherwise

.. option:: --db_port <端口号>

    数据库监听端口号，默认是 5432

.. option:: --db-filter <筛选器>

    隐藏不匹配<筛选器> 的数据库。过滤器是一种 `正则表达式` ，补充:

    - ``%h`` 被请求的整个主机名替换
    - ``%d`` 被替换为请求中除www以外的子域名(因此域odoo.com和www.odoo.com都将匹配odoo数据库)

.. option:: --db-template <模板>

    当从数据库管理界面创建一个新的数据库，使用指定的 `template database` 。
    默认是 ``template1`` 。

built-in HTTP
-------------

.. option:: --no-xmlrpc

   不要启动HTTP或长轮询工作（可能仍启动cron工作）

    .. warning:: 如果设置了 :option:`--test-enable` ，则没有效果，
                 因为测试需要一个可访问的HTTP服务器

.. option:: --xmlrpc-interface <interface>

    HTTP服务器监听的TCP/IP地址，默认为 ``0.0.0.0`` （所有地址）

.. option:: --xmlrpc-port <port>

    HTTP服务器监听的端口，默认是8069

.. option:: --longpolling-port <port>

    用于多进程或gevent模式下长轮询连接的TCP端口，默认为8072.默认（线程）模式下不使用。

日志
-------

默认情况下，Odoo显示 level_ ``info`` 的所有了工作流日志记录（仅 ``warning`` ）日志记录，
并且日志输出发送到 ``stdout`` 。各种选项可用于将日志重定向到其他目的地并自定义日志输出量

.. option:: --logfile <file>

    将日志输出发送到指定的文件，而不是stdout。 在Unix上，
    文件 `可以由外部日志循环程序管理<https://docs.python.org/2/library/logging.handlers.html#watchedfilehandler>` ，并且在替换时将自动重新打开

.. option:: --logrotate

    启用 `日志循环 <https://docs.python.org/2/library/logging.handlers.html
    #timedrotatingfilehandler>`_ daily，保留30个备份。 日志循环频率和备份数量不可配置。

.. option:: --syslog

    日志到系统的事件记录器：`系统日志的Unix系统 <https://docs.python.org/2/library/logging.handlers.html#sysloghandler>` 和 `Windows上的事件日志<https://docs.python.org/2/library/logging.handlers.html#nteventloghandler>`_ 。

    两者都不可配置

.. option:: --log-db <dbname>

    日志到指定数据库的 ``ir.logging`` 模型（ ``ir_logging`` 表）。
    数据库可以是“当前”PostgreSQL中的数据库的名称，或者例如 `a PostgreSQL URI`_ 。 例如 日志聚合

.. option:: --log-handler <handler-spec>


    :samp:`{LOGGER}:{LEVEL}` ，在提供 ``LEVE`` 下启用 ``LOGGER`` 。
    ``odoo.models:DEBUG`` 将在模型中启用 ``DEBUG`` 级别的所有日志消息。

    * 冒号 ``:`` 是必须的
    * 可以省略记录器来配置根处理程序
    * 如果级别被省略，则设置为 ``INFO``

    可以重复该选项以配置多个记录器 例如

    .. code-block:: 控制台

        $ odoo-bin --log-handler :DEBUG --log-handler werkzeug:CRITICAL --log-handler odoo.fields:WARNING

.. option:: --log-request

    对RPC请求启用DEBUG日志记录，等效于 ``--log-handler=odoo.http.rpc.request:DEBUG``

.. option:: --log-response

    为RPC响应启用DEBUG日志记录，等效于 ``--log-handler=odoo.http.rpc.response:DEBUG``

.. option:: --log-web

    启用DEBUG日志记录HTTP请求和响应，等效于 `--log-handler=odoo.http:DEBUG``

.. option:: --log-sql

    启用SQL查询的DEBUG日志记录，等效于 ``--log-handler=odoo.sql_db:DEBUG``

.. option:: --log-level <level>

    在特定记录器上更容易设置预定义级别的快捷方式。 在 ``odoo`` 和 ``werkzeug`` 记录器上设置
    “真正的”级别（``critical``，``error``，``warn``，``debug``），
    （除了 ``debug``，它只在 ``odoo`` 设置）。

    Odoo还提供了适用于不同记录器集合的调试伪级别：

    ``debug_sql``
        将SQL记录器设置为 ``debug``

        等效于 ``--log-sql``
    ``debug_rpc``
        将 ``odoo`` 和HTTP请求记录器设置为 ``debug``

        等效于 ``--log-level debug --log-request``
    ``debug_rpc_answer``
        设置 ``odoo`` 和HTTP请求和响应记录器 ``debug``

        等效于 ``--log-level debug --log-request --log-response``

    .. note::

        如果 :option:`--log-level` 和 :option:`--log-handler` 发生冲突，后者被使用


.. _reference/cmdline/scaffold:

Scaffolding
===========

.. program:: odoo-bin scaffold

Scaffolding是自动创建的骨架结构，以简化（新模块，在Odoo的情况下）引导。 虽然不必要，但它避免了设置基本结构和查找所有起始要求的烦琐。

Scaffolding可通过 :command:`odoo-bin scaffold` 子命令。

.. option:: -t <template>

    一个模板目录，文件通过jinja2_传递，然后复制到 ``destination`` 目录

.. option:: name

    要创建的模块的名称，可以以各种方式来生成程序化名称（例如模块目录名称，型号名称，_）

.. option:: destination

    在哪个目录中创建新模块，默认为当前目录

.. _reference/cmdline/config:

配置文件
==================

大多数命令行选项也可以通过配置文件指定。 大多数时候，他们使用类似的名称，
前缀 ``-`` 被删除，其他 ``-`` 被替换为 ``_`` 。 :option:`--db-template` 成为 ``db_template`` 。

某些转化与模式不匹配：

* :option:`--db-filter` 变成 ``dbfilter``
* :option:`--no-xmlrpc` 对应于 ``xmlrpc`` 布尔值
* 记录预设（所有选项以 ``--log-`` 开头除了 :option: `--log-handler` 和 :option: `--log-db` ）
  只是直接在配置文件中添加内容到 ``log_handler``
* :option:`--smtp` 被存储为 ``smtp_server``
* :option:`--database` 被存储为 ``db_name``
* :option:`--debug` 被存储为 ``debug_mode`` (布尔值)
* :option:`--i18n-import` 和 :option:`--i18n-export` 根本不可用于配置文件

默认配置文件是 :file:`{$ HOME} /.odoorc` ，可以使用 :option:`--config <odoo-bin -c>` 重写。
指定 :option:`--save <odoo-bin -s>` 会将当前配置状态保存回该文件。

.. _jinja2: http://jinja.pocoo.org
.. _正则表达式: https://docs.python.org/2/library/re.html
.. _密码认证:
    http://www.postgresql.org/docs/9.3/static/auth-methods.html#AUTH-PASSWORD
.. _数据库模板:
    http://www.postgresql.org/docs/9.3/static/manage-ag-templatedbs.html
.. _级别:
    https://docs.python.org/2/library/logging.html#logging.Logger.setLevel
.. _a PostgreSQL URI:
    http://www.postgresql.org/docs/9.2/static/libpq-connect.html#AEN38208
.. _Werkzeug 代理支持:
    http://werkzeug.pocoo.org/docs/contrib/fixers/#werkzeug.contrib.fixers.ProxyFix
.. _pyinotify: https://github.com/seb-m/pyinotify/wiki
