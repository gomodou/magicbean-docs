Incremental ETL
******************

在一般的生产环境ETL场景下，我们从源数据做数据抽取到数仓中并非全部都是一次全量加载。
大部分情况下，无论是业务库还是日志库，数据是持续更新入仓的。
业务库持续更新依赖主键，比如以ID为准更新用户信息。日志库持续更新依赖时间戳，比如按日分区的用户行为数据。

通常情况下，完成类似增量计算的流程可能需要ETL开发人员用类似 `Sqoop` 的数据抽取工具先把数据搬运到 `HDFS` 上， 然后用 `Spark(SQL)` 写出取数逻辑，最终验证完成后接入到调度系统中设定时间每日执行。
本例中我们使用魔豆系统做增量式更新的演示，尝试“一站式”的解决上述开发流程。

在之前的示例中，我们已经使用 ``Table Loader`` 完成了一次性加载的工作。
接下来需要配置该节点 ``Process`` 里的 ``Load strategy`` 为 ``Incremental load`` 模式，用于完成增量更新的操作。

在下面示例中将会提供 `原始数据` 和 `更新数据` 分别代表第一次加载的数据集和增量数据集。
如果你需要跟随本文档测试，需要先将原始数据导入自己的数据库中，然后在魔豆完成第一次计算后，按更新数据去修改源表，以此来模拟源表更新。

业务库更新的典型场景是处理有 `upsert` 情况的表，下例 `user` 表中主键为 ``id``，这意味着：

- 新用户注册会产生新的记录
- 旧用户修改会更新对应需要变更的记录

1. 构建Pipeline
=================

原始数据:

========= ============ ======== ============ 
   id        name       gender  registeTime 
========= ============ ======== ============ 
     1       AAA          F      2021-11-10      
     2       BBB          M      2021-11-10
     3       CCC          NA     2021-11-10
========= ============ ======== ============ 

配置如下：

1. 首先确保 ``Table Loader`` 节点配置为 ``Incremental load`` 模式
2. 配置 ``Incremental Key`` 选择为 ``registeTime`` ，这项配置需要用户选择传入适合用于做时间分区的字段，类型限定为 `Timestamp` 类型或 `Date` 类型
3. 配置 ``Write mode`` 为 ``Upsert``， 配置 `upsert` 更新的字段有 ``id/name/gender/registeTime``
4. 配置完成后 ``Save``， ``Table Loader`` 配置完成

``Table Loader`` 配置过程演示：

.. image:: ../_static/tableloaderexample.gif
  :width: 600
  :alt: tableloaderexample

5. 配置 ``Entity Identifier`` 连接 ``Table Loader``，将加载的数据结果转存成魔豆系统可用的数据集
6. ``Entity Identifier`` 中 ``Entity Type`` 选择 ``User``，``Column`` 选择 ``id``

完成上述步骤后执行 ``Run``，就完成了数据的第一次加载。
画布中共有两个节点 ``Table Loader`` 和 ``Entity Identifier``，分别预览节点生成的数据结果会发现：

- ``Table Loader`` 中选择的增量字段 ``registeTime`` 清洗为 ``__dt__``，魔豆系统会以 ``__dt__`` 为数据的分区键存储载入的数据
- ``Entity Identifier`` 中将 ``id`` 列重命名为 ``__uuid__`` 列，以此作为魔豆系统中的用户标识


2. 添加定时任务
==============

构建Pipeline完成后，单击画布空白处 ``Add schedule``，会跳出 ``Workflow schedule run`` 弹窗。
这里需要注意的是，如果想将一个pipeline设置为调度任务，必须先保证已经 ``Run`` 成功。
我们可以通过这里的配置，将pipeline添加为一个调度任务，让它每日定期执行。配置步骤如下：

1. ``Repeat mode`` 选择为 ``Every``
2. ``Every`` 选择的为 ``day`` 
3. ``at`` 选择为 ``00:10``
4. ``End date`` 默认空置即可 

完成上述配置后保存。配置栏中会看到任务的触发时间提示。
这样的配置默认会在系统后台生成一个类似 `cron` 的定时任务触发器，魔豆系统在每天凌晨的 `00:10` 自动执行该链路。

3. 数据更新
===========

现在，我们模拟数据更新的情况发生。假设一天已过，业务库发生了变更（如果你在跟着教程一步一步做，现在需要按照 `更新数据` 去修改源表）。

更新数据：

========= ============ ======== ============ 
   id        name       gender  registeTime 
========= ============ ======== ============ 
     1       AAA          F      2021-11-10      
     2       BBB          M      2021-11-10
     3       CCC          F      2021-11-10
     4       DDD          M      2021-11-11
========= ============ ======== ============ 

观察 `更新数据` 发现：

- 用户 ``CCC`` 修改了自己的 ``gender``，由原来的 `NA` 更新为 `F`
- 有新用户 ``DDD`` 注册，产生一行新记录 

在源数据发生 `upsert` 的前提下，手动执行 ``Run`` 再次触发 `pipeline`。
待 `pipeline` 执行完成后，预览最新生成数据会发现，系统捕捉到了 `更新` 和 `新增` 后的两处变化，生成好了最新的数据集。
相比于传统增量数仓开发流程，使用魔豆系统可以更高效的实现同等结果的工作。
