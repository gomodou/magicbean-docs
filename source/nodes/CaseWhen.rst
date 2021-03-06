CaseWhen
*************

场景
=====

多条件匹配节点。用法与SQL Case When语句相同，主要用于列级别的数据转换、清洗等数据加工环节。

输入
=====

假设输入数据集或表为：

===============  ============
  gender           name
===============  ============
   F                AAA 
   M                BBB
  __NA__            CCC 
===============  ============


配置
=====

1. ``Input column`` 输入列，`CaseWhen` 条件默认操作列。
2. ``Output column`` 输出列列名，默认覆盖输入列。
3. ``outputColType``  输出列类型，默认与输入列一致。
4. 单击``Add case`` 在 ``Add case`` 弹窗中配置 ``when`` 到 ``then`` 的条件。示例中当 ``when`` ``=`` ``F`` 就 ``then`` 为 ``Female``，当 ``when`` ``=`` ``M`` 就 ``then`` 为 ``Male``
5. ``otherwise`` 不符合case when 的其它情况。可省略不写，不写情况下默认从 ``input`` 列取值。示例中配置为 ``Unknown``。

输出
=====

在输入的示例数据中，`gender` 列为简写的 `F/M/NA` ，使用 `CaseWhen` 配置清洗规则，得出结果如下：

===============  ============
  gender           name
===============  ============
   Female          AAA 
   Male            BBB
   Unknown         CCC 
===============  ============