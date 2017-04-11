:banner: banners/upgrade_api.jpg
:types: api


:code-column:

.. _reference/upgrade-api:

===========
升级接口(Upgrade API)
===========

简介
~~~~~~~~~~~~

这份文档描述了升级Odoo的数据库至一个更新的版本的接口(API).

它允许升级一个数据库而不必依赖位于
https://upgrade.odoo.com
html的表格, 虽然数据库还是会遵从那个表格描述的相同的流程.


需要的步骤如下:

* :ref:`创建一个请求 <upgrade-api-create-method>`
* :ref:`上传数据库快照(dump) <upgrade-api-upload-method>`
* :ref:`运行升级流程 <upgrade-api-process-method>`
* :ref:`获得数据库请求的状态 <upgrade-api-status-method>`
* :ref:`下载升级的数据库快照(dump) <upgrade-api-download-method>`

方法(method)
~~~~~~~~~~~

.. _upgrade-api-create-method:

创建一个数据库升级请求
===================================

这个动作创建一个需要如下信息的数据库请求:

* 你的合同参考
* 你的邮件地址
* 目标版本 (你想升级至的 Odoo 版本)
* 你的请求的目的 (测试或生产环境)
* 数据库快照的名字 (必需但只是补充性的)
* 可选的服务器时区 (针对 Odoo 源版本 < 6.1)

介绍 ``创建(create)`` 的方法
---------------------

.. py:function:: https://upgrade.odoo.com/database/v1/create

    创建一个数据库升级请求

    :param str contract: (必需) 企业的合同号码
    :param str email: (必需) 邮件地址
    :param str target: (必需) 你想要升级至的Odoo的版本. 有效的选项是: 6.0, 6.1, 7.0, 8.0
    :param str aim: (必需) 你升级数据库请求的目的. 有效的选项是: 测试, 生产.
    :param str filename: (必需) 一个你的数据库快照文件的纯粹的补充性的名字
    :param str timezone: (可选) 你服务器使用的时区. 只针对Odoo源版本 < 6.1
    :return: 请求的结果
    :rtype: JSON 字典

这个 *创建(create)* 的方法返回一个如下键值的JSON字典:

.. _upgrade-api-json-failure:

``失败(failures)``
''''''''''''

包含错误的一个列表.

一个字典的列表, 每个字典给出一个特定错误的信息.
每个字典依据不同的错误类型可以包含不同的键值
但是你总是会获得 ``原因(reason)`` 和 ``消息(message)`` 两个键:

* ``reason``: 错误的类型
* ``message``: 易于阅读的信息

一些可能的键值:

* ``code``: 一个错误值
* ``value``: 一个错误值
* ``expected``: 一个包含有效值的列表

查看一侧的示例.

.. rst-class:: setup doc-aside

.. switcher::

    .. code-block:: json

            {
              "failures": [
                {
                  "expected": [
                    "6.0",
                    "6.1",
                    "7.0",
                    "8.0",
                  ],
                  "message": "Invalid value \"5.0\"",
                  "reason": "TARGET:INVALID",
                  "value": "5.0"
                },
                {
                  "code": "M123456-abcxyz",
                  "message": "Can not find contract M123456-abcxyz",
                  "reason": "CONTRACT:NOT_FOUND"
                }
              ]
            }


