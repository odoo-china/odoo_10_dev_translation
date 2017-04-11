:banner: banners/reports.jpg

.. highlight:: xml

============
QWeb 报表
============

报表使用HTML/QWeb编写，与Odoo中所有常规的视图一样。您可以使用 :ref:`QWeb 控制流工具 <reference/qweb>`。PDF呈现由 wkhtmltopdf_ 执行。

如果你想要创建一个特定模型的报告，你需要定义 :ref:`reference/reports/report` 和 :ref:`reference/reports/templates` 。如果你愿意，你也可以为这个报告指定一个特定的 :ref:`reference/reports/paper_formats` 。最后，如果你需要访问多个模型，你可以定义 :ref:`reference/reports/custom_reports` 类，它允许您访问模板中的更多模型和记录。

.. _reference/reports/report:

报表
======

每个报告都必须通过 :ref:`<reference/actions/report>` 来声明.

为简单起见，``<report>`` 元素可用于定义报告，而不必手动设置 :ref:`the action
<reference/actions/report>` 及其周围环境。那个 ``<report>`` 可以具有以下属性：

``id``
    生成记录的 :term:`external id`
``name`` (必填)
    只是用来在列表视图里面作为描述显示或者排序
``model`` (必填)
   	报表的模型
``report_type`` (必填)
    用于PDF报告的 ``qweb-pdf`` 或用于HTML的 ``qweb-html``
``report_name``
    报告的名称(这将是PDF输出的名称)
``groups``
    :class:`~odoo.fields.Many2many` 字段允许查看/使用当前报表的分组
``attachment_use``
    如果设置为True，报表将使用由 ``attachment`` 表达式生成的名称作为记录的附件存储；如果需要报表只生成一次（例如法律原因）
``attachment``
    定义报告名称的Python表达式；该记录可以作为变量 ``object`` 访问
``paperformat``
    希望使用的纸张格式的外部ID（如果未指定，则为默认的纸张格式）

例如::

    <report
        id="account_invoices"
        model="account.invoice"
        string="Invoices"
        report_type="qweb-pdf"
        name="account.report_invoice"
        file="account.report_invoice"
        attachment_use="True"
        attachment="(object.state in ('open','paid')) and
            ('INV'+(object.number or '').replace('/','')+'.pdf')"
    />

.. _reference/reports/templates:

报表模板
===============


可用的最小模板
-----------------------

最小模板如下所示::

    <template id="report_invoice">
        <t t-call="report.html_container">
            <t t-foreach="docs" t-as="o">
                <t t-call="report.external_layout">
                    <div class="page">
                        <h2>Report title</h2>
                        <p>This object's name is <span t-field="o.name"/></p>
                    </div>
                </t>
            </t>
        </t>
    </template>

调用 ``external_layout`` 会在报表中添加默认的页眉和页脚。PDF的核心是 ``<div class="page">`` 内的内容。模板 ``id`` 必须是报表声明中指定的名称；例如 ``account.report_invoice`` 用于上面的报表。由于这是一个QWeb模板，您可以访问模板接收的 ``docs`` 对象的所有字段。

报表中包含一些特定的变量，主要有：

``docs``
    记录当前报表
``doc_ids``
    ``docs`` 记录的id列表
``doc_model``
    模型为 ``docs`` 记录
``time``
    引用Python标准库的 :mod:`python:time`
``user``
    ``res.user`` 记录用户打印报表
``res_company``
    记录当前 ``user`` 的公司

如果想访问模板中的其他记录/模型，将需要 :ref:`自定义报表 <reference/reports/custom_reports>`。

可翻译模板
----------------------

如果希望报表为可翻译的(例如，合作伙伴的语言)，则需要定义两个模板:

* 主报表模板
* 可翻译模板

可以从属性 ``t-lang`` 中设置语言代码(例如 ``fr`` 或 ``en_US``)或记录字段的主模板调用可翻译模板。如果使用可翻译的字段（如国家/地区名称，销售条件等），则需要重新浏览具有适当上下文的相关记录。

.. warning::

    如果您的报表模板不使用可翻译记录字段，则以其他语言重新浏览记录不是必需的，并且会影响性能。

例如，让我们看看销售模块的销售订单报表::

    <!-- Main template -->
    <template id="report_saleorder">
        <t t-call="report.html_container">
            <t t-foreach="docs" t-as="doc">
                <t t-call="sale.report_saleorder_document" t-lang="doc.partner_id.lang"/>
            </t>
        </t>
    </template>

    <!-- Translatable template -->
    <template id="report_saleorder_document">
        <!-- Re-browse of the record with the partner lang -->
        <t t-set="doc" t-value="doc.with_context({'lang':doc.partner_id.lang})" />
        <t t-call="report.external_layout">
            <div class="page">
                <div class="oe_structure"/>
                <div class="row">
                    <div class="col-xs-6">
                        <strong t-if="doc.partner_shipping_id == doc.partner_invoice_id">Invoice and shipping address:</strong>
                        <strong t-if="doc.partner_shipping_id != doc.partner_invoice_id">Invoice address:</strong>
                        <div t-field="doc.partner_invoice_id" t-options="{&quot;no_marker&quot;: True}"/>
                    <...>
                <div class="oe_structure"/>
            </div>
        </t>
    </template>


