结果说明
========

DNBelab C4流程分析顺利执行完后， 指定的输出目录结构如下：

-  **01.data** 提取 barcode和
   UMI序列，并对下机数据进行质控，与参考基因组进行比对注释生成的bam文件，获取所有
   beads的原始表达量矩阵 raw_matrix。

-  **02.count**
   确定真实有效beads结果文件，合并同一个液滴内的多个beads的组合文件将细胞tag添加到
   bam文件。生成细胞表达矩阵的结果目录 filter_matrix。

-  **03.analysis**
   对细胞表达矩阵进行质控，过滤低质量的细胞根据表达矩阵进行细胞聚类分析和marker基因的结果文件
   。

-  **04.report** 数据汇总和可视化网页报告的结果文件。

-  **output** 输出分析结果文件目录。

-  **log** 分析日志，分析使用的脚本和程序以及分析开始完成时间。

.. _1-运行步骤说明:

1. 运行步骤说明
---------------

.. _11-数据质控比对注释:

1.1 数据质控比对注释
~~~~~~~~~~~~~~~~~~~~

.. _111-功能描述:

1.1.1 功能描述
^^^^^^^^^^^^^^

-  使用scStar去除cDNA文库原始数据中低质量、cell barcode含N的reads；提取
   reads中的 cell barcode和 UMI序列， 过滤掉带有无法识别或矫正后仍无法与
   white list比对上的reads，并对质控结果进行统计将合格的
   reads比对到参考基因组 。使用 Anno根据基因注释信息文件进行注释，进行
   UMIs校正，并对 beads的原始
   reads数目，UMIs数以及基因数目进行统计。使用 PISA count生成所有
   beads的表达矩阵。

-  使用parseFq去除 oligo文库原始数据中低质量、cell barcode含 N的
   reads；提取 reads中的 cell barcode序列
   ，过滤掉带有无法识别或矫正后仍无法与 white list比对上的
   reads，并对质控结果进行统计。

.. _112-输入项:

1.1.2 输入项
^^^^^^^^^^^^

1）\ **scStar**\ 对 cDNA文库数据进行质控并提取 cell barcode和
umi序列信息，将有效的数据与参考基因组进行比对生成 bam文件。

输入参数如下：

+-----------------------+-----------+---------------------------+
| 参数                  | 类型      | 描述                      |
+=======================+===========+===========================+
| --outSAMattributes    | String    | 默认值 : singleCell。输出 |
|                       |           | BAM文件的格式tag包含      |
|                       |           | UR(UMI序列)和 CB(cell     |
|                       |           | barcode的序列)。          |
+-----------------------+-----------+---------------------------+
| --outSAMtype          | String    | 默认值 : BAM              |
|                       |           | Unsorted。输出结果格      |
|                       |           | 式，默认为BAM格式且不排序 |
|                       |           | 。                        |
+-----------------------+-----------+---------------------------+
| --genomeDir           | Directory | 参考基因组 scStar         |
|                       |           | index目录                 |
+-----------------------+-----------+---------------------------+
| --runThreadN          | Integer   | 程序运行时调用的进程数。  |
+-----------------------+-----------+---------------------------+
| --limitOutSJcollapsed | Integer   | 默认值 :                  |
|                       |           | 10000000。                |
|                       |           | 最大检测到的剪接点数量。  |
+-----------------------+-----------+---------------------------+
| --outFileNamePrefix   | String    | 输出结果目录 。           |
+-----------------------+-----------+---------------------------+
| --stParaFile          | File Path | scStar质控的配置文件。    |
+-----------------------+-----------+---------------------------+
| --limitIObufferSize   | Integer   | 默认值 : 350000000。      |
|                       |           | 每个线程的输入/输出的最   |
|                       |           | 大可用缓冲区大小(字节)。  |
+-----------------------+-----------+---------------------------+
| --outSAMmode          | String    | 默认值 : NoQC。           |
|                       |           | 输出                      |
|                       |           | BAM文件中不包含质量数据。 |
+-----------------------+-----------+---------------------------+

**Notice**\ ：\ ``--outSAMtype``\ 在分析时不能进行排序，因为后续针对multi
mapping的reads会进行矫正舍去步骤需要同一条reads的比对在相邻位置。\ ``--outSAMmode``\ 不包含质量值是为了降低bam存储的空间。

2）\ **Anno** 对 scStar生成的 Aligned.out.bam注释，对 umi序列
进行汉明距离为 1的碱基错配纠正。

输入参数如下 :