``请求(request)``
'''''''''''

如果 *创建(create)* 方法成功了, *请求(request)* 的键对应的值将会是一个
包含创建请求的各种信息的字典:

最重要的键值是:

* ``id``: 请求的id
* ``key``: 你的这次请求的私人键值

这两个值会被其他方法使用(上传, 处理和状态)

其他的键值会在介绍 :ref:`状态方法 <upgrade-api-status-method>` 的时候进行介绍.


示例脚本(Sample script)
'''''''''''''

这里有建立数据库升级请求的两个示例, 使用如下的方法:

* 一个使用的是python编程语言里的pycurl库
* 一个使用的是bash编程语言里的 `curl <http://curl.haxx.se>`_ (使用http传输数据的工具)
  和 `jq <https://stedolan.github.io/jq>`_ (JSON 处理程序):

.. rst-class:: setup doc-aside

.. switcher::

    .. code-block:: python

        from urllib import urlencode
        from io import BytesIO
        import pycurl
        import json

        CREATE_URL = "https://upgrade.odoo.com/database/v1/create"
        CONTRACT = "M123456-abcdef"
        AIM = "test"
        TARGET = "8.0"
        EMAIL = "john.doe@example.com"
        FILENAME = "db_name.dump"

        fields = dict([
            ('aim', AIM),
            ('email', EMAIL),
            ('filename', DB_SOURCE),
            ('contract', CONTRACT),
            ('target', TARGET),
        ])
        postfields = urlencode(fields)

        c = pycurl.Curl()
        c.setopt(pycurl.URL, CREATE_URL)
        c.setopt(c.POSTFIELDS, postfields)
        data = BytesIO()
        c.setopt(c.WRITEFUNCTION, data.write)
        c.perform()

        # transform output into a dict:
        response = json.loads(data.getvalue())

        # get http status:
        http_code = c.getinfo(pycurl.HTTP_CODE)
        c.close()

    .. code-block:: bash

        CONTRACT=M123456-abcdef
        AIM=test
        TARGET=8.0
        EMAIL=john.doe@example.com
        FILENAME=db_name.dump
        CREATE_URL="https://upgrade.odoo.com/database/v1/create"
        URL_PARAMS="contract=${CONTRACT}&aim=${AIM}&target=${TARGET}&email=${EMAIL}&filename=${FILENAME}"
        curl -sS "${CREATE_URL}?${URL_PARAMS}" > create_result.json

        # check for failures
        failures=$(cat create_result.json | jq -r '.failures[]')
        if [ "$failures" != "" ]; then
          echo $failures | jq -r '.'
          exit 1
        fi

.. _upgrade-api-upload-method:

上传你的数据库快照(dump)
============================

有两个方法来上传你的数据库快照:

*  ``upload`` 方法使用的是HTTPS协议
*  ``request_sftp_access`` 方法使用的是SFTP协议

``upload`` 方法
---------------------

对于上传你的数据库快照来说, 这是最简单和最直接的方法, 它使用 HTTPS 协议.

.. py:function:: https://upgrade.odoo.com/database/v1/upload

    上传一个数据库快照

    :param str key: (必需)你的私人键值
    :param str request: (必需)你的请求id
    :return: 请求的结果
    :rtype: JSON字典

请求的id和私人的键值是通过 :ref:`create method
<upgrade-api-create-method>` 方法获得的

结果是一个JSON字典, 它包含了一个 ``失败(failures)`` 的列表, 如果一切顺利的话它应该为空.

.. rst-class:: setup doc-aside

.. switcher::

    .. code-block:: python

        import os
        import pycurl
        from urllib import urlencode

        UPLOAD_URL = "https://upgrade.odoo.com/database/v1/upload"
        DUMPFILE = "openchs.70.cdump"

        fields = dict([
            ('request', '10534'),
            ('key', 'Aw7pItGVKFuZ_FOR3U8VFQ=='),
        ])
        headers = {"Content-Type": "application/octet-stream"}
        postfields = urlencode(fields)

        c = pycurl.Curl()
        c.setopt(pycurl.URL, UPLOAD_URL+"?"+postfields)
        c.setopt(pycurl.POST, 1)
        filesize = os.path.getsize(DUMPFILE)
        c.setopt(pycurl.POSTFIELDSIZE, filesize)
        fp = open(DUMPFILE, 'rb')
        c.setopt(pycurl.READFUNCTION, fp.read)
        c.setopt(
            pycurl.HTTPHEADER,
            ['%s: %s' % (k, headers[k]) for k in headers])

        c.perform()
        c.close()

    .. code-block:: bash

        UPLOAD_URL="https://upgrade.odoo.com/database/v1/upload"
        DUMPFILE="openchs.70.cdump"
        KEY="Aw7pItGVKFuZ_FOR3U8VFQ=="
        REQUEST_ID="10534"
        URL_PARAMS="key=${KEY}&request=${REQUEST_ID}"
        HEADER="Content-Type: application/octet-stream"
        curl -H $HEADER --data-binary "@${DUMPFILE}" "${UPLOAD_URL}?${URL_PARAMS}"

.. _upgrade-api-request-sftp-access-method:

``request_sftp_access`` 方法
----------------------------------

这个方法推荐用于大的数据库快照.
它使用SFTP协议并支持断点续传.

它会新建一个临时的SFTP服务器, 通过它你可以链接并且允许你上传你的数据库快照来使用SFTP客户端.

.. py:function:: https://upgrade.odoo.com/database/v1/request_sftp_access

    新建一个SFTP服务器

    :param str key: (必需)你的私人键值
    :param str request: (必需)你的请求id
    :param str ssh_keys: (必需)指向你要使用的ssh公共秘钥的文件路径
    :return: 请求的结果
    :rtype: JSON字典

请求的id和私人的键值是通过 :ref:`create method
<upgrade-api-create-method>` 方法获得的

列出你的ssh公共秘钥的文件应该类似于一个标准的 ``authorized_keys`` 文件.
这个文件应该只包含公共秘钥, 空白行或注释(以 ``#`` 字符开头的行)

你的数据库升级请求应该处于 ``初步(draft)`` 状态.

``request_sftp_access`` 方法返回一个JSON字典,它包含如下的键值:


.. rst-class:: setup doc-aside

.. switcher::

    .. code-block:: python

        import os
        import pycurl
        from urllib import urlencode

        UPLOAD_URL = "https://upgrade.odoo.com/database/v1/request_sftp_access"
        SSH_KEYS="/path/to/your/authorized_keys"

        fields = dict([
            ('request', '10534'),
            ('key', 'Aw7pItGVKFuZ_FOR3U8VFQ=='),
        ])
        postfields = urlencode(fields)

        c = pycurl.Curl()
        c.setopt(pycurl.URL, UPLOAD_URL+"?"+postfields)
        c.setopt(pycurl.POST, 1)
        c.setopt(c.HTTPPOST,[("ssh_keys",
                                (c.FORM_FILE, SSH_KEYS,
                                c.FORM_CONTENTTYPE, "text/plain"))
                            ])

        c.perform()
        c.close()

    .. code-block:: bash

        REQUEST_SFTP_ACCESS_URL="https://upgrade.odoo.com/database/v1/request_sftp_access"
        SSH_KEYS=/path/to/your/authorized_keys
        KEY="Aw7pItGVKFuZ_FOR3U8VFQ=="
        REQUEST_ID="10534"
        URL_PARAMS="key=${KEY}&request=${REQUEST_ID}"

        curl -sS "${REQUEST_SFTP_ACCESS_URL}?${URL_PARAMS}" -F ssh_keys=@${SSH_KEYS} > request_sftp_result.json

        # check for failures
        failures=$(cat request_sftp_result.json | jq -r '.failures[]')
        if [ "$failures" != "" ]; then
          echo $failures | jq -r '.'
          exit 1
        fi


``失败(failures)``
''''''''''''

一个包含错误的列表. 查看 :ref:`failures <upgrade-api-json-failure>` ,
里面包含了对在失败情况下返回的JSON字典的解释.

``请求(request)``
'''''''''''

