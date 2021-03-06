开发自测流程
==============

.. _dev_yewu:

确认业务及技术文档
----------------------
 根据产品确定的业务逻辑，阅读 :doc:`doc` 和 :doc:`docp2p`，并确认文档中的功能点无疑义，如有问题，及时与返利网技术支持确认；

 .. important::
    * 技术文档可能无法完整描述业务逻辑，请确保在开发前已经完整了解整个业务合作流程。
    * 如涉及合作模式、是否快捷登录、佣金政策、订单数据敏感字段是否提供等业务问题，优先与产品经理确认。

.. _dev_jump:

开发跳转接口
--------------

 1、文档中的接口示例仅罗列了作为基础CPS推广所需的参数及接口格式，商家可根据自身系统架构自定义，最低功能要求是支持：一个字符串跟踪参数和一个目标地址参数；

 2、跟踪参数需原值返回，返利网用以区分用户信息；目标地址参数用来跳转到指定商品详情/促销活动等页面，提升用户体验，为空时默认到首页或其他约定页；

 3、跳转接口实现记录跟踪信息并跳转至目标页的功能；如有快捷登录或P2P合作，则包含账号绑定/登录等功能；

 4、跳转接口支持签名校验，因不涉及敏感信息，默认不校验，由商家技术确认是否需要；快捷登录默认开启校验；

 .. important::
    * 及时提供跳转接口的完整示例给返利网（**必须包含发布到生产后的Url规则**）；

    * 返利网拼接跳转接口，默认对参数进行UrlEncode处理，如有特例，请及时反馈；

.. _dev_jump_test:

自测跳转接口
--------------

 1、注册并登录返利网，确保当前为返利登录状态；

 2、使用如下链接进行跳转::

    http://fun.fanli.com/goshop/go?id={s_id}&go={urlencode(目标地址)}&wp={1|0}

 .. note::
    * {s_id}为合作时提供的 :ref:`doc_param` ；
    * 如商家的接口区分pc/h5，则wp=1表示h5，为空时或wp=0表示跳转pc；

 3、上述链接会生成完整的跳转接口url并redirect跳转，开发可对生成的url进行自测；

 4、主要自测功能点：

    * 是否可以正常跳转到target_url；
    * 用户注册/下单是否记录了返利网的推广来源及传入的uid/tc跟踪信息；

 .. important::
    * 如需进行非生产环境的接口地址自测，请提供所需接口地址给返利技术进行后台配置；

.. _dev_push:

订单推送
---------------

 1、将需要推送的订单按照 :ref:`faq_order_b2c` 或 :ref:`faq_order_p2p` 生成xml；

 2、将整个xml内容，作为参数content的value值，以POST方式请求接口::

    http(s)://union.fanli.com/dingdan/push/shopid/{$s_id}

 .. note::
    * 注意参数urlencode及中文处理 ；
    * 可使用postman工具进行测试，或参考 :ref:`faq_demo_java` ；

 3、推送后会有 :ref:`doc_order_return` ，如推送成功可在返利网个人中心订单列表中显示订单内容，也可联系返利网技术支持根据订单号查询推送的订单报文详情；

 .. important::
    * 订单报文中的uid如果使用示例中的测试账号uid=6，则无法直接在订单列表中看到内容，请使用真实跳转的uid值。

 4、订单状态或内容发生变更时，需要再次推送；

.. _dev_query:

订单查询
---------------

 1、开发订单查询接口；

 2、验证接口根据下单时间、更新时间或指定订单号查询是否返回对应的数据；

 .. important::
    * 在任何时刻查询，返回的订单内容、状态及lastmod字段值都应该是当前最新值；
    * 返利网调用查询接口一般为每3～5分钟，查询最近10～30分钟内的订单，以保证订单及时同步；
    * 但因不可控原因，可能会进行手工补单，此时会按照下单时间区间进行请求；
    * 基于上述需求，要求接口既能及时返回新产生及新变更的订单，也可返回指定时间段产生的订单；

 3、测试case：用户在 T1 时间点下单，并在 T2 时间点完成支付，则按照要求：

    * 接口请求时间在T1后，时间区间仅包含T1，update=1/0均可返回订单；
    * 接口请求时间在T2后，时间区间仅包含T1，且update=0可返回订单；
    * 接口请求时间在T2后，时间区间仅包含T2，且update=1可返回订单；
    * 指定订单号，不指定查询区间，接口可返回单条订单信息；

 .. note::
    一句话解释：update=1或空，可返回create_time **或** last_modify_time为指定区间的订单；update=0， **仅** 返回create_time为指定区间的订单。

.. _dev_union_test: 

功能联调
----------------

 在完成自测并发布上线后，返利网技术会按照产品需求进行全流程测试，以确认是否符合上线条件；测试过程中，需商家技术进行配合；

 .. important::
    * 返利网技术测试属黑盒测试，受时间和成本限制，可能无法覆盖所有异常情况，恳请合作方进行有效功能自测，尽可能减少上线后因功能异常带来的客诉。

 更多测试功能点请参阅 :doc:`test` 。

 谢谢！