+----------+-----------+---------------------------------------------+
| 参数     | 类型      | 描述                                        |
+==========+===========+=============================================+
| -I       | File Path | scStar生成的 bam文件路径 。                 |
+----------+-----------+---------------------------------------------+
| -A       | File Path | GTF注释文件。                               |
+----------+-----------+---------------------------------------------+
| -L       | File Path | scStar生成的                                |
|          |           | cDNA_barcode_counts_raw.txt文件路径。       |
+----------+-----------+---------------------------------------------+
| -O       | File Path | 输出结果目录。                              |
+----------+-----------+---------------------------------------------+
| -C       | Integer   | 程序运行时调用的进程数。                    |
+----------+-----------+---------------------------------------------+
| -M       | String    | 默认值 : chrM。线粒体染色体名称。           |
+----------+-----------+---------------------------------------------+
| -B       | File Path | cDNA文库结构文件路径，为                    |
|          |           | json格式文件                                |
|          |           | ，包含结构位置、汉明距离允许错配碱基数量和  |
|          |           | cell barcode白名单信息。                    |
+----------+-----------+---------------------------------------------+
| --intron | Flag      | 默认添加该参数。将比对到intronic区域的      |
|          |           | reads纳入分析 。                            |
+----------+-----------+---------------------------------------------+
| --anno   | Integer   | 数字 0为使用 v1版本的注释逻辑，数字1使用    |
|          |           | v2版本注释逻辑                              |
+----------+-----------+---------------------------------------------+

3）\ **PISA count** 对注释后的final.bam分析获取所有 beads的表达矩阵。

输入参数如下：

+-----------+-----------+--------------------------------------------+
| 参数      | 类型      | 描述                                       |
+===========+===========+============================================+
| -@        | Integer   | 解析 bam文件时调用的进程数。               |
+-----------+-----------+--------------------------------------------+
| -cb       | TAG       | 默认值 : CB。 定义 cell barcode在          |
|           |           | bam记录中标签名称 。                       |
+-----------+-----------+--------------------------------------------+
| -anno_tag | TAG       | 默认值 : GN。定义注释标签在 DNBelab        |
|           |           | C4中为基因名 。                            |
+-----------+-----------+--------------------------------------------+
| -umi      | TAG       | 默认值 : UB。定义 UMI标签，若同一个        |
|           |           | anno                                       |
|           |           | _tag有超过一个记录有相同标签，则只计数一次 |
|           |           | 。                                         |
+-----------+-----------+--------------------------------------------+
| -outdir   | Director  | 表达矩阵输出目录 。                        |
+-----------+-----------+--------------------------------------------+
| -bam      | File Path | 输入的bam文件。                            |
+-----------+-----------+--------------------------------------------+

**Notice**\ ：PISA软件可参考\ `PISA
Wiki <https://github.com/shiquan/PISA>`__

.. _113-输出项:

1.1.3 输出项
^^^^^^^^^^^^

-  **cDNA_barcode_counts_raw.txt** cDNA文库磁珠 cell barcode对应
   reads数目文件，第一列为磁珠的 cell barcode序列，第二列为带有该
   barcode的 reads数 。

-  **Index_barcode_counts_raw.txt** oligo文库磁珠 cell barcode对应
   reads数目文件，第一列为磁珠的 cell barcode序列，第二列为带有该
   barcode的 reads数 。

-  **cDNA_sequencing_report.csv** cDNA文库质控后统计文件。

-  **Index_sequencing_report.csv** oligo文库质控后统计文件。

-  **Index_reads.fq.gz** oligo文库通过质控并完成 cell barcode、droplet
   index和 umi提取的 fastq格式的序列文件。

-  **final.bam** cDNA文库数据比对注释后的 bam文件。

-  **alignment_report.csv** 比对统计结果。

-  **anno_report.csv** 功能区域注释统计结果文件。

-  **beads_stat.txt** beads统计结果。

-  **Log.final.out** STAR比对结束后比对统计信息。

-  **Log.out** STAR软件运行时的信息。

-  **Log.progress.out** STAR运行进程监控文件。

-  **raw_matrix** 所有 beads的表达矩阵文件目录。

部分结果内容展示：

1）\ ``sequencing_report.csv`` 内容如下:

-  **Number of Fragments** 下机数据reads总数。

-  **Fragments pass QC** 通过质控的reads数目。

-  **Fragments Filtered on Low Qulity** cell
   barcode含N或不满足质量值条件而被舍弃的reads数目。

-  **Fragments with Failed Barcodes** 配对cell
   barcode白名单失败的reads数目。

-  **Fragments too short after Adapter Trimming**
   序列中存在接头序列并切除接头序列剩下区域过短的reads数目。

-  **Fragments with Exactly Matched Barcodes**
   完全匹配上不需要错配纠错的cell barcode的reads数目。

-  **Fragments with Adapter** 序列中存在接头序列的 reads数目占比。

-  **Q30 bases in Cell Barcode** cell
   barcode区域碱基质量值＞30的碱基个数占cell barcode区域碱基总数百分比。

-  **Q30 bases in Sample Barcode**
   样本barcode区域碱基质量值＞30的碱基个数占样本barcode区域碱基总数百分比。

