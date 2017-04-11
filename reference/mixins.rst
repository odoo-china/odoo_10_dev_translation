:banner: banners/mixins.jpg

.. _reference/mixins:

=========================
混入类(Mixins)和有用的类们
=========================

Odoo实现了一些有用的类和混入类,它们使你在你的对象上更容易地添加常用的行为.
这个指南会用示例和使用场景来详细描述他们.

.. _reference/mixins/mail:

消息特性(Messaging features)
==================

.. _reference/mixins/mail/chatter:

消息集成(Messaging integration)
---------------------

基础的消息系统
''''''''''''''''''''''

把消息特性集成进你的模型非常简单. 只需要继承(inheriting) ``mail.thread`` 模型
并且添加消息字段(和它们合适的控件)至你的表格视图,就可以完全工作了.

.. admonition:: 示例

    我们创建一个最简单的模型来代表一个商务旅行. 因为组织这类的旅行一般牵扯到很多人和很多讨论,
    让我们在这个模型上添加消息交流.

    .. code-block:: python

        class BusinessTrip(models.Model):
            _name = 'business.trip'
            _inherit = ['mail.thread']
            _description = 'Business Trip'

            name = fields.Char()
            partner_id = fields.Many2one('res.partner', 'Responsible')
            guest_ids = fields.Many2many('res.partner', 'Participants')

    在表格视图中:

    .. code-block:: xml

        <record id="businness_trip_form" model="ir.ui.view">
            <field name="name">business.trip.form</field>
            <field name="model">business.trip</field>
            <field name="arch" type="xml">
                <form string="Business Trip">
                    <!-- 你常规的表格视图在这里
                    ...
                    接下来是会话集成 -->
                    <div class="oe_chatter">
                        <field name="message_follower_ids" widget="mail_followers"/>
                        <field name="message_ids" widget="mail_thread"/>
                    </div>
                </form>
            </field>
        </record>

一旦你在模型上添加了会话支持, 用户就可以很容易的在你模型上的任意记录里添加信息或者内部通知;
这里的每个信息会发送一个提醒(给所有信息的接收者, 对于内部通知来说,给员工(*base.group_user*)
用户). 如果你的邮件网关和收集器地址正确的配置了的话, 这些通知会被作为邮件发送并且可以从邮件客户端直接回复;
自动路由系统会把回复路由至正确的进程.

服务器端, 一些帮助类功能可以使你很容易的发送信息来管理你记录的接收者:

.. rubric:: 发布消息

.. method:: message_post(self, body='', subject=None, message_type='notification', subtype=None, parent_id=False, attachments=None, content_subtype='html', **kwargs)

    在一个存在的进程中发布一个新的消息, 返回新的 mail.message ID.

    :param str body: 消息的主体, 通常是无害的原始的HTML
    :param str message_type: 参考mail_message.type 字段
    :param str content_subtype: 如果是纯文本: 转换主体为html
    :param int parent_id: 在私密讨论的情况下, 通过添加父partner进消息来处理前一个消息的回复
    :param list(tuple(str,str)) attachments: 附件的元组列表,形式是
        ``(name,content)``, 其内容不是base64编码的
    :param `\**kwargs`: 额外的关键字参数会被用于新的mail.message记录的默认的列值
    :return: 新建的mail.message的ID
    :rtype: int

.. method:: message_post_with_view(views_or_xmlid, **kwargs):

    使用ir.qweb引擎的一个使用view_id来发送邮件/发布一个消息的助手方法.
    这个方法是独立的,因为在允许批处理视图的模板和组合器中什么也没有.
    这个方法可能在模板处理ir.ui.views时消失.

    :param str or ``ir.ui.view`` record: 外部id或者需要发送的记录的视图

.. method:: message_post_with_template(template_id, **kwargs)

    使用模板来发送邮件的助手方法

    :param template_id: 需要渲染来生成消息主体的模板的id
    :param `\**kwargs`: 生成一个mail.compose.message向导的参数(其继承自mail.message)

.. rubric:: 接收消息

这些方法在一个新的邮件被邮件网关处理的时候调用. 这些邮件可能是新的进程
(如果他们通过 :ref:`别名(alias) <reference/mixins/mail/alias>` 到达)
或者只是一个已经存在的进程的回复. 复写他们使你可以设置进程记录的值,其依赖来自邮件本身的一些值
(比如更新一个数据或者一个邮件地址, 添加抄送地址作为接收者, 等等.).

.. method:: message_new(msg_dict, custom_values=None)

    当一个新的消息在一个给定的线程模型上接收到的时候,被 ``message_process`` 调用,
    如果这个消息不属于一个存在的线程中.

    默认的行为是创建一个相应模型的新的记录(基于从消息释放的一些很基本的信息).
    可以通过复写这个方法来实现额外的行为.

    :param dict msg_dict: 一个包含邮件细节和附件的映射(map). 查看 ``message_process``
        和 ``mail.message.parse`` 来获得详细信息.
    :param dict custom_values: 在创建新的线程记录时, 传递给create()的附加字段值的一个可选字典;
        请当心使用,这些值可能会覆盖来自这个消息的其他值.
    :rtype: int
    :return: 新创建的线程对象的id

.. method:: message_update(msg_dict, update_vals=None)

    当一个新的消息被一个现存的进程接收到时被 ``message_process`` 调用.
    默认的行为是用从接收到的邮件的 ``update_vals`` 来更新这个记录.

    可以通过复写这个方法来实现额外的行为.

    :param dict msg_dict: 一个包含邮件细节和附件的映射(map). 查看 ``message_process``
        和 ``mail.message.parse`` 来获得详细信息.
    :param dict update_vals: 一个包含更新给定id的记录的字典,如果字典是None或者空的,不会执行写操作.
    :return: True

.. rubric:: 接收者的管理

.. method:: message_subscribe(partner_ids=None, channel_ids=None, subtype_ids=None, force=True)

    把partner添加进接收者的记录里.

    :param list(int) partner_ids: 将要提交至记录的partner ID
    :param list(int) channel_ids: 将要提交至记录的channel ID
    :param list(int) subtype_ids: 将要提交至记录的channels/partner的子类型的ID(如果是 ``None`` 则是默认的子类)
    :param force: 如果为真, 在使用参数中给定的子类创建一个新的接收者时,删除存在的接收者
    :return: 成功/失败
    :rtype: bool

.. method:: message_subscribe_users(user_ids=None, subtype_ids=None)

    message_subscribe 的一个包装, 使用users来代替partners.

    :param list(int) user_ids: 将要提交至记录的user ID; 如果为 ``空(None)``, 提交当前的用户.
    :param list(int) subtype_ids: 将要提交的channels/partners子类的ID
    :return: 成功
    :rtype: bool

.. method:: message_unsubscribe(partner_ids=None, channel_ids=None)

    把partner从接受者记录里移除.

    :param list(int) partner_ids: 将要提交至记录的partner ID
    :param list(int) channel_ids: 将要提交至记录的channel ID
    :return: 真
    :rtype: bool


.. method:: message_unsubscribe_users(user_ids=None)

    message_subscribe 的一个包装,使用users.

    :param list(int) user_ids: 将要提交至记录的user ID; 如果为 ``空(None)``, 提交当前的用户.
    :return: 真
    :rtype: bool


记录改变(Logging changes)
'''''''''''''''