如果调用成功了,关联到 *请求(request)* 键的值会是一个包含你的SFTP链接参数的字典:

* ``hostname``: 链接的主机地址
* ``sftp_port``: 链接的端口
* ``sftp_user``: 用于链接SFTP的用户
* ``shared_file``: 你使用的文件名 (与当你在创建 :ref:`create method <upgrade-api-create-method>` 请求时使用的 ``filename`` 值是一样的)
* ``request_id``: 相关的升级请求id (只是补充信息, 对于链接来说非必须)
* ``sample_command``: 一个使用 'sftp' 客户端的示例命令

在正常情况下,你应该可以使用示例命令链接.

你只能访问 ``shared_file``. 其他文件你都不能访问,
并且你不能在你的SFTP服务器上的共享环境中新建一个新的文件.

使用 'sftp' 客户端
+++++++++++++++++++++++

一旦你适应你的SFTP客户端成功的链接, 你可以上传你的数据库快照.
下面是使用 'sftp' 客户端的一个示例对话:

::

    $ sftp -P 2200 user_10534@upgrade.odoo.com
    Connected to upgrade.odoo.com.
    sftp> put /path/to/openchs.70.cdump openchs.70.cdump
    Uploading /path/to/openchs.70.cdump to /openchs.70.cdump
    sftp> ls -l openchs.70.cdump
    -rw-rw-rw-    0 0        0          849920 Aug 30 15:58 openchs.70.cdump