主模板使用 ``doc.partner_id.lang`` 作为可转换模板 ``t-lang`` 参数，因此会以客户的语言呈现出来。这样，每个销售订单的报表将以客户的语言打印出来。如果希望仅仅翻译文档的正文，但将页眉和页脚保留为默认语言，则可以通过以下方式调用报表的外部布局::

    <t t-call="report.external_layout" t-lang="en_US">

.. note::

    请注意，这仅仅适用于调用外部模板，无法通过在除了 ``t-call`` 之外的xml节点上设置 ``t-lang`` 属性来翻译文档的一部分。如果希望翻译模板的一部分，可以使用此部分模板创建外部模板，并用 ``t-lang`` 属性从主模板中调用


条形码
--------

条形码是由控制器返回的图像，并且可以很容易地嵌入到报告中，这要归功于QWeb的语法:

.. code-block:: html

    <img t-att-src="'/report/barcode/QR/%s' % 'My text in qr code'"/>

更多参数可以作为查询字符串传递

.. code-block:: html

    <img t-att-src="'/report/barcode/?
        type=%s&value=%s&width=%s&height=%s'%('QR', 'text', 200, 200)"/>


有用的注释
--------------
* Twitter Bootstrap和FontAwesome类可以直接在报告模板中使用
* 本地CSS可以直接放在模板中
* 全局CSS可以插入到主报表布局中，通过继承其模板并插入CSS::

    <template id="report_saleorder_style" inherit_id="report.style">
      <xpath expr=".">
        <t>
          .example-css-class {
            background-color: red;
          }
        </t>
      </xpath>
    </template>
* 如果PDF报表缺少样式，可以检查 :ref:`这些指令<reference/backend/reporting/printed-reports/pdf-without-styles>`。

.. _reference/reports/paper_formats:

纸张格式
============

纸张格式是 ``report.paperformat`` 的记录，可以包含以下属性:

``name`` (必填)
    只是用来在列表视图里面作为描述显示或者排序
``description``
    格式的描述
``format``
    定义格式(A0 到 A9, B0 到 B10, Legal, Letter,Tabloid,...) 或 ``custom`` ；默认为A4纸大
    小。如果定义了页面尺寸，则不能使用非定义格式。
``dpi``
    输出DPI，默认为90
``margin_top``, ``margin_bottom``, ``margin_left``, ``margin_right``
    边距大小（mm）
``page_height``, ``page_width``
    页面尺寸（mm）
``orientation``
    图像
``header_line``
    显示标题行
``header_spacing``
    标题间距（mm）

例如::

    <record id="paperformat_frenchcheck" model="report.paperformat">
        <field name="name">French Bank Check</field>
        <field name="default" eval="True"/>
        <field name="format">custom</field>
        <field name="page_height">80</field>
        <field name="page_width">175</field>
        <field name="orientation">Portrait</field>
        <field name="margin_top">3</field>
        <field name="margin_bottom">3</field>
        <field name="margin_left">3</field>
        <field name="margin_right">3</field>
        <field name="header_line" eval="False"/>
        <field name="header_spacing">3</field>
        <field name="dpi">80</field>
    </record>

.. _reference/reports/custom_reports:

自定义报表
==============

报表模型中有一个默认的 ``get_html`` 函数，用于查找名为 :samp:`report.{module.report_name}` 的模型。如果存在，将使用它来调用QWeb引擎；否则将使用通用函数。如果希望通过在模板中更多的东西（例如其他模型记录）来自定义报表，则可以定义这个模型，覆盖函数 ``render_html`` ，并传递 ``docargs`` 字典中的对象:

.. code-block:: python

    from odoo import api, models

    class ParticularReport(models.AbstractModel):
        _name = 'report.module.report_name'
        @api.model
        def render_html(self, docids, data=None):
            report_obj = self.env['report']
            report = report_obj._get_report_from_name('module.report_name')
            docargs = {
                'doc_ids': docids,
                'doc_model': report.model,
                'docs': self,
            }
            return report_obj.render('module.report_name', docargs)

网页报表
=====================

报表由报表模块动态生成，可以通过网址直接访问

例如，可以通过访问 \http://<server-address>/report/html/sale.report_saleorder/38 ，以html模式访问销售订单报表

或者可以访问PDF版本 \http://<server-address>/report/pdf/sale.report_saleorder/38

.. _wkhtmltopdf: http://wkhtmltopdf.org