``mail`` 模块对字段添加了一个有力的跟踪系统, 允许你来记录在记录对话中特定字段的改变.

为了添加一个字段的跟踪, 只需要简单的添加 ``onchange`` 的值的属性为track_visibility
(如果它应该只在字段改变的情况下被显示于通知中) 或者 ``always``
(如果这个值应该被一直显示于通知中,即使这个特定的字段没有改变 - 比如,在通过一直添加命名字段
来使通知更具有说明性时最有用).

.. admonition:: 示例

    让我们跟踪我们商务旅行的命名和责任者的改变:

    .. code-block:: python

        class BusinessTrip(models.Model):
            _name = 'business.trip'
            _inherit = ['mail.thread']
            _description = 'Business Trip'

            name = fields.Char(track_visibility='always')
            partner_id = fields.Many2one('res.partner', 'Responsible',
                                         track_visibility='onchange')
            guest_ids = fields.Many2many('res.partner', 'Participants')

    从现在起, 针对于旅行的命名和责任人的每个改变会在记录上进行记录.
    这个 ``name`` 字段会被显示于通知中并且对这个通知给出更多的信息(即使名字没有改变).


子类型(Subtypes)
''''''''

子类型让你对消息有更细致的控制. 子类型作为一个通知的分级,
允许提交者至一个文档来定制化他们希望接收的通知的子类型.

子类型在你的模型里被当做一个数据来创建; 模型有如下的字段:

``name`` (强制的) - :class:`~odoo.fields.Char`
    子类型的名称, 会在通知的自定义弹出框中显示
``description`` - :class:`~odoo.fields.Char`
    会添加进子类型发布的消息的描述. 如果为空, 名字会被添加进来
``internal`` - :class:`~odoo.fields.Boolean`
    有内部子类型的消息只会对雇员可见, 就像在 ``base.group_user`` 组中的组员
``parent_id`` - :class:`~odoo.fields.Many2one`
    自动提交的链接子类型; 比如项目子类型通过这个链接链接至任务子类型. 当有人被提交至一个项目,
    他会被通过这个父子类型中找到的子类型提交至这个项目所有的任务中
``relation_field`` - :class:`~odoo.fields.Char`
    比如, 当链接项目和任务子类型, 关系字段是任务的project_id字段
``res_model`` - :class:`~odoo.fields.Char`
    应用子类型的至的模型; 如果为False, 这个子类型应用于所有模型
``default`` - :class:`~odoo.fields.Boolean`
    当提交时这个子类型是否是被激活的
