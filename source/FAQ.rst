常见问题
========

.. _1不同测序策略模式和试剂如何设置参数获取文库结构信息软件如何适配分析:

1.不同测序策略模式和试剂如何设置参数获取文库结构信息，软件如何适配分析？
------------------------------------------------------------------------

我们设置了多个参数来读取文库的结构，包括--chemistry、--darkreaction和--customize。

在使用默认参数时，软件会自动识别是否进行了暗反应以及试剂的版本。当然我们也可以使用--chemistry、--darkreaction来定义它。目前--chemistry包括"scRNAv1HT",
和"scRNAv2HT"，--darkreaction可以分别对cDNA的R1和oligo的R1R2进行设置。比如cDNA的R1设置暗反应，oligo的R1设置暗反应，R2不设置暗反应，那么我们可以使用--darkreaction
"R1,R1"。如果--chemistry、--darkreaction依然无法来读取文库结构，我们可以使用--customize来自定义文库结构。

``scRNAv1HT``\ 试剂的文库结构

-  cDNA：
.. figure:: https://s2.loli.net/2022/09/27/xOMpQlhtEZHJofB.png
   :align: center
   :width: 45%

-  oligo：
.. figure:: https://s2.loli.net/2022/09/27/IzaBlQOb2SvEjrW.png
   :align: center
   :width: 45%

分析中使用json文件来识别cell barcode、umi、read等序列信息。

json文件格式如下：

.. code:: json

   {

       "cell barcode tag":"CB",

       "cell barcode":[

   	{

   	    "location":"R1:1-10",

               "distance":"1",

               "white list":[

                   "TAACAGCCAA",

                   "CTAAGAGTCC",

                   ...

                   "GTCTTCGGCT"

               ]

   	},

   	{

   	    "location":"R1:11-20"

               "distance":"1",

               "white list":[

                   "TAACAGCCAA",

                   "CTAAGAGTCC",

                   ...

                   "GTCTTCGGCT"

               ]

   	},

       ],

       "UMI tag":"UR",

       "UMI":{

   	"location":"R1:21-30",

       },

       "read 1":{

   	"location":"R2:1-100",

       }

   }

json文件key对应的tag信息

+---------------------------+-----------------------------------------+
| key                       | comment                                 |
+===========================+=========================================+
| cell barcode tag          | SAM tag for cell barcode, after         |
|                           | corrected. "CB" is suggested.           |
+---------------------------+-----------------------------------------+
| cell barcode              | JSON array for cell barcode segments    |
+---------------------------+-----------------------------------------+
| cell barcode raw tag      | SAM tag for raw cell barcode; "CR" is   |
|                           | suggested.                              |
+---------------------------+-----------------------------------------+
| cell barcode raw qual tag | SAM tag for cell barcode sequence       |
|                           | quality; "CY" is suggested.             |
+---------------------------+-----------------------------------------+
| distance                  | minimal Hamming distance                |
+---------------------------+-----------------------------------------+
| white list                | white list for cell barcodes            |
+---------------------------+-----------------------------------------+
| location                  | location of sequence in read 1 or 2     |
+---------------------------+-----------------------------------------+
| sample barcode tag        | SAM tag for sample barcode              |
+---------------------------+-----------------------------------------+
| sample barcode            | SAM tag for sample barcode sequence     |
|                           | quality                                 |
+---------------------------+-----------------------------------------+
| UMI tag                   | SAM tag for UMI; "UR" is suggested for  |
|                           | raw UMI; "UB" is suggested for          |
|                           | corrected UMI                           |
+---------------------------+-----------------------------------------+
| UMI qual tag              | SAM tag for UMI sequence quality        |
+---------------------------+-----------------------------------------+
| UMI                       | location value for the UMI              |
+---------------------------+-----------------------------------------+
| read 1                    | read 1 location                         |
+---------------------------+-----------------------------------------+
| read 2                    | read 2 location                         |
+---------------------------+-----------------------------------------+

cDNA的R1和oligo的R1R2都进行了暗反应时位置信息

.. code:: bash

   cDNA 

   cell barcode:R1:1-10、R1:11-20

   umi:R1:21-30

   read 1:R2:1-100

   oligo

   cell barcode:R1:1-10、R1:11-20

   read 1:R2:1-30

cDNA的R1和oligo的R1都进行了暗反应,oligo的R2没有进行暗反应时位置信息

.. code:: bash

   cDNA 

   cell barcode:R1:1-10、R1:11-20

   umi:R1:21-30

   read 1:R2:1-100

   oligo

   cell barcode:R1:1-10、R1:11-20

   read 1:R2:1-10,R2:17-26,R2:33-42
   

其他测序策略可自定义json文件，根据位置信息填写location。

.. _2cellcalling应该选择哪个参数:

2.cell_calling应该选择哪个参数？
--------------------------------

默认的 cell calling方法是 emptydrops。

-  emptydrops:

   | 先判定有效液滴 beads先采用高 umi阈值法预期捕获 N个
     beads，则按照每个 Barcode对应的 UMI数进行排序，在 UMI数最高的 N个
     cell barcode中，取第 99分位 Barcode对应的 UMI数目除以 10，作为
     cut-off。所有 cell barcode中对应的 UMI数目高于该
     cut-off即为细胞，否则为背景 ），然后使用 emptydrops对低 umi的
     beads与背景 beads进行区分（确定背景空液滴集合，使用
     Dirichlet-multinomial模型将其与每个 beads对应的 UMI
     count进行差异显著性检验，差异显著即为有效液滴内 beads否则为背景 beads）。

-  | barcoderanks：
   | 将 cell barcode按照UMI数目从高到低排列，并拟合曲线，曲线斜率变化大的点对应的
     UMI数目即为 cut-off 所有 cell barcode对应的 UMI数目高于该
     cut-off为有效液滴内 beads，否则为背景 beads。

如果对获取的细胞结果不满意，可更换cell calling方法重新进行计算或者使用forcecells确定使用 umi数量排序前 N个 beads用于分析。


.. _3-对某些参数不满意重新分析:

3. 对某些参数不满意，重新分析？
-------------------------------

DNBelab C4分析流程支持跳过已完成的步骤 。例如 02.count步骤中合并多
beads分析报错，则不需要重新分析01.data步骤步骤。DNBC4tools只需要在原分析中添加参数
``--process count,analysis,report``\ 可以 跳过
data的分析步骤的分析步骤。分析结果不满意需要重新分析时，需要判定需要调整的分析参数位于哪个阶段，然后选择分析接下来的步骤。

在 DNBC4tools data,count,analysis,report中有些参数在主程序
run中没有。通常情况下这些参数使用默认值分析即可。如果确实需要修改这些参数，可使用
data,count,analysis,report模块进行分析，再使用 run
-process参数将后续的结果分析。例如，使用
run得到分析结果和报告后，对细胞分群的结果不满意，可使用 DNBC4tools
analysis –resolution调整分群的分辨率，分析完成后在使用 DNBC4tools run
–process report完成后续的 report分析。