如果你的连接中断了, 你可以继续传输你的文件,只需要在命令行里面加入 ``-a`` :

.. code-block:: text

    sftp> put -a /path/to/openchs.70.cdump openchs.70.cdump
    Resuming upload of /path/to/openchs.70.cdump to /openchs.70.cdump

如果你不想手动输入这个命令并且需要使用脚本自动化你的数据库升级, 你可以使用一个batch文件或者把你的命令pipe至'sftp':

::

  echo "put /path/to/openchs.70.cdump openchs.70.cdump" | sftp -b - -P 2200 user_10534@upgrade.odoo.com

参数 ``-b`` 需要一个文件名. 如果文件名是 ``-``, 他就从标准输入来读取命令.


.. _upgrade-api-process-method:

请求处理你的请求
==============================

请求升级平台来处理你的数据库快照的动作.

``处理(process)`` 方法
----------------------

.. py:function:: https://upgrade.odoo.com/database/v1/process

    处理一个数据库快照

    :param str key: (必需)你的私人键值
    :param str request: (必需)你的请求id
    :return: 请求的结果
    :rtype: JSON字典

请求的id和私人的键值是通过 :ref:`create method
<upgrade-api-create-method>` 方法获得的

结果是一个JSON字典, 它包含了一个 ``失败(failures)`` 的列表, 如果一切顺利的话它应该为空.

.. rst-class:: setup doc-aside

.. switcher::

    .. code-block:: python

        from urllib import urlencode
        from io import BytesIO
        import pycurl
        import json

        PROCESS_URL = "https://upgrade.odoo.com/database/v1/process"

        fields = dict([
            ('request', '10534'),
            ('key', 'Aw7pItGVKFuZ_FOR3U8VFQ=='),
        ])
        postfields = urlencode(fields)

        c = pycurl.Curl()
        c.setopt(pycurl.URL, PROCESS_URL)
        c.setopt(c.POSTFIELDS, postfields)
        data = BytesIO()
        c.setopt(c.WRITEFUNCTION, data.write)
        c.perform()

        # transform output into a dict:
        response = json.loads(data.getvalue())

        # get http status:
        http_code = c.getinfo(pycurl.HTTP_CODE)
        c.close()

    .. code-block:: bash

        PROCESS_URL="https://upgrade.odoo.com/database/v1/process"
        KEY="Aw7pItGVKFuZ_FOR3U8VFQ=="
        REQUEST_ID="10534"
        URL_PARAMS="key=${KEY}&request=${REQUEST_ID}"
        curl -sS "${PROCESS_URL}?${URL_PARAMS}"

.. _upgrade-api-status-method:

获得你的请求状态
=============================

这个动作查询你数据库升级请求的状态.

``status`` 方法
---------------------

.. py:function:: https://upgrade.odoo.com/database/v1/status

    查询一个数据库升级请求的状态

    :param str key: (必需)你的私人键值
    :param str request: (必需)你的请求id
    :return: 请求的结果
    :rtype: JSON字典

请求的id和私人的键值是通过 :ref:`create method
<upgrade-api-create-method>` 方法获得的

结果是一个包含你的数据库升级请求的各种信息的JSON字典.

.. rst-class:: setup doc-aside

.. switcher::

    .. code-block:: python

        from urllib import urlencode
        from io import BytesIO
        import pycurl
        import json

        STATUS_URL = "https://upgrade.odoo.com/database/v1/status"

        fields = dict([
            ('request', '10534'),
            ('key', 'Aw7pItGVKFuZ_FOR3U8VFQ=='),
        ])
        postfields = urlencode(fields)

        c = pycurl.Curl()
        c.setopt(pycurl.URL, PROCESS_URL)
        c.setopt(c.POSTFIELDS, postfields)
        data = BytesIO()
        c.setopt(c.WRITEFUNCTION, data.write)
        c.perform()

        # transform output into a dict:
        response = json.loads(data.getvalue())

        c.close()

    .. code-block:: bash

        STATUS_URL="https://upgrade.odoo.com/database/v1/status"
        KEY="Aw7pItGVKFuZ_FOR3U8VFQ=="
        REQUEST_ID="10534"
        URL_PARAMS="key=${KEY}&request=${REQUEST_ID}"
        curl -sS "${STATUS_URL}?${URL_PARAMS}"