``sequence`` - :class:`~odoo.fields.Integer`
    用于在通知自定义的弹出框排列子类型
``hidden`` - :class:`~odoo.fields.Boolean`
    用于子类型在通知自定义的弹出框是否隐藏


使用字段跟踪的接口子类型,可以允许根据用户可能感兴趣而提交至不同的通知.
为了实现这样, 你可以复写 ``_track_subtype()`` 函数:

.. method:: _track_subtype(init_values)

    根据已经更新的值来给出由记录改变触发的子类型.

    :param dict init_values: 记录的原始值; 只有修改字段出现在字典中
    :returns: 一个子类型的完全外部id或者为False如果没有子类型被触发的话


.. admonition:: 示例

    在我们的示例类里添加一个 ``状态(state)`` 字段并且在这个字段值改变的时候使用一个特定的子类型来触发一个通知.

    首先我们要定义我们的子类型:

    .. code-block:: xml

        <record id="mt_state_change" model="mail.message.subtype">
            <field name="name">Trip confirmed</field>
            <field name="res_model">business.trip</field>
            <field name="default" eval="True"/>
            <field name="description">Business Trip confirmed!</field>
        </record>


    然后, 我们需要复写 ``track_subtype()`` 函数. 这个函数通过跟踪系统来调用,
    以此依据当前被应用的改变来获得是哪个子类型应该被使用. 在我们的情况中,
    当 ``state`` 字段从 *draft* 变为 *confirmed* 时我们想使用我们新鲜出炉的子类型:

    .. code-block:: python

        class BusinessTrip(models.Model):
            _name = 'business.trip'
            _inherit = ['mail.thread']
            _description = 'Business Trip'

            name = fields.Char(track_visibility='onchange')
            partner_id = fields.Many2one('res.partner', 'Responsible',
                                         track_visibility='onchange')
            guest_ids = fields.Many2many('res.partner', 'Participants')
            state = fields.Selection([('draft', 'New'), ('confirmed', 'Confirmed')],
                                     track_visibility='onchange')

            def _track_subtype(self, init_values):
                # init_values 包含在改变之前的被修改的字段的值
                #
                # 应用的值可以通过记录来访问因为他们已经在缓存中了
                self.ensure_one()
                if 'state' in init_values and self.state == 'confirmed':
                    return 'my_module.mt_state_change'  # Full external id
                return super(BusinessTrip, self)._track_subtype(init_values)


自定义通知(Customizing notifications)
'''''''''''''''''''''''''

当发送通知至接收者时, 在模板中添加按钮来允许从邮件快速动作是很有用的.
即使一个直接连接至记录表格视图的简单按钮也很有用; 虽然在大部分情况中你不想在外部用户中显示这些按钮.

这个通知系统允许用以下的几种方式自定义通知模板:

- 显示 *访问按钮*: 这些按钮在通知邮件的头部可见并且允许接收者直接访问记录的表格视图
- 显示 *跟从按钮*: 这些按钮允许接收者直接快速的从记录里提交
- 显示 *解除跟从按钮*: 这些按钮允许接收者直接快速的从记录里解除提交
- 显示 *自定义动作按钮*: 这些按钮被用于特定的步骤并且允许你来制作邮件里直接可用的一些有用的行为
  (比如转化一个机会至一个商机, 让财务经理使一个花费表格生效等等)

这些按钮设置可以应用于不同的组,这些组你可以通过复写函数 ``_notification_recipients`` 自定义.

.. method:: _notification_recipients(message, groups)

    根据已经更新的值给出记录上面的改变触发的子类型.

    :param ``record`` message: ``mail.message`` 当前发出的记录
    :param list(tuple) groups: 表格元组的列表(group_name, group_func,group_data)里面有:

        group_name
          是一个只用于可以复写和操作组的标识.
          默认的组是 ``user`` (链接至雇员用户的接收者),
          ``portal`` (链接至外部用户的接收者) 和 ``customer`` (没有链接至任何用户的接收者).
          一个复写使用的示例是添加一个链接至res.groups的组,就像Hr Officer给他们来设置特定的动作按钮.
        group_func
          是一个函数的指针,它把一个partner记录作为参数. 这个方法会被应用于接收者来获得是否他们属于一个给定的组.
          只有第一个匹配的组被保留. 评估顺序是这个列表的顺序.
        group_data
          一个包含通知邮件参数的字典,其有如下的键值对:

          - has_button_access
              是否在邮件中显示访问 <Document> 按钮. 对新组来说默认为真, 对外部/客户来说为假.
          - button_access
              有url和按钮标题的字典
          - has_button_follow
              是否在邮件中显示跟随按钮(如果接收者当前没有跟随这个线程). 对新组来说默认为真, 对外部/客户来说为假.
          - button_follow
              有url和按钮标题的字典
          - has_button_unfollow
              是否在邮件中显示解除跟随跟随按钮(如果接收者当前已经跟随这个线程). 对新组来说默认为真, 对外部/客户来说为假.
          - button_unfollow
              有url和按钮标题的字典
          - actions
              在通知邮件中显示的动作按钮的列表. 每个动作是一个包含url和按钮标题的字典.

    :returns: 一个子类型的完全外部id或者没有子类型被触发的话就是False


