Incremental ETL
******************

在一般的生产环境ETL场景下，我们从源数据做数据抽取到数仓中并非全部都是一次全量加载。
大部分情况下，无论是业务库还是日志库，数据是持续更新入仓的。
业务库持续更新依赖主键，比如以ID为准更新用户信息。日志库持续更新依赖时间戳，比如按日分区的用户行为数据。

通常情况下，完成类似增量计算的流程可能需要ETL开发人员用类似 `Sqoop` 的数据抽取工具先把数据搬运到 `HDFS` 上， 然后用 `Spark(SQL)` 写出取数逻辑，最终验证完成后接入到调度系统中设定时间每日执行。
本例中我们使用魔豆系统分别针对业务库场景和日志库场景做这种增量式更新的演示，最终“一站式”的解决上述开发流程。

在之前的示例中，我们已经使用 ``Table Loader`` 完成了一次性加载的工作。
接下来，配置该节点 ``Process`` 里的 ``Load strategy`` 为 ``Incremental load`` 模式，完成增量更新的操作。

在下面示例中将会提供 `原始数据` 和 `更新数据` 分别代表第一次加载的数据集和增量数据集。

1. 业务库增量更新
=================

业务库更新的典型场景是处理有 `upsert` 情况的表。
下例 `user` 表中主键为 ``id`` ，用户可能会更新自己的 ``name`` 和 ``gender``。

原始数据:

========= ============ ======== ============ 
   id        name       gender   sendTime 
========= ============ ======== ============ 
     1       AAA          F      2021-11-10      
     2       BBB          M      2021-11-10
     3       CCC          NA     2021-11-10
========= ============ ======== ============ 





增量数据

========= ============ ======== 
   id        name       gender 
========= ============ ======== 
     1       AAA          F
     2       BBB          M
     3       CCC          F
========= ============ ======== 

增量更新结果


2. 日志型增量更新
=================

原始数据

=========== ============ ============ 
evenetname    userId       sendTime
=========== ============ ============ 
   enter         1        2021-11-10
   enter         2        2021-11-10
   paid          2        2021-11-10
=========== ============ ============ 

增量数据

=========== ============ ============ 
evenetname    userId       sendTime
=========== ============ ============ 
   enter        1         2021-11-11
   paid         1         2021-11-11
   enter        4         2021-11-11
   enter        5         2021-11-11
=========== ============ ============ 


增量更新结果