输出示例
-------------

``请求(request)`` 键包含很多关于你请求的有用的信息:

``id``
    请求id
``key``
    你的私人键值
``email``
    当创建请求的时候你输入的邮件地址
``target``
    你创建请求时输入的Odoo的目标版本
``aim``
    在创建请求时输入的你数据库升级请求的目的(测试, 生产)
``filename``
    创建请求时提供的文件名
``timezone``
    创建请求时提供的时区
``state``
    请求的状态
``issue_stage``
    在Odoo主服务器上问题的阶段
``issue``
    在Odoo主服务器上的问题id
``status_url``
    访问你的数据库升级请求的html页面的URL
``notes_url``
    获得你的数据库升级请求的注释的URL
``original_sql_url``
    用于把你上传的数据库(未升级)作为一个SQL流而取得的URL
``original_dump_url``
    用于把你上传的数据库(未升级)作为一个压缩文件而取得的URL
``upgraded_sql_url``
    用于把你上传的数据库作为一个SQL流而取得的URL
``upgraded_dump_url``
    用于把你上传的数据库作为一个压缩文件而取得的URL
``modules_url``
    获取你的自定义模块的URL
``filesize``
    你上传的数据库文件的大小
``database_uuid``
    你的数据库的唯一ID
``created_at``
    在你创建请求时的日期
``estimated_time``
    升级你的数据库的估计时间
``processed_at``
    你的数据库升级开始的时间
``elapsed``
    升级你的数据库已经消耗的时间
``filestore``
    你的附件转换为了存储文件
``customer_message``
    关于你的请求的一个重要信息
``database_version``
    你上传的数据库(未升级)的一个猜测版本
``postgresql``
    你上传的数据库(未升级)的一个猜测的Postgresql版本
``compressions``
    你上传的数据库的压缩方法

.. rst-class:: setup doc-aside

.. switcher::

    .. code-block:: json

        {
          "failures": [],
          "request": {
            "id": 10534,
            "key": "Aw7pItGVKFuZ_FOR3U8VFQ==",
            "email": "john.doe@example.com",
            "target": "8.0",
            "aim": "test",
            "filename": "db_name.dump",
            "timezone": null,
            "state": "draft",
            "issue_stage": "new",
            "issue": 648398,
            "status_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/status",
            "notes_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/upgraded/notes",
            "original_sql_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/original/sql",
            "original_dump_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/original/archive",
            "upgraded_sql_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/upgraded/sql",
            "upgraded_dump_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/upgraded/archive",
            "modules_url": "https://upgrade.odoo.com/database/eu1/10534/Aw7pItGVKFuZ_FOR3U8VFQ==/modules/archive",
            "filesize": "912.99 Kb",
            "database_uuid": null,
            "created_at": "2015-09-09 07:13:49",
            "estimated_time": null,
            "processed_at": null,
            "elapsed": "00:00",
            "filestore": false,
            "customer_message": null,
            "database_version": null,
            "postgresql": "9.4",
            "compressions": [
              "pgdmp_custom",
              "sql"
            ]
          }
        }

.. _upgrade-api-download-method:

下载你的数据库快照
==============================

除了使用 :ref:`status method <upgrade-api-status-method>` 提供的URL来下载你已经迁移的数据库以外,
你也可以使用 :ref:`request_sftp_access method <upgrade-api-request-sftp-access-method>` 描述的SFTP协议

区别是你只能下载迁移的数据库. 不能上传.

你的数据库升级请求应该处于 ``完成(done)`` 状态.

一旦你成功的使用你的SFTP客户端连接了, 你就可以下载你的数据库快照.
下面有一个使用'sftp'客户端的示例对话:

::

    $ sftp -P 2200 user_10534@upgrade.odoo.com
    Connected to upgrade.odoo.com.
    sftp> get upgraded_openchs.70.cdump /path/to/upgraded_openchs.70.cdump
    Downloading /upgraded_openchs.70.cdump to /path/to/upgraded_openchs.70.cdump