在动作列表中的url可以通过调用 ``_notification_link_helper()`` 函数自动生成:


.. method:: _notification_link_helper(self, link_type, **kwargs)

    对于一个当前记录的给定的类型生成一个链接(或者一个特定的记录如果关键字 ``model`` 和 ``res_id`` 被设置了的话).

    :param str link_type: 需要生成的链接类型; 可以是下面的任意值:

        ``view``
          连接至记录的表格视图
        ``assign``
          分配记录的用户至记录的 ``user_id`` 字段(如果有的话)
        ``follow``
          自解释的
        ``unfollow``
          自解释的
        ``workflow``
          触发一个工作流信号; 这个信号的名字应该由关键字 ``signal`` 提供
        ``method``
          调用记录的一个方法; 方法的名字应该由关键字 ``method`` 提供
        ``new``
          打开一个新纪录的空的表格视图; 你可以通过提供其id来指定一个在 ``action_id`` 里的特定的动作(数据库id或者完全解析的外部id)

    :returns: 选择的记录的链接的类型
    :rtype: str

.. admonition:: 示例

    让我们在商务旅行状态改变通知里添加一个自定义的按钮;
    这个按钮会重置状态至Draft并且只对Travel Manager组里的成员可见(``business.group_trip_manager``)

    .. code-block:: python

        class BusinessTrip(models.Model):
            _name = 'business.trip'
            _inherit = ['mail.thread', 'mail.alias.mixin']
            _description = 'Business Trip'

            # Pevious code goes here

            def action_cancel(self):
                self.write({'state': 'draft'})

            def _notification_recipients(self, message, groups):
                """ Handle Trip Manager recipients that can cancel the trip at the last
                minute and kill all the fun. """
                groups = super(BusinessTrip, self)._notification_recipients(message, groups)

                self.ensure_one()
                if self.state == 'confirmed':
                    app_action = self._notification_link_helper('method',
                                        method='action_cancel')
                    trip_actions = [{'url': app_action, 'title': _('Cancel')}]

                new_group = (
                    'group_trip_manager',
                    lambda partner: bool(partner.user_ids) and
                    any(user.has_group('business.group_trip_manager')
                    for user in partner.user_ids),
                    {
                        'actions': trip_actions,
                    })

                return [new_group] + groups


    请注意我可能已经在这个方法之外定义了我的评估函数并且定义了一个全局的函数来实现它而不是用一个匿名函数,
    但是为了在这个有时会很枯燥的文档中更简洁和简单, 我选择了前者而不是后者.

复写默认值(Overriding defaults)
'''''''''''''''''''

这里有很多方法让你自定义 ``mail.thread`` 模型的行为, 包含(但不限于):

``_mail_post_access`` - :class:`~odoo.models.Model`  属性
    为了在模型上发布一个消息需要的访问权限; 默认为需要一个 ``write`` , 也可以设置为 ``read``

环境关键字(Context keys):
    这些环境关键字可以被用于控制 ``mail.thread`` 特性
    像自动提交或者在调用 ``create()`` 时的字段跟踪或者
    ``write()`` (或者可能有用的任何其他方法).

    - ``mail_create_nosubscribe``: 在新建或者message_post, 不要提交当前用户至新的线程
    - ``mail_create_nolog``: 在创建时,别记录自动的'<Document>
      created' 消息
    - ``mail_notrack``: 在新建或者写操作时, 不要执行值跟踪新建消息
    - ``tracking_disable``: 在新建或者写操作时, 不执行MailThread特性
      (自动提交, 跟踪, 发步, ...)
    - ``mail_auto_delete``: 自动删除邮件通知; 默认为真
    - ``mail_notify_force_send``: 如果要发送少于50邮件通知, 那就直接发送它们而不使用队列; 默认为真
    - ``mail_notify_user_signature``: 在邮件通知中添加当前用户签名; 默认为真


.. _reference/mixins/mail/alias:

邮件别名(Mail alias)
----------

别名是可配置的邮件地址,其连接至特定的记录(它通常继承 ``mail.alias.mixin`` 模型)
其会在通过邮件联系时创建新的记录. 它们是让你系统从外部访问的一个简单方便的方法,
允许用户或者客户快速地在你的数据库中创建记录而不用直接连接至Odoo.

