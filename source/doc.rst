B2C对接文档
======================

该文档（v4.5）适用于商家与返利的技术对接，基于此文档约定，可以实现用户从返利网跳转至商 家网站首页或指定目标页，且在 RD(推广认定)期内产生的下单(包括未付款取消及最终成交、结算)，记为返利网的推广来源，同时将订单数据返回给返利网。

.. important::
    文档中标记文字为重要或易误解逻辑，请重点关注。

.. _doc_req:

开发需求
--------

 * 需商家开发的包括：一个跳转接口（即入站链接）和一个订单查询接口。同时，实现订单的实时推送至返利网订单接口。

 * 根据双方商务约定，跳转接口实现CPS/CPN、快捷登录的全部或部分功能；


.. _doc_param:

固定参数
--------

 返利网提供的固定参数包括：

 =========== ======= =======================================
 参数名       类型    描述
 =========== ======= =======================================
 s_id        int     商家在返利网的唯一ID，也称作shopid/sid
 shop_key    string  商家与返利网接口用以加密的密钥
 =========== ======= =======================================

.. _doc_jump:

跳转接口
---------

 * 返利网用户通过跳转接口进入商家网站，跳转时附带跟踪参数，接口处理完跟踪参数后（即记录返利网推广效果），将用户重定向至默认页面或指定的落地页；

 * 商家如不采用接口统一跳转的模式，也可在网站任意url后直接附加跟踪参数，实现上述记录跟踪信息；

 .. note::
    返利网不限定具体技术实现方式，只要满足功能要求即可（记录推广效果+跳转落地页）。

.. _doc_jump_sample:

接口示例
^^^^^^^^^^
  ::

   http://www.shop.com/fanli?uid=6&target_url=xxx&tc=abc123&tracking_id=12345&action_time=1294820691&code=d7b6e7b74aea

.. _doc_jump_param:

参数说明
^^^^^^^^^^

 =========== ======= ============== =========================================================================
 参数名称     类型     要求           说明
 =========== ======= ============== =========================================================================
 uid         string  必选，可为空     返利网会员标识；用户下单后与订单进行关联并随订单返回；包含字母与数字，不区分大小写
 target_url  string  必选，可为空     跳转目标落地页，为空时跳转到商家首页或约定的推广、注册页等；
 tc          string  必选，可为空     返利网订单跟踪信息；用户下单后与订单进行关联并随订单返回；
 tracking_id bigint  可选            返利网跳转的递增序列号，商家无需处理，双方可用来在日志中定位跳转详情、排查异常；
 action_time int     可选            时间戳，用以进行参数校验；
 code        string  可选            校验关键参数，防止被篡改；code=md5(uid+shop_key+action_time)
 =========== ======= ============== =========================================================================

 .. note::
    * "必选，可为空"是指商家接口必须对该参数进行处理，但不限定该参数的值为空值；
    * 以上各参数名除tracking_id外，均可由商家自定义； 
    * 如果商家不考虑对链接进行安全性校验，则不用处理action_time及code逻辑；
    * 如果约定了进行校验，但校验失败，则页面给出提示，提示用户重新从返利网跳转；具体交互样式，由双方产品确认；

.. _doc_jump_note:

重要约定
^^^^^^^^^^

 .. important::
  * 跳转接口的传参均不做空值限制；uid、tc等关键字段，必须原值传回；正常流程中不会产生空值，但商家处理逻辑不能将空值作为推广是否有效、订单是否推送的限制；

  * 跳转接口中的target_url 参数是为了更好的用户体验，可直接跳转到指定的商品详情页、专题活动页等，但不能因为任意指定target_url而导致推广效果记录失效。


.. _doc_order:

订单接口
-----------

.. _doc_order_query:

查询接口
^^^^^^^^^^

 返利网通过商家提供的订单查询接口，实时获取到属于返利网推广的订单数据。示例::

   http://www.shop.com/queryOrder?begin_date=2013-08-01%2014:00:00&end_date=2013-08-01%2015:00:00&order_id=a12345678&date_type=update

.. _doc_order_query_param:

参数说明
^^^^^^^^^^

 =========== ========= ======== =======================================
 参数名称     类型       要求     说明
 =========== ========= ======== =======================================
 begin_date  datetime  必选     订单查询开始时间点，精确到秒；
 end_date    datetime  必选     订单查询结束时间点，精确到秒；
 order_id    string    可选     指定订单号查询，返回该订单的详情；
 update      int       可选     0订单生成时间,1订单最后更新时间，默认为1
 =========== ========= ======== =======================================

 .. note::
    * 为满足查询时效性及对历史订单的补抓需求，接口应支持返回指定时间段内新产生、以及状态发生更新的订单；
    * 根据update的不同，传入的时间参数含义不同（下单时间或状态更新时间）；默认为1（注：应包含新产生订单，此时下单时间与最新状态更新时间相同）；