-  **Q30 bases in UMI**
   UMI区域碱基质量值＞30的碱基个数占UMI区域碱基总数百分比。

-  **Q30 bases in Reads** 碱基质量值＞30的碱基个数占总碱基总数百分比。

2）\ ``alignment_report.csv``\ 内容如下:

-  **Raw reads**
   bam文件中所有的比对条目数目（为了降低bam文件的存储大小，bam文件中不包含未比对上的reads，所以Raw
   reads和Mapped reads的数目相同）。

-  **Mapped reads** 成功比对上的reads百分比。

-  **Plus strand** 比对上参考基因组正链的reads数目。

-  **Minus strand** 比对上参考基因组负链的reads数目。

-  **Mitochondria ratio**
   比对上参考基因组中线粒体染色体的reads比例（默认线粒体染色体名称为chrM）。

-  **Mapping quality corrected reads**
   比对到多个位置的reads，将比对到外显子区域的条目设置成primary
   hit并将MAPQ调整成255，统计调整质量值的reads数目。

3）\ ``anno_report.csv``\ 内容如下：

-  **Reads Mapped to Genome (Map Quality >= 0)**
   比对上参考基因组的reads比例（为了降低bam文件的存储大小，bam文件中不包含未比对上的reads，所以该值为0。

-  **Reads Mapped Confidently to Exonic Regions**
   比对上外显子区域的reads比例。

-  **Reads Mapped Confidently to Intronic Regions**
   比对上内含子区域的reads比例。

-  **Reads Mapped to both Exonic and Intronic Regions**
   同时比对上外显子和内含子的reads比例（在v2中由于注释逻辑的更改，该值为0.0%）。

-  **Reads Mapped Antisense to Gene**
   比对上基因的reads中，比对上反义链的比例。

-  **Reads Mapped to Intergenic Regions** 比对上基因间区的reads比例。

-  **Reads Mapped to Gene but Failed to Interpret Type**
   比对上基因但没有注释信息的reads比例（在v2中由于注释逻辑的更改，该值为0.0%）。

.. _12-细胞获取表达量计算:

1.2 细胞获取表达量计算
~~~~~~~~~~~~~~~~~~~~~~

.. _121-功能描述:

1.2.1 功能描述
^^^^^^^^^^^^^^

分析raw matrix矩阵， 区分有效液滴内和背景的 beads使用 barcoderanks或
emptydrops方法进行 cell calling。计算 beads之间的相似度，根据
beads间的相似度对同一液滴内的 beads合并， 合并后的生成的
bam计算细胞表达量矩阵 。

.. _122-输入项:

1.2.2 输入项
^^^^^^^^^^^^

| 1）\ **cell_calling.R** 对所有 beads表达量矩阵计算区分有效液滴内
  beads和背景 beads。
| 输入参数如下：

+---------------+-----------+----------------------------------------+
| 参数          | 类型      | 描述                                   |
+===============+===========+========================================+
| --matrix      | Directory | 所有 beads表达量矩阵目录 。            |
+---------------+-----------+----------------------------------------+
| --outdir      | Directory | 分析输出结果目录。                     |
+---------------+-----------+----------------------------------------+
| --method      | String    | 默认值 : emptydrops。cell              |
|               |           | calling使用方法，包含 barcoderanks和   |
|               |           | emptydrops。                           |
+---------------+-----------+----------------------------------------+
| --expectcells | Integer   | 默认值 : 3000。 期望获取 beads数 。    |
+---------------+-----------+----------------------------------------+
| --forcecells  | Integer   | 默认值 : 0。截取 beads数。             |
+---------------+-----------+----------------------------------------+
| --minumi      | Integer   | 使用emptydrops方法时，定义 beads最小的 |
|               |           | umi数目，低于该数目的 beads过滤舍弃 。 |
+---------------+-----------+----------------------------------------+

| 2）\ **mergeBarcodes** oligo数据的 cell barcode和 droplet
  index对应统计 counts数目。
| 输入参数如下：

+------+-----------+-------------------------------------------------+
| 参数 | 类型      | 描述                                            |
+======+===========+=================================================+
| -b   | File Path | 所有beads的cell                                 |
|      |           | barcode列表，用于过滤olligo的cell barcode。     |
+------+-----------+-------------------------------------------------+
| -f   | File Path | 质控并完成cell barcode、 droplet index和        |
|      |           | umi提取的 fastq格式的序列 文件 。               |
+------+-----------+-------------------------------------------------+
| -n   | String    | 样本名称 。                                     |
+------+-----------+-------------------------------------------------+
| -o   | Directory | 分析输出结果目录。                              |
+------+-----------+-------------------------------------------------+

3）\ **s1.get.similarityOfBeads** 计算 beads之间相似度（同一液滴内的
beads具有较一致的 oligo droplet index）。