别名(Aliases)对比传入邮件网关(Incoming Mail Gateway)
'''''''''''''''''''''''''''''''''

一些人使用传入邮件网关来实现同样的目的. 你仍然需要正确的配置邮件网关来使用别名,
但是一个单独的捕捉域已经足够了,因为所有的路由会在Odoo内部完成.
别名相对于邮件网关有几种优势:

* 更容易配置
    * 一个单独的传入网关可被很多别名来使用; 这避免了子啊你的域名上配置多个邮件
      (所有的配置在Odoo内部完成)
    * 不需要系统访问权限来配置别名
* 更一致
    * 在相关记录上可配置, 而不是在设置子菜单中
* 服务器端更容易复写
    * 混入类模型的构建基础就是可扩展性, 相对于一个邮件网关,允许你从传入的邮件来更容易地释放有用的数据.


别名支持集成(Alias support integration)
'''''''''''''''''''''''''

别名通常配置于一个父模型上,其会在通过邮件交流时被创建一个特定的记录.
比如,项目有创建任务或问题的别名, 销售团队有生成商机的别名.

.. Note:: 通过别名创建的模型 **必须** 继承 ``mail_thread`` 模型.

通过继承 ``mail.alias.mixin`` 会添加别名支持; 这个混入类会对每个创建的父类生成一个新的 ``mail.alias`` 记录
(比如, 每个 ``project.project`` 记录有它的 ``mail.alias`` 记录在生成时初始化).

.. Note:: 别名可被手动生成并且被一个简单的 :class:`~odoo.fields.Many2one` 字段支持.
    这个指南假设你希望一个完全的包含自动创建别名,指定记录的默认值等的集合.

不像 ``mail.thread`` 继承, ``mail.alias.mixin`` **需要** 一些特定的复写才能正确工作.
这些复写会指定生成的别名的值, 就像它必须生成的记录和这些记录可能依赖的在其父对象上的一些默认值:

.. method:: get_alias_model_name(vals)

    返回别名的模型名字. 那些没有回复至已经存在的记录的传入的邮件会导致一个这个别名模型的新的记录的生成.
    这个值可能依赖 ``vals``, 当这个模型的一个记录被创建时传递给 ``create`` 的字典的值.

    :param vals dict: 新创建的用于保持别名的记录的值
                      the alias
    :return: 模型名称
    :rtype: str

.. method:: get_alias_values()

    返回新建一个别名的值, 或者当其创建后在别名上写.
    并不是完全强制的, 它通常被用于确保一个新建的记录会链接至别名的父
    (比如任务在当前的项目创建)通过设置一个别名 ``alias_defaults`` 字段的默认值的字典.

    :return: 会被写入新的别名的值的字典
    :rtype: dict

``get_alias_values()`` 复写是十分有趣的因为它允许你很容易的修改你的别名的行为.
在别名中可以设置的字段上, 下面的这些十分有趣:

``alias_name`` - :class:`~odoo.fields.Char`
    邮件别名的名字, 例如 '工作' 如果你想抓取 <jobs@example.odoo.com> 的邮件
``alias_user_id`` - :class:`~odoo.fields.Many2one` (``res.users``)
    通过别名接收邮件来创建的记录的所有者;
    如果这个字段没有设置,系统会基于发送者(From)的地址尝试找到正确的所有者,
    或者会使用管理员账户如果在那个地中中找不到系统用户的话
``alias_defaults`` - :class:`~odoo.fields.Text`
    在给别名创建新的记录是被评估来提供默认值Python字典
``alias_force_thread_id`` - :class:`~odoo.fields.Integer`
    线程(记录)的可选ID,所有收到的邮件会被附加其上,即使他们没有被回复; 如果设置了的话,
    这会完全禁用新记录的生成方法
``alias_contact`` - :class:`~odoo.fields.Selection`
    使用邮件网关在文档上发布一个消息的策略

    - *everyone*: 所有人都可发布
    - *partners*: 只有认证的合作伙伴
    - *followers*: 只有相关文档的接收者或者接收通道的成员

请注意别名使用了 :ref:`授权继承(delegation inheritance) <reference/orm/inheritance>`,
这意味着虽然别名存储于另一个表格中, 你可以从父对象中直接访问这些字段.
这使你可以很容易的让别名在记录的表格视图中变得可配置.

.. admonition:: 示例

    让我们在我们的商务旅行上添加一个别名来通过邮件来直接创建花费.

    .. code-block:: python

        class BusinessTrip(models.Model):
            _name = 'business.trip'
            _inherit = ['mail.thread', 'mail.alias.mixin']
            _description = 'Business Trip'

            name = fields.Char(track_visibility='onchange')
            partner_id = fields.Many2one('res.partner', 'Responsible',
                                         track_visibility='onchange')
            guest_ids = fields.Many2many('res.partner', 'Participants')
            state = fields.Selection([('draft', 'New'), ('confirmed', 'Confirmed')],
                                     track_visibility='onchange')
            expense_ids = fields.One2many('business.expense', 'trip_id', 'Expenses')
            alias_id = fields.Many2one('mail.alias', string='Alias', ondelete="restrict",
                                       required=True)

            def get_alias_model_name(self, vals):
            """ 指定在别名收到一个消息时要创建的模型 """
                return 'business.expense'

            def get_alias_values(self):
            """ 指定一些会在别名创建时设置的默认值 """
                values = super(BusinessTrip, self).get_alias_values()
                # alias_defaults 保存一个会被写入到通过这个别名创建所有记录的字典
                #
                # 这种情况下, 我们想所有的花费记录被发送至一个旅行别名上,其湖北链接至相应的商务旅行上
                values['alias_defaults'] = {'trip_id': self.id}
                # 默认情况下,我们只需要这个旅行的接收者可以发布花费
                values['alias_contact'] = 'followers'
                return values

        class BusinessExpense(models.Model):
            _name = 'business.expense'
            _inherit = ['mail.thread']
            _description = 'Business Expense'

            name = fields.Char()
            amount = fields.Float('Amount')
            trip_id = fields.Many2one('business.trip', 'Business Trip')
            partner_id = fields.Many2one('res.partner', 'Created by')

    我们需要我们的别名在我们的商务旅行的表格视图中很容易的配置, 所以让我们添加如下内容仅我们的表格视图:

    .. code-block:: xml

        <page string="Emails">
            <group name="group_alias">
                <label for="alias_name" string="Email Alias"/>
                <div name="alias_def">
                    <!-- 当在视图模式中时显示一个链接,一个可配置的字段当在编辑模式中时 -->
                    <field name="alias_id" class="oe_read_only oe_inline"
                            string="Email Alias" required="0"/>
                    <div class="oe_edit_only oe_inline" name="edit_alias"
                         style="display: inline;" >
                        <field name="alias_name" class="oe_inline"/>
                        @
                        <field name="alias_domain" class="oe_inline" readonly="1"/>
                    </div>
                </div>
                <field name="alias_contact" class="oe_inline"
                        string="Accept Emails From"/>
            </group>
        </page>

    现在我们可以直接通过商务旅行的表格视图来改变这个别名的地址,并且改变可发送邮件给这个别名的人员.

    我们接下来会在创建花费时,在我们的花费模型上复写 ``message_new()`` 来取得邮件中的值:

    .. code-block:: python

        class BusinessExpense(models.Model):
            # 之前的代码在这里
            # ...

            def message_new(self, msg, custom_values=None):
                """ 复写为根据邮件来设置值.

                在这个例子中, 我们只是使用邮件标题作为花费的名字,试着找到一个有这个邮件地址的合作伙伴
                并且进行一个正则匹配来找到花费的总数."""
                name = msg_dict.get('subject', 'New Expense')
                # 匹配最后一个字符串中的浮点数
                # 例如: '50.3 bar 34.5' 变为 '34.5'. 这是潜在的价格
                # 来编码花费. 如果不是, 就使用1.0
                amount_pattern = '(\d+(\.\d*)?|\.\d+)'
                expense_price = re.findall(amount_pattern, name)
                price = expense_price and float(expense_price[-1][0]) or 1.0
                # 通过查找其地址来找到合作伙伴
                partner = self.env['res.partner'].search([('email', 'ilike', email_address)],
                                                         limit=1)
                defaults = {
                    'name': name,
                    'amount': price,
                    'partner_id': partner.id
                }
                defaults.update(custom_values or {})
                res = super(BusinessExpense, self).message_new(msg, custom_values=defaults)
                return res


.. _reference/mixins/website:

网站特性(Website features)
================

.. _reference/mixins/website/utm:

访问者跟踪(Visitor tracking)
----------------

``utm.mixin`` 类可以被用于跟踪在线的营销/通信活动,其实现是通过连接至特定资源的参数.
这个混入类添加了3个字段至你的模型:

* ``campaign_id``: :class:`~odoo.fields.Many2one` 字段连接至 ``utm.campaign``
  对象 (例如 Christmas_Special, Fall_Collection, etc.)
* ``source_id``: :class:`~odoo.fields.Many2one` 字段连接至 ``utm.source``
  对象 (例如 搜索引擎,邮件列表等等.)
* ``medium_id``: :class:`~odoo.fields.Many2one` 字段连接至 ``utm.medium``
  对象 (i.e. 邮寄信件, 电子邮件, 社交网络更新等等.)

这些模型有一个单独的字段 ``命名(name)`` (比如,他们只是用于区别不同的活动而没有任何特殊的行为).

一旦一个客户使用url
(比如 http://www.odoo.com/?campaign_id=mixin_talk&source_id=www.odoo.com&medium_id=website)
里设置的这些参数来访问你的网站,三个cookies文件就用这些参数设置了访问者的网站.
一旦这个继承了utm.mixin的对象被从网站创建了(比如商机表格,工作应用等),
utm.mixin 代码就开始生效并且从cookies取得相应的值来在新的记录里设置他们.
一旦这一步完成了,你可以接下来在定义报表和视图时使用
campaign/source/medium字段作为任何其他字段(分组方法, 等等.).

想要扩展这个行为, 只需要简单的添加一个关系字段至一个简单模型上
(这个模型应该支持 *快速创建* (比如使用一个 ``name`` 值来调用 ``create()`` )
并且扩展这个函数 ``tracking_fields()``:

.. code-block:: python

    class UtmMyTrack(models.Model):
        _name = 'my_module.my_track'
        _description = 'My Tracking Object'

        name = fields.Char(string='Name', required=True)


    class MyModel(models.Models):
        _name = 'my_module.my_model'
        _inherit = ['utm.mixin']
        _description = 'My Tracked Object'

        my_field = fields.Many2one('my_module.my_track', 'My Field')

        @api.model
        def tracking_fields(self):
            result = super(MyModel, self).tracking_fields()
            result.append([
            # ("URL_PARAMETER", "FIELD_NAME_MIXIN", "NAME_IN_COOKIES")
                ('my_field', 'my_field', 'odoo_utm_my_field')
            ])
            return result

这会让系统用在url参数 ``my_field`` 里的值生成一个名字为 *odoo_utm_my_field* 的cookie;
一旦一个这个模型的新的记录被一个网站表格调用来创建, ``utm.mixin`` 的通用的复写方法 ``create()``
会从cookie里取得这个字段的默认的值
(并且这个 ``my_module.my_track`` 记录如果不存在的话,会被在运行中创建).

你可以在下面的模型中找到集成的具体示例:

* ``crm.lead`` 在 CRM (*crm*) 应用中
* ``hr.applicant`` 在 Recruitment Process (*hr_recruitment*) 应用中
* ``helpdesk.ticket`` 在 Helpdesk (*helpdesk* - Odoo Enterprise only) 应用中

.. _reference/mixins/website/published:

网站可见性(Website visibility)
------------------

你可以很容易的在任何一个你的记录里添加一个网站可见性的开关.
并且这个混入类非常容易来手动实现,它是继 ``mail.thread`` 之后最常用的; 这证明了它的用处.
这个混入类的最典型的使用情况是任何有一个前端页面的对象; 控制页面的可见性可以使你慢慢的编辑这个页面,
并且直到你满意之后才发布他.

为了包含这个功能,你只需要继承 ``website.published.mixin``:

.. code-block:: python

    class BlogPost(models.Model):
        _name = "blog.post"
        _description = "Blog Post"
        _inherit = ['website.published.mixin']

这个混入类在你的模型上添加了两个字段:

* ``website_published``: :class:`~odoo.fields.Boolean` 字段,它代表了发布的状态
* ``website_url``: :class:`~odoo.fields.Char` 字段,它代表了对象被访问的URL

请注意这个最后的字段是一个运算字段并且必须在你的类中被实现:

.. code-block:: python

    def _compute_website_url(self):
        for blog_post in self:
            blog_post.website_url = "/blog/%s" % (log_post.blog_id)

一旦这个机制起作用了,你只需要改编你的前端和后端视图来使得他们可被访问到.
在后端, 在按钮盒中添加一个按钮是最常用的方法:

.. code-block:: xml

    <button class="oe_stat_button" name="website_publish_button"
        type="object" icon="fa-globe">
        <field name="website_published" widget="website_button"/>
    </button>

在前端,需要一些安全检查来避免向网站访问者展示 'Edition' 按钮:

.. code-block:: xml

    <div id="website_published_button" class="pull-right"
         groups="base.group_website_publisher"> <!-- or any other meaningful group -->
        <t t-call="website.publish_management">
          <t t-set="object" t-value="blog_post"/>
          <t t-set="publish_edit" t-value="True"/>
          <t t-set="action" t-value="'blog.blog_post_action'"/>
        </t>
    </div>

请注意你必须把你的对象作为变量 ``object`` 传递给模板;
在这个例子中, ``blog.post`` 记录被作为 ``blog_post`` 变量传递给 ``qweb`` 渲染引擎,
为了发布管理模板,这样做很是很必须的.
``publish_edit`` 变量允许前端按钮链接至后端(允许你很容易的来切换前端至后端,或者相反的操作);
如果设置了, 你必须指定你想在后端调用位于 ``action`` 变量的的动作的完全的外部id
(注意这个模型必须有表格视图).

``website_publish_button`` 动作在混入类中定义并把其行为加入到你的对象中:
如果这个类有一个有效的 ``website_url`` 计算功能,用户在点击这个按钮的时候会被重定向到前端;
用户接下来可以在前端直接发布这个页面. 这保证了不会发生任何意外的在线发布.
如果没有运算功能, 就会只是触发 ``website_published`` 的布尔值.

.. _reference/mixins/website/seo:

网站元数据(Website metadata)
----------------

这个简单的混入类只是允许你很容的把元数据注入到你的前端页面中.

.. code-block:: python

    class BlogPost(models.Model):
        _name = "blog.post"
        _description = "Blog Post"
        _inherit = ['website.seo.metadata', 'website.published.mixin']

这个混入类在你的模型上添加了3个字段:

* ``website_meta_title``: :class:`~odoo.fields.Char` 字段,其允许你设置你页面的额外的标题
* ``website_meta_description``: :class:`~odoo.fields.Char` 字段,
  其包含一个页面的简短描述(有时用于搜索引擎的结果)
* ``website_meta_keywords``: :class:`~odoo.fields.Char` 字段,
  其包含一些关键字来让搜索引擎更好地对你的页面进行分类;
  "Promote" 工具会帮助你很容易的选择语意相关的关键字

这些字段在前端使用编辑器工具条中的"Promote"工具可以进行编辑.
设置这些字段可以帮助搜索引擎更好的索引你的页面.
注意搜索引擎不会只依据这些元数据来构建他们的结果; 最好的搜索引擎优化实践依然要参考可靠的素材.

.. _reference/mixins/misc:

其他(Others)
======

.. _reference/mixins/misc/rating:

客户评价(Customer Rating)
---------------

评价混入类允许通过发送邮件来让客服评分, 字段转换为一个看板流程并且聚合你的评价的统计数据.

在你的模型上添加评价
'''''''''''''''''''''''''''