.. _doc_order_push:

订单推送
^^^^^^^^^

 商家通过返利网提供的接口，将属于返利网推广的订单数据实时推送到返利网接口中。数据推送需实时且符报文格式标准，推送失败时请增加重试机制。推送地址::

    http(s)://union.fanli.com/dingdan/push/shopid/1234

 .. note::
    * POST方式请求接口，如果以key/value方式提交，key=content；
    * 推送地址中的1234为文档提到的商家在返利网的s_id，技术对接时提供；

.. _doc_order_return:

推送返回
^^^^^^^^^

 .. literalinclude:: /sample/return.xml
    :language: xml

 * error_code 返回 1 为推送成功，0 为重复推送，其他表示失败;
 * error_description 推送成功或重复时返回订单号(order_id),失败则返回出错说明;

.. _doc_order_sample:

订单报文示例
^^^^^^^^^^^^^^^^^

 订单查询及推送采用一致的订单数据格式，示例如下:

 .. literalinclude:: /sample/order.xml
    :language: xml

 .. note::
    * 扩展信息字段(extension)可根据具体商务合作内容进行增删，以满足双方的合作需求；
    * 受XML格式限制，如有中文或特殊字符，请使用<![CDATA[...]]>进行处理；

.. _doc_order_param:

订单字段说明
^^^^^^^^^^^^^^^^^

 ================ ========= ======== ============================================================
 参数名称          类型       要求     说明
 ================ ========= ======== ============================================================
 orders           order     必填      以数组形式存储多个订单order信息
 s_id             int       必填      合作商家在返利网的编号,返利网提供
 order_id         string    必填      订单号
 order_id_parent  string    必填      父订单号（若无父子订单逻辑，和订单号保持一致）
 order_time       datetime  必填      订单创建时间
 uid              string    必填      跳转接口传入的uid值,原值返回
 uname            string    必填      商家用户唯一性标识，快捷登录时可使用跳转时传入的uname值，见补充说明
 tc               string    必填      跳转接口传入的tc值，原值返回
 pay_time         datetime  必填      订单支付时间，为空表示未支付
 status           int       必填      订单状态标识，见补充说明
 locked           int       可选      订单状态是否已锁定不可变更(如已过退货期、交易完成不可退货)，默认为0
 lastmod          datetime  必填      订单状态最后一次变更时间，用以保证多次推送时更新逻辑不产生混乱；
 is_newbuyer      int       必填      是否为商家的新购物用户，见补充说明
 platform         int       必填      订单产生的平台 1:PC平台;2:wap/app移动平台
 remark           string    可选      备注信息 
 products         product   必填      以数组形式存储多个商品product信息
 pid              string    必填      商品编号或SKU
 title            string    必填      商品名称
 category         string    必填      类别编码，见补充说明
 category_title   string    必填      类别名称
 url              string    必填      商品url
 num              int       必填      商品/SKU数量
 price            decimal   必填      商品单价,单位元,两位小数；
 real_pay_fee     decimal   必填      商品结算总金额,单位元,两位小数=price*num-优惠&折扣&退货等
 refund_num       int       可选      退货数量，退货后real_pay_fee需更新,num不更新
 commission       decimal   必填      佣金总额，两位小数；real_pay_fee*佣金比例
 comm_type        string    必填      佣金分类，见补充说明
 extension        扩展       选填      order或product的扩展信息
 ================ ========= ======== ============================================================

.. _doc_order_param_plus:

字段补充说明
^^^^^^^^^^^^^^^^^
 1、uname：用以区分用户在合作商城方的唯一性。如果是快捷登录模式，该字段可直接返回跳转链接中的参数值；如果是非快捷登录模式，可以返回商家的用户ID信息；

 2、category及category_tile：商品在合作商城的类目信息，可以为顶级分类，也可以返回多级分类，格式无强制要求；

 3、comm_type：根据双方商务合作约定，用以区分订单/商品不同的返佣规则；

    * 如果是全场根据订单交易额统一返佣比例，则可统一为固定值，例如：A；

    * 如根据品类区分，则可按照商品大类目进行设定（不建议使用最小商品分类，因分类较多且后续可能会不断增加，不便于双方维护）；

    * 也可直接按照佣金比例对应设置，例如返15%的为A，返10%的为B；如果有不参与返佣的商品或订单也设一个标识为0%的独立分类，例如N；

    * 该字段与category不同，category标识的是商品分类属性，comm_type标识的是商品佣金属性；

 4、realpay_fee：该价格为商品总价扣除了可能的优惠券、折扣、退货等，为实际该商品用以佣金结算的总价，整单优惠则需按比例均摊到商品；

  .. important::
     该字段的字面含义“实际支付金额”，并非用户付款金额，而是用以结算佣金的金额。

 5、status：订单状态值由商家自定义，并在订单报文中准确反馈。同时将状态值列表提供给返利网；例如：1已下单；2已付款；3已消费；4已发货；5已确认收货；6维权退货

 6、is_newbuyer：合作CPN时，需随订单传回用户的新客状态。

    * 如果确认为新客，则为1，老客为0；

    * 如商务约定为支付后判定新老客，则在未支付前的订单报文中设为2，表示未确定。

  .. important::
     新客状态与订单金额无关。商务合作可能约定首单金额超过XX元才结算佣金，但在订单推送时，不可对该字段添加金额限制条件，只要符合约定的首单规则，则标记该字段；同时需要在commission字段中返回佣金值；

 7、extension：因商家类型不同，针对特殊商家订单报文增加extension属性。该属性可附加为订单属性（用户注册信息），也可附加为商品属性（理财产品期限、收益率等）；


.. _doc_order_note:

重要约定
^^^^^^^^^^

 .. important::
  基于提高用户体验的考虑，约定如下，如有特例，请双方商务和产品另行约定：

  * 返利网的跟单模式定义为 **只要是返利网带来的推广均需跟单** ，包括但不限于：快捷登陆、非快捷登陆模式跳转下单；跳转完成后，退出并切换商家账号下单；

  * 部分商城可能存在非结佣的特殊商品，默认约定返利网带来的所有推广订单均进行推送，非结佣的部分可以用单独的comm_type进行标识。

  * 返利网要求在订单产生后即推送订单，而不是等待完成付款后；如有特例，需双方产品确认。

.. _doc_unionlogin:

快捷登录
------------

 快捷登录是普通CPS/CPN的功能增强版本，在原有基础上增加了对用户登录信息的处理，以使跳转完成后用户即处于登录状态并可立即进行下单操作；

 具体实现是在上述 :ref:`doc_jump` 上增加了用户信息校验的额外参数。

 .. note::
    * 是否进行快捷登录合作，以商务和产品约定为准。

.. _doc_unionlogin_sample:

接口示例
^^^^^^^^^^^

  ::

   http://www.shop.com/fanli?uid=6&target_url=xxx&tc=abc123&tracking_id=12345&action_time=1294820691&code=d7b6e7b74aea236623ea1aa6830f6360&syncname=true&uname=6@51fanli&usersafekey=849b59ee2e3af476

.. _doc_unionlogin_param:

参数说明
^^^^^^^^^^^^

 ================ ========= ======== ============================================================
 参数名称          类型       要求     说明
 ================ ========= ======== ============================================================
 syncname         bool      必选      是否进行快捷登录，为false则跳过登录的处理，但仍然算返利网的推广；
 action_time      int       必选      时间戳，用以进行参数校验；
 uname            string    必选      快捷登录用户名，标识返利网唯一用户；
 code             string    必选      校验关键参数，防止被篡改；code=md5(uname+shop_key+action_time)；
 usersafekey      string    必选      uname验证信息，与uname一一对应，作为对uname的校验；
 ================ ========= ======== ============================================================

 .. important::
    * uname格式为12345@51fanli形式，如果商家系统不允许使用@特殊字符，可自行转换；
    * **快捷登录接口中的code计算方式与非快捷登录的不同**；

.. _doc_unionlogin_liucheng:

快捷登录流程
^^^^^^^^^^^^^^

.. image:: images/unionlogin.png
   :align: right
   :scale: 50 %

.. _doc_unionlogin_yueding:

重要约定
^^^^^^^^^^^^^

 1、建议将uname做为商家登录用户名，如不满足商家登录名规则限制，商家可自行决定处理方案，但需与返利网对接产品确认；

 2、选择快捷登录时，订单报文中的uname字段使用接口传入的值，同样标识用户在商城的唯一性。

 3、uid字段无固定规则，商家不能使用该字段值用以逻辑判断及用户唯一性标识；应使用uname作为唯一标识；

 4、action_time验证：如果action_time与当前日期差距过大（如超过5分钟），考虑到安全，可将此次快捷登录的请求判定为无效，等同于syncname=false；

 5、不论用户是否选择了快捷登录(syncname=true)，或者快捷登录失败，当次跳转 **均算作返利网的有效推广** ，如用户产生下单，则需返回数据；
