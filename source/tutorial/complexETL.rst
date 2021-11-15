Complex ETL
************

1. 数据源分析
=============

本示例中使用开放数据集 `eCommerce events history in electronics store <https://www.kaggle.com/mkechinov/ecommerce-events-history-in-electronics-store>`_ 完成一次包含脏数据清理和二值化处理的ETL示例。
该数据集是一张典型的行为数据表，建表语句是：

.. code-block:: sql

   create table example_events 
   (
       event_time    date,
       event_type    text,
       product_id    integer,
       category_id   numeric,
       category_code text,
       brand         text,
       price         numeric,
       user_id       numeric,
       user_session  text
   );


我们只关心示例数据中的三个字段 ``user_id/brand/price``，机器学习中的推荐系统模型可以利用这样的数据去预测某个用户的消费意愿强弱。
`price` 代表购买商品的金额，我们可以假设消费金额越高代表用户消费意愿越强。
`brand` 是商品品牌，但是值可能为空。
真实的行为表中，可能会出现空值或异常值，比如下面示例数据中最后一行记录。
我们一般的做法是将这种数据丢弃。异常值如果投入模型使用，会把模型带偏，所以需要在预处理阶段做清洗操作。

===================== ======== =============
      user_id          brand    price 
===================== ======== =============
1515915625519388267    d-link   215.41                                            
1515915625519380411    ritmix   33.32 
1515915625519320570    sony     42.42
1515915625513238515             9999999.99
===================== ======== =============

为了适配机器学习，我们将示例数据集进行二值化处理。
用户购买的商品金额大于 `100` 清洗为 `1`，小于则清洗为 `0`。

2. Pipeline配置
================

魔豆系统中，示例pipeline项目是 `ExampleProject/complexe ETL` **XXXXX** 。
本次构建的ETL链路整体效果如下：

.. image:: ../_static/complexetl.png
  :width: 600
  :alt: complexetl

具体配置步骤如下：

1. 配置 ``Table Loader`` 节点完成数据接入的工作。``input`` 内配置导入的表。此处示例表名为 ``example_events``。 使用 ``Add operation`` 的 ``Keep or drop column`` 配置只保留 ``user_id/brand/price`` 三个字段
2. 导入表配置完成后，使用 ``SQL`` 节点连接，在该节点内，提供了一个SQL编辑器，单击配置栏 ``Process`` 里的 ``Edit`` 按钮即可打开SQL编辑器窗口。编辑器窗口列出了当前可以操作的 ``Dataset`` 和可以使用的函数 ``Function`` 。SQL语句执行的结果就是下一个节点的输入。这里我们先用SQL节点的方式把 ``brand`` 为空的记录过滤掉，在编辑框内执行：

.. code-block:: sql

   SELECT * FROM example_events WHERE brand IS NOT NULL

3. 基于SQL节点执行结果，连接并配置 ``Case When`` 节点做二值化处理。配置 ``Process`` 下 ``Input column`` 和 ``Output column`` 为 ``price`` 字段。 点击 ``Add case`` 配置当 `price` ``>`` `40` 的时候，``then`` 为 `1`，然后勾选 ``otherwise`` 设置为 ``0`` 。 完成配置点击 ``Save``。(二值化就算是完成了，如果使用SQL或者其它数据处理脚本语言你可能还需要搜索下类似的 `case when` 条件怎么写。)
4. `Entity Identifier` 配置 ``Process`` ，让 ``Entity Type`` 设置为 ``User``，``Column`` 设置为 ``user_id`` 。一个经过数据清洗和二值化处理的数据集制作完成。
