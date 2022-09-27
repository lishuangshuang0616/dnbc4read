常见问题
========

.. _1不同测序策略模式如何设置参数软件怎么适配分析:

1.不同测序策略模式如何设置参数，软件怎么适配分析？
--------------------------------------------------

2.0版本试剂的文库结构

-  cDNA：
.. figure:: https://s2.loli.net/2022/09/27/xOMpQlhtEZHJofB.png
   :align: center
   :width: 45%

-  oligo：
.. figure:: https://s2.loli.net/2022/09/27/IzaBlQOb2SvEjrW.png
   :align: center
   :width: 45%

软件分析中使用目录\ **DNBC4tools/config**\ 内的json文件来识别cell
barcode、umi、read等序列信息。

json文件格式如下：

.. code:: shell

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

在默认分析中，cDNA文库和oligo文库分开测序，且cDNA和oligo将固定序列进行了暗反应。使用默认参数，即\ ``DNBelabC4_scRNA_beads_readStructure.json``\ 和\ ``DNBelabC4_scRNA_oligo_readStructure.json``

.. code:: 

   cDNA 
   cell barcode:R1:1-10、R1:11-20
   umi:R1:21-30
   read 1:R2:1-100
   oligo
   cell barcode:R1:1-10、R1:11-20
   read 1:R2:1-30

如果cDNA文库和oligo文库在一张芯片测序，且cDNA和oligo只在R1端进行了暗反应。使用\ ``DNBelabC4_scRNA_beads_readStructure.json``\ 和\ ``DNBelabC4_scRNA_oligomix_readStructure.json``\ ，或者在DNBC4tools命令行中增加参数--mixseq。

.. code:: 

   cDNA 
   cell barcode:R1:1-10、R1:11-20
   umi:R1:21-30
   read 1:R2:1-100
   oligo
   cell barcode:R1:1-10、R1:11-20
   read 1:R2:1-10,R2:17-26,R2:33-42

如果其他测序策略可自定义json文件，根据位置信息填写location。

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

.. _3-磁珠合并原理:

3. 磁珠合并原理？
-----------------

大磁珠携带有两种接头，其中带有umi和polyA的捕获mRNA，另一种结合引物结合小磁珠，与小磁珠结合的序列长度较短构建了oligo文库。oligo文库结构：

.. figure:: https://s2.loli.net/2022/09/27/IzaBlQOb2SvEjrW.png
   :align: center
   :width: 50%

oligo文库左边绿色这部分为大磁珠的cell barcode，右边为小磁珠的cell
barcode和umi序列（cell
barcode为小磁珠身份信息，umi为去除pcr作用），通过对oligo的下机数据分析得到大磁珠的cell
barcode和小磁珠的cell
barcode的组合counts信息，即02.count/*_CB_UB_count.txt文件。

相同液滴内多个大磁珠会结合相似的小磁珠（种类和丰度，符合泊松分布），根据上面原理，计算大磁珠间的相似性，得到同一液滴内的大磁珠，合并同一液滴内大磁珠的umi信息。02.count/*_barcodeTranslate.txt文件为beads对应细胞信息。

.. _4-对某些参数不满意重新分析:

4. 对某些参数不满意，重新分析？
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