为了添加评分支持, 只需要继承 ``rating.mixin`` 模型:

.. code-block:: python

    class MyModel(models.Models):
        _name = 'my_module.my_model'
        _inherit = ['rating.mixin', 'mail.thread']

        user_id = fields.Many2one('res.users', 'Responsible')
        partner_id = fields.Many2one('res.partner', 'Customer')

混入类的行为被加入到你的模型中:

* ``rating.rating`` 记录会被链接至你模型的 ``partner_id`` 字段(如果有这个字段的话).

  - 这个行为可以通过函数 ``rating_get_partner_id()`` 来复写,如果你使用其他字段而不是 ``partner_id``

* ``rating.rating`` 记录会被链接至你模型的合作伙伴的 ``user_id`` 字段 (如果有这个字段的话)
    (比如被评分的合作伙伴)

  - 这个行为可以通过函数 ``rating_get_rated_partner_id()`` 来复写,
    如果你使用其他的字段而不是 ``user_id`` (注意这个函数必须返回一个
    ``res.partner``, 对于 ``user_id`` ,系统会自动的提取合作伙伴的用户)

* 会话历史会显示评分事件(如果你的模型继承了 ``mail.thread``)

使用邮件发送评分请求
''''''''''''''''''''''''''''''

如果你希望发送邮件来请求一个评分, 只需要生成一个有链接至评分对象的邮件即可.
一个十分基础的邮件模板看起来像下面这样:

.. code-block:: xml

    <record id="rating_my_model_email_template" model="mail.template">
                <field name="name">My Model: Rating Request</field>
                <field name="email_from">${object.rating_get_rated_partner_id().email or '' | safe}</field>
                <field name="subject">Service Rating Request</field>
                <field name="model_id" ref="my_module.model_my_model"/>
                <field name="partner_to" >${object.rating_get_partner_id().id}</field>
                <field name="auto_delete" eval="True"/>
                <field name="body_html"><![CDATA[
    % set access_token = object.rating_get_access_token()
    <p>Hi,</p>
    <p>How satsified are you?</p>
    <ul>
        <li><a href="/rating/${access_token}/10">Satisfied</a></li>
        <li><a href="/rating/${access_token}/5">Not satisfied</a></li>
        <li><a href="/rating/${access_token}/1">Very unsatisfied</a></li>
    </ul>
    ]]></field>
    </record>

你的客户接下来会收到一个链接到一个简单页面的邮件,这个页面允许客户提供一个交互的反馈
(包括一个自由文本组成的的反馈信息including a free-text).

你可以很容易的通过定义一个评分的动作来使用你的表格视图集成你的评分:

.. code-block:: xml

    <record id="rating_rating_action_my_model" model="ir.actions.act_window">
        <field name="name">Customer Ratings</field>
        <field name="res_model">rating.rating</field>
        <field name="view_mode">kanban,pivot,graph</field>
        <field name="domain">[('res_model', '=', 'my_module.my_model'), ('res_id', '=', active_id), ('consumed', '=', True)]</field>
    </record>

    <record id="my_module_my_model_view_form_inherit_rating" model="ir.ui.view">
        <field name="name">my_module.my_model.view.form.inherit.rating</field>
        <field name="model">my_module.my_model</field>
        <field name="inherit_id" ref="my_module.my_model_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@name='button_box']" position="inside">
                <button name="%(rating_rating_action_my_model)d" type="action"
                        class="oe_stat_button" icon="fa-smile-o">
                    <field name="rating_count" string="Rating" widget="statinfo"/>
                </button>
            </xpath>
        </field>
    </record>

请注意有默认的评分视图 (kanban,pivot,graph) ,其允许你快速的查看你客户的评分.

你可以在下面的模型中找到具体的集成示例:

* ``project.task`` 在 Project (*rating_project*) 应用中
* ``helpdesk.ticket`` 在 Helpdesk (*helpdesk* - Odoo Enterprise only) 应用中

