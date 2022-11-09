结果说明
========

DNBelab C4流程分析顺利执行完后，指定的输出目录结构如下：

-  **01.data**
   提取barcode和UMI序列，并对下机数据进行质控，与参考基因组进行比对注释生成的bam文件，获取所有beads的原始表达量矩阵raw_matrix。

-  **02.count**
   确定真实有效beads结果文件，合并同一个液滴内的多个beads的组合文件将细胞tag添加到bam文件。生成细胞表达矩阵的结果目录filter_matrix。

-  **03.analysis**
   对细胞表达矩阵进行质控，过滤低质量的细胞根据表达矩阵进行细胞聚类分析和marker基因的结果文件。

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

-  使用scStar去除cDNA文库原始数据中低质量、cell
   barcode含N的reads；提取reads中的cell
   barcode和UMI序列，过滤掉带有无法识别或矫正后仍无法与white
   list比对上的reads，并对质控结果进行统计将合格的reads比对到参考基因组。使用Anno根据基因注释信息文件进行注释，进行UMIs校正，并对beads的原始reads数目，UMIs数以及基因数目进行统计。使用PISA
   count生成所有beads的表达矩阵。

-  使用parseFq去除oligo文库原始数据中低质量、cell barcode含N的
   reads；提取reads中的cell
   barcode序列，过滤掉带有无法识别或矫正后仍无法与white
   list比对上的reads，并对质控结果进行统计。

.. _112-输入项:

1.1.2 输入项
^^^^^^^^^^^^

1）\ **scStar** 对cDNA文库数据进行质控并提取cell
barcode和umi序列信息，将有效的数据与参考基因组进行比对生成bam文件。

输入参数如下：

+-----------------------+-----------+---------------------------+
| 参数                  | 类型      | 描述                      |
+=======================+===========+===========================+
| --outSAMattributes    | String    | 默认值：single            |
|                       |           | Cell。输出BAM文件的格式ta |
|                       |           | g包含UR(UMI序列)和CB(cell |
|                       |           | barcode的序列)。          |
+-----------------------+-----------+---------------------------+
| --outSAMtype          | String    | 默认值：BAM               |
|                       |           | Unsorted。输出结果格式    |
|                       |           | ，默认为BAM格式且不排序。 |
+-----------------------+-----------+---------------------------+
| --genomeDir           | Directory | 参考基因组scStar          |
|                       |           | index目录。               |
+-----------------------+-----------+---------------------------+
| --runThreadN          | Integer   | 程序运行时调用的进程数。  |
+-----------------------+-----------+---------------------------+
| --limitOutSJcollapsed | Integer   | 默认值：10000000。        |
|                       |           | 最大检测到的剪接点数量。  |
+-----------------------+-----------+---------------------------+
| --outFileNamePrefix   | String    | 输出结果目录。            |
+-----------------------+-----------+---------------------------+
| --stParaFile          | File Path | scStar质控的配置文件。    |
+-----------------------+-----------+---------------------------+
| --limitIObufferSize   | Integer   | 默认值：350000000         |
|                       |           | 。每个线程的输入/输出的最 |
|                       |           | 大可用缓冲区大小(字节)。  |
+-----------------------+-----------+---------------------------+
| --outSAMmode          | String    | 默认值：NoQC。输出        |
|                       |           | BAM文件中不包含质量数据。 |
+-----------------------+-----------+---------------------------+

**Notice**\ ：\ ``--outSAMtype``\ 在分析时不能进行排序，因为后续针对multi
mapping的reads会进行矫正舍去步骤需要同一条reads的比对在相邻位置。\ ``--outSAMmode``\ 不包含质量值是为了降低bam存储的空间。

2）\ **Anno**
对scStar生成的Aligned.out.bam注释，对umi序列进行汉明距离为1的碱基错配纠正。

输入参数如下 :

+----------+-----------+---------------------------------------------+
| 参数     | 类型      | 描述                                        |
+==========+===========+=============================================+
| -I       | File Path | scStar生成的bam文件路径。                   |
+----------+-----------+---------------------------------------------+
| -A       | File Path | GTF注释文件。                               |
+----------+-----------+---------------------------------------------+
| -L       | File Path | scStar                                      |
|          |           | 生成的cDNA_barcode_counts_raw.txt文件路径。 |
+----------+-----------+---------------------------------------------+
| -O       | File Path | 输出结果目录。                              |
+----------+-----------+---------------------------------------------+
| -C       | Integer   | 程序运行时调用的进程数。                    |
+----------+-----------+---------------------------------------------+
| -M       | String    | 默认值：chrM。线粒体染色体名称。            |
+----------+-----------+---------------------------------------------+
| -B       | File Path | cDNA文库结构文件路径，为json格式文件，包    |
|          |           | 含结构位置、汉明距离允许错配碱基数量和cell  |
|          |           | barcode白名单信息。                         |
+----------+-----------+---------------------------------------------+
| --intron | Flag      | 默认添加该                                  |
|          |           | 参数。将比对到intronic区域的reads纳入分析。 |
+----------+-----------+---------------------------------------------+
| --anno   | Integer   | 数字0为使用                                 |
|          |           | v1版本的注释逻辑，数字1使用v2版本注释逻辑。 |
+----------+-----------+---------------------------------------------+

3）\ **PISA count** 对注释后的final.bam分析获取所有beads的表达矩阵。

输入参数如下：

+-----------+-----------+--------------------------------------------+
| 参数      | 类型      | 描述                                       |
+===========+===========+============================================+
| -@        | Integer   | 解析bam文件时调用的进程数。                |
+-----------+-----------+--------------------------------------------+
| -cb       | TAG       | 默认值：CB。定义cell                       |
|           |           | barcode在bam记录中标签名称。               |
+-----------+-----------+--------------------------------------------+
| -anno_tag | TAG       | 默认值：GN。定义注释标签在DNBelab          |
|           |           | C4中为基因名。                             |
+-----------+-----------+--------------------------------------------+
| -umi      | TAG       | 默认值：UB。定义UMI标签，若同一个anno_t    |
|           |           | ag有超过一个记录有相同标签，则只计数一次。 |
+-----------+-----------+--------------------------------------------+
| -outdir   | Director  | 表达矩阵输出目录。                         |
+-----------+-----------+--------------------------------------------+
| -bam      | File Path | 输入的bam文件。                            |
+-----------+-----------+--------------------------------------------+

**Notice**\ ：PISA软件可参考\ `PISA
Wiki <https://github.com/shiquan/PISA>`__

.. _113-输出项:

1.1.3 输出项
^^^^^^^^^^^^

-  **cDNA_barcode_counts_raw.txt** cDNA文库磁珠cell
   barcode对应reads数目文件，第一列为磁珠的cell
   barcode序列，第二列为带有该barcode的reads数。

-  **Index_barcode_counts_raw.txt** oligo文库磁珠cell
   barcode对应reads数目文件，第一列为磁珠的cell
   barcode序列，第二列为带有该barcode的reads数。

-  **cDNA_sequencing_report.csv** cDNA文库质控后统计文件。

-  **Index_sequencing_report.csv** oligo文库质控后统计文件。

-  **Index_reads.fq.gz** oligo文库通过质控并完成cell barcode、droplet
   index和umi提取的fastq格式的序列文件。

-  **final.bam** cDNA文库数据比对注释后的bam文件。

-  **alignment_report.csv** 比对统计结果。

-  **anno_report.csv** 功能区域注释统计结果文件。

-  **beads_stat.txt** beads统计结果。

-  **Log.final.out** STAR比对结束后比对统计信息。

-  **Log.out** STAR软件运行时的信息。

-  **Log.progress.out** STAR运行进程监控文件。

-  **raw_matrix** 所有beads的表达矩阵文件目录。

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

-  **Fragments with Adapter** 序列中存在接头序列的reads数目占比。

-  **Q30 bases in Cell Barcode** cell
   barcode区域碱基质量值＞30的碱基个数占cell barcode区域碱基总数百分比。

-  **Q30 bases in Sample Barcode**
   样本barcode区域碱基质量值＞30的碱基个数占样本barcode区域碱基总数百分比。

-  **Q30 bases in UMI**
   UMI区域碱基质量值＞30的碱基个数占UMI区域碱基总数百分比。

-  **Q30 bases in Reads** 碱基质量值＞30的碱基个数占总碱基总数百分比。

2）\ ``alignment_report.csv`` 内容如下:

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

3）\ ``anno_report.csv`` 内容如下：

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

分析raw matrix矩阵，
区分有效液滴内和背景的beads使用barcoderanks或emptydrops方法进行cell
calling。计算beads之间的相似度，根据beads间的相似度对同一液滴内的beads合并，合并后的生成的bam计算细胞表达量矩阵。

.. _122-输入项:

1.2.2 输入项
^^^^^^^^^^^^

| 1）\ **cell_calling.R**
  对所有beads表达量矩阵计算区分有效液滴内beads和背景beads。
| 输入参数如下：

+---------------+-----------+----------------------------------------+
| 参数          | 类型      | 描述                                   |
+===============+===========+========================================+
| --matrix      | Directory | 所有beads表达量矩阵目录。              |
+---------------+-----------+----------------------------------------+
| --outdir      | Directory | 分析输出结果目录。                     |
+---------------+-----------+----------------------------------------+
| --method      | String    | 默认值：emptydrops。cell               |
|               |           | calling使                              |
|               |           | 用方法，包含barcoderanks和emptydrops。 |
+---------------+-----------+----------------------------------------+
| --expectcells | Integer   | 默认值：3000。期望获取beads数。        |
+---------------+-----------+----------------------------------------+
| --forcecells  | Integer   | 默认值：0。截取beads数。               |
+---------------+-----------+----------------------------------------+
| --minumi      | Integer   | 使用emptydrops方法时，定义beads最小    |
|               |           | 的umi数目，低于该数目的beads过滤舍弃。 |
+---------------+-----------+----------------------------------------+

| 2）\ **mergeBarcodes** oligo数据的cell barcode和droplet
  index对应统计counts数目。
| 输入参数如下：

+------+-----------+-------------------------------------------------+
| 参数 | 类型      | 描述                                            |
+======+===========+=================================================+
| -b   | File Path | 所有beads的cell                                 |
|      |           | barcode列表，用于过滤olligo的cell barcode。     |
+------+-----------+-------------------------------------------------+
| -f   | File Path | 质控并完成cell barcode、droplet                 |
|      |           | index和umi提取的fastq格式的序列文件。           |
+------+-----------+-------------------------------------------------+
| -n   | String    | 样本名称。                                      |
+------+-----------+-------------------------------------------------+
| -o   | Directory | 分析输出结果目录。                              |
+------+-----------+-------------------------------------------------+

3）\ **s1.get.similarityOfBeads**
计算beads之间相似度（同一液滴内的beads具有较一致的oligo droplet
index）。

输入参数如下：

+---------------------------+-----------+---------------------------+
| 参数                      | 类型      | 描述                      |
+===========================+===========+===========================+
| Sample name               | String    | 输入样本名称。            |
+---------------------------+-----------+---------------------------+
| CB_UB_count.txt           | File Path | cell barcode和droplet     |
|                           |           | inde                      |
|                           |           | x对应统计counts数目文件。 |
+---------------------------+-----------+---------------------------+
| beads_barcodes.txt        | File Path | cell                      |
|                           |           | calling获                 |
|                           |           | 取的有效液滴内beads列表。 |
+---------------------------+-----------+---------------------------+
| oligo_type8.txt           | File Path | oligo droplet             |
|                           |           | index白名单文件。         |
+---------------------------+-----------+---------------------------+
| Similarity.all.csv        | File Path | 输出所有cell              |
|                           |           | barcode之间存             |
|                           |           | 在的相似度统计结果文件。  |
+---------------------------+-----------+---------------------------+
| Similarity.droplet.csv    | File Path | 输出初步过滤后的cell      |
|                           |           | barcode相似度统计         |
|                           |           | 结果文件（根据有效液滴内  |
|                           |           | beads过滤）。             |
+---------------------------+-----------+---------------------------+
| Simila                    | File Path | 对Similarity.droplet.     |
| rity.droplet.filtered.csv |           | csv中存在的相同条目进行去 |
|                           |           | 除（第一列和第二列的cell  |
|                           |           | barcode互换类型）。       |
+---------------------------+-----------+---------------------------+
| -n                        | Integer   | 程序运行所调用的进程数。  |
+---------------------------+-----------+---------------------------+

4）\ **combinedListOfBeads.py**
对相似度进行过滤并生成液滴内的beads对应信息。

输入参数如下:

+----------------------+-----------+---------------------------+
| 参数                 | 类型      | 描述                      |
+======================+===========+===========================+
| --similarity_droplet | File Path | 初步过滤后的cell          |
|                      |           | ba                        |
|                      |           | rcode相似度统计结果文件（ |
|                      |           | 根据真实有效beads过滤）。 |
+----------------------+-----------+---------------------------+
| --beads_list         | File Path | cell                      |
|                      |           | calling获                 |
|                      |           | 取的有效液滴内beads列表。 |
+----------------------+-----------+---------------------------+
| --combined_list      | File Path | 液                        |
|                      |           | 滴内的beads对应信息列表。 |
+----------------------+-----------+---------------------------+
| --simi_threshold     | Float     | 默认                      |
|                      |           | 值：0.2。相似度过滤阈值。 |
+----------------------+-----------+---------------------------+

5）\ **tagAdd** 将细胞tag信息存入bam文件中

输入参数如下:

========== ========= ========================================
参数       类型      描述
========== ========= ========================================
-bam       File Path 输入final.bam文件。
-file      File Path 有效液滴内beads对应信息列表。
-out       File Path 输出添加了细胞tag信息后的bam文件。
-tag_check TAG       默认值：CB:Z:。beads的cell barcode信息。
-tag_add   TAG       默认值：DB:Z:。添加细胞tag信息。
-n         Integer   程序运行所调用的进程数。
========== ========= ========================================

.. _123-输出项:

1.2.3 输出项
^^^^^^^^^^^^

-  **beads_barcodes.txt** 有效液滴内beads的cell barcode信息文件。

-  **beads_barcodes_hex.txt** 有效液滴内beads的十六进制cell
   barcode信息文件。

-  **cutoff.csv** 按照umi数量排序的cell barcode和是否为有效液滴内beads。

-  **beads_barcode_all.txt** 所有beads的cell barcode信息。

-  **CB_UB_count.txt** oligo的cell barcode和droplet
   index组合count统计表，统计每个磁珠cell barcode捕获到的droplet
   index序列的UMIs数目。第一列表示 droplet index
   UMI数量，第二列是droplet index序列，第三列是磁珠cell barcode序列。

-  **Similarity.all.csv** 所有beads之间存在的相似度统计结果文件。

-  **Similarity.droplet.csv** 初步过滤后的cell
   barcode相似度统计结果文件（有效液滴内beads过滤）。

-  **Similarity.droplet.filtered.csv**
   对Similarity.droplet.csv中存在的相同条目进行去除（第一列和第二列的cell
   barcode互换类型）。

-  **combined_list.txt** 在同一液滴中的beads的cell
   barcode组合的文件。第一列磁珠cell barcode，第二列为cell ID。

-  **barcodeTranslate_hex.txt** barcodeTranslate.txt中beads的cell
   barcode为十六进制。

-  **barcodeTranslate.txt** 在同一液滴中的beads的cell
   barcode组合的文件。第一列磁珠beads barcode，第二列为cell
   ID。（v2版本中与combined_list.txt相同）

-  **cellNumber_merge.png**
   每个cell含有beads数量统计结果条形图png格式图片。

-  **cellNumber_merge.pdf**
   每个cell含有beads数量统计结果条形图pdf格式图片。

-  **filter_matrix** 细胞表达量矩阵目录。

-  **cellCount_report.csv** 细胞统计信息文件。

-  **anno_decon_sorted.bam**
   排序后的anno_decon.bam（将合并后的细胞tag信息存入bam文件）文件。

-  **cell_count_detail.xls** 每个细胞中基因、umi的组合测序reads的数量。

-  **saturation.xls** 不同fraction饱和度分析结果文件。

部分结果内容展示：

1）\ ``cellCount_report.csv`` 内容如下:

-  **Fraction Reads in Cells**
   位于有效液滴内beads且比对上转录本的reads和所有比对上转录本的reads的比值。

-  **Estimated Number of Cells** 鉴定的细胞数量。

-  **Total Reads Number of Cells** 所有比对上细胞的reads数量。

-  **Mean reads per cell** 每个细胞中平均的reads数量。

-  **Mean UMI counts per cell** 每个细胞中平均的umi数量。

-  **Median UMI Counts per Cell** 细胞中umi数量的中位数。

-  **Total Genes Detected** 统计所有比对上的基因数量。

-  **Mean Genes per Cell** 每个细胞中平均的基因数量。

-  **Median Genes per Cell** 细胞中基因数量的中位数。

.. _13-质控聚类:

1.3 质控聚类
~~~~~~~~~~~~

.. _131-功能描述:

1.3.1 功能描述
^^^^^^^^^^^^^^

对细胞表达矩阵进行质控过滤双胞，低质量的细胞。
降维聚类区分不同细胞群体以及输出候选的各细胞群标记基因，对细胞群体注释。

.. _132-输入项:

1.3.2 输入项
^^^^^^^^^^^^

1）\ **QC_analysis.R** 对细胞表达矩阵进行过滤。

输入参数如下:

+------+-----------+-------------------------------------------------+
| 参数 | 类型      | 描述                                            |
+======+===========+=================================================+
| -I   | Directory | 细胞表达矩阵文件目录 。                         |
+------+-----------+-------------------------------------------------+
| -D   | Integer   | 默认值：20。                                    |
|      |           | Dou                                             |
|      |           | bletFinder预测双胞的PCs参数显著的主成分的数量。 |
+------+-----------+-------------------------------------------------+
| -P   | Float     | 默认值：0.05。预测双胞比例。                    |
+------+-----------+-------------------------------------------------+
| -M   | String    | 默认值：auto。线粒体基因列表文件，auto表        |
|      |           | 示选择基因名前缀为mt或MT的基因作为线粒体基因。  |
+------+-----------+-------------------------------------------------+
| -MP  | Integer   | 默认值：15。过滤线粒体基因比例。                |
+------+-----------+-------------------------------------------------+
| -F   | Integer   | 默认值：200。细胞含有的基因数目的最小值。       |
+------+-----------+-------------------------------------------------+
| -B   | String    | 样本名称。                                      |
+------+-----------+-------------------------------------------------+
| -O   | Director  | 输出文件路径。                                  |
+------+-----------+-------------------------------------------------+

2）\ **Cluster_analysis.R**
降维聚类区分不同细胞群体以及输出候选的各细胞群标记基因 。

输入参数如下:

+------+-----------+-------------------------------------------------+
| 参数 | 类型      | 描述                                            |
+======+===========+=================================================+
| -I   | Directory | QC分析结果目录。                                |
+------+-----------+-------------------------------------------------+
| -D   | Integer   | 默认值：20。                                    |
|      |           | 用于PCA降维后的降维聚类使用的显著主成分的数量。 |
+------+-----------+-------------------------------------------------+
| -PC  | Float     | 默认值：50。用于PCA降维的主成分的数量。         |
+------+-----------+-------------------------------------------------+
| -RES | Float     | 默认值：0.5。细胞聚类分辨率。该参数设置下游     |
|      |           | 聚类的细胞群体数量，增加该值能得到更多的分群。  |
+------+-----------+-------------------------------------------------+
| -O   | Director  | 输出文件路径。                                  |
+------+-----------+-------------------------------------------------+
| -SP  | String    | 输入样本物种名称。只有Homo_sapiens,Mus          |
|      |           | _musculus,Human,Mouse可以进行细胞群体注释分析。 |
+------+-----------+-------------------------------------------------+

.. _132-输出项:

1.3.2 输出项
^^^^^^^^^^^^

1）过滤输出文件位于输出目录下的\ **QC**\ 目录内。

-  **raw_QCplot.png**
   所有细胞的基因、UMIs数目和线粒体比例小提琴图。如果未识别到线粒体基因将没有线粒体比例小提琴图。

-  **filter_QCplot.png**
   过滤后细胞的基因、UMIs数目和线粒体比例小提琴图。如果未识别到线粒体基因将没有线粒体比例小提琴图。

-  **doublets_info.txt**
   双胞统计结果文件。第一列为细胞名称，最后一列为是否鉴定为双胞。

-  **QCobject.RDS** rds格式的文件用于存储QC的结果用于后续降维聚类分析。

2）降维聚类 输出文件位于输出目录下的 **Clustering**\ 目录内。

-  **clustering_plot.png** 细胞聚类结果的UMAP展示图片。

-  **cluster.csv**
   记录每个细胞meta数据的表格文件（包括群体、umap坐标、umi数量、基因数量和预测的细胞类型）。

-  **cluster_cell.stat** 细胞聚类的结果及每个类群的细胞数目统计。

-  **marker.csv**
   所有marker基因的表格文件第一列为基因名，第二列为群体、第三列矫正后的p_value，第四列为p_value，第五列为该基因在该群体与其他群体之间的差异倍数，第六列pct.1为在当前cluster细胞中检测到该基因表达的细胞比例，第七列pct.2为在其它cluster细胞中检测到该基因表达的细胞比例。

-  **cell_report.csv** 用于降维聚类的细胞个数统计。

-  **cluster_annotation.png** 细胞聚类且注释后的UMAP展示图片。

-  **clustering_annotation_object.RDS**
   rds格式的文件用于存储降维聚类注释的结果用于后续复现该分析结果。

.. _14-报告生成:

1.4 报告生成
~~~~~~~~~~~~

.. _141-功能描述:

1.4.1 功能描述
^^^^^^^^^^^^^^

对前三个步骤的分析结果进行整理整合，生成html格式的分析报告。

.. _142-输出项:

1.4.2 输出项
^^^^^^^^^^^^

-  **scRNA_report.html** 网页分析报告。

-  **anno_decon_sorted.bam**
   排序后的anno_decon.bam文件，可用于后续分析。

-  **anno_decon_sorted.bam.bai** 排序后的anno_decon.bam文件的 bai文件。

-  **attachment**
   目录内包括细胞过滤的结果QC和降维聚类的结果Clustering。如果分析包含intronic
   reads，目录内会增加exonic区域的reads的表达量矩阵
   splice_matrix以及用于RNA velocity分析的表达量矩阵RNAvelocity_matrix。

-  **filter_feature.h5ad** h5ad格式的细胞表达量矩阵。

-  **filter_matrix** 细胞表达量矩阵。

-  **metrics_summary.xls** 部分参数的结果统计。

-  **raw_matrix** 所有beads的表达量矩阵。

.. _2-结果报告说明:

2. 结果报告说明
---------------

网页报告由\ **SUMMARY**\ 和\ **ANALYSIS**\ 组成，可点击切换。

**SUMMARY**\ 包括\ **Sample information**\ 、\ **Beads to
cells**\ 、\ **Summary**\ 、\ **Sequencing**\ 和\ **Mapping &
Annotation**\ 五部分。

.. _21-sample-information:

2.1 Sample information
~~~~~~~~~~~~~~~~~~~~~~

-  **Estimated number of cell** 细胞数目。

-  **Median UMI counts per cell** 细胞UMIs中位数。

-  **Median genes per cell** 细胞基因中位数。

-  **Mean reads per cell** 细胞平均reads数。

.. _22-beads-to-cells:

2.2 Beads to cells
~~~~~~~~~~~~~~~~~~
.. figure:: https://s2.loli.net/2022/09/27/aIsnpq9HQE3XKWL.png
   :alt: 
   
-  左图展示了beads的UMIs数目分布
   ,并推测出存在细胞的液滴内的磁珠深蓝色区域）、低UMIs和背景磁珠混合区域（浅蓝色渐变区域）、背景磁珠（位于空液滴的磁珠，灰色区灰色区域）。来自同一细胞的不同mRNA会带有相同的磁珠条形码序列和随机的UMI序列，但由于建库过程中存在的凋亡损伤细胞所释放到背景环境中的mRNA会混入反应体系中，所以空液滴内磁珠也会捕获到环境中的mRNA。

-  右图展示了每个有效液滴中包含的磁珠数目统计。

.. _23-summary:

2.3 Summary
~~~~~~~~~~~

.. figure:: https://s2.loli.net/2022/09/27/3vSYEadXxWqo7Tm.png
   :alt: 

-  **Sample name** 样本名称。

-  **Species** 样本物种名称。

-  **Estimated number of cell** 鉴定到细胞数目。

-  **Mean reads per cell** 细胞平均reads数目。

-  **Mean UMI count per cell** 细胞平均UMI数目。

-  **Median UMI counts per cell** 细胞UMI中位数。

-  **Total genes detected** 检测到的总基因种类数目。

-  **Mean genes per cell** 细胞平均基因数目。

-  **Median genes per cell** 细胞基因中位数。

-  **Fraction Reads in cells**
   比对到转录本上的reads位于有效液滴内beads的比例。

-  **Sequencing saturation** 测序饱和度。

-  **Number of cells used for clustering**
   质控后用于聚类分析的细胞数目。

.. _24-sequencing:

2.4 Sequencing
~~~~~~~~~~~~~~

.. figure:: https://s2.loli.net/2022/09/27/XTiRYeLK4ozQyFE.png
   :alt: 

-  **Number of reads** 下机数据reads总数。

-  **Reads pass QC** 通过质控的reads数目。

-  **Reads with exactly matched barcodes**
   完全匹配上不需要错配纠错的cell barcode的reads数目。

-  **Reads with failed barcodes** 配对cell
   barcode白名单失败的reads数目。

-  **Reads filtered on low quality** cell
   barcode含N或不满足质量值条件而被舍弃的reads数目。

-  **Q30 bases in Cell Barcode** cell
   barcode区域碱基质量值＞30的碱基个数占cell barcode区域碱基总数百分比。

-  **Q30 bases in UMI**
   UMI区域碱基质量值＞30的碱基个数占UMI区域碱基总数百分比。

-  **Q30 bases in reads**
   序列碱基质量值＞30的碱基个数占总碱基总数百分比。

.. _25-mapping--annotation:

2.5 Mapping & Annotation
~~~~~~~~~~~~~~~~~~~~~~~~

.. figure:: https://s2.loli.net/2022/09/27/DkwT3hWPaxcmfHd.png
   :alt: 

-  **Reads pass QC** 通过质控的 reads数目。

-  **Mapped reads** 比对上参考基因组的reads数目。

-  **Plus strand** 比对上参考基因组正链的reads数目。

-  **Minus strand** 比对上参考基因组负链的reads数目。

-  **Mitochondria ratio**
   比对上参考基因组中线粒体染色体的reads比例（默认线粒体染色体名称为chrM）。

-  **Mapping quality corrected reads**
   比对到多个位置的reads，将比对到外显子区域的条目设置成primary
   hit并将MAPQ调整成255，统计调整质量值的reads数目。

-  **Reads Mapped to Genome (Map Quality >= 0)**
   比对上参考基因组的reads比例。

-  **Reads mapped to exonic regions** 比对上外显子区域的reads比例。

-  **Reads mapped to intronic regions** 比对上内含子区域的reads比例。

-  **Reads mapped antisense to gene**
   比对上基因的reads中，比对上反义链的比例。

-  **Reads mapped to intergenic regions** 比对上基因间区的reads比例。

-  **Include introns**
   分析中是否包含比对到内含子区域的reads用于表达量计算。

**ANALYSIS包括Cluster、Marker、Cell Annotation和Saturation。**

.. _26-cluster:

2.6 Cluster
~~~~~~~~~~~

.. figure:: https://s2.loli.net/2022/09/27/1e7u4WECF9jcr5x.png
   :alt: 

-  左边UMAP图展示的是通过lovain算法对每个细胞进行聚类，聚为同一类的细胞具有相似的表达谱。每个点代表一个细胞，并按照不同的细胞类别予以着色。

-  右边UMAP图展示的是每个细胞的中UMI数分布。利用UMAP算法处理得到二维横纵坐标，每个点代表一个细胞，并按照UMI数不同予以着色。

.. _27-marker:

2.7 Marker
~~~~~~~~~~

.. figure:: https://s2.loli.net/2022/09/27/M5ia64oUk1GyPOx.png
   :alt: 

显示了每个细胞类别中差异表达基因。每个基因在每个簇与其余样品之间进行差异表达测试。P-val值是表达差异的统计显著性的量度，P-val值越小，与理论相似程度越高。p_val_adj是基于bonferroni校正，使用数据集中的所有基因进行调整后的p值。avg_log2FC是指一个簇中某基因表达与其他细胞中平均表达比例的对数值。pct.1在当前
cluster细胞中检测到该基因表达的细胞比例pct.2是在其它cluster细胞中检测到该基因表达的细胞比例。

.. _28-cell-annotation:

2.8 Cell Annotation
~~~~~~~~~~~~~~~~~~~

.. figure:: https://s2.loli.net/2022/09/27/3tGPLuOfQYpHNqa.png
   :alt: 

基于R包**\scHCL\**(注释物种为人 )和**\scMCA\**(注释物种为小鼠)的自动注释结果。只有当物种为
**\Human\** 和**\Mouse\** 时会得到该注释结果，其他物种时报告显示\ *There
is no such species reference for annnotation.*\ 。

.. _29-saturation:

2.9 Saturation
~~~~~~~~~~~~~~

.. figure:: https://s2.loli.net/2022/09/27/DIZ681Sa7RiljMn.png
   :alt: 

-  左边曲线图展示了不同比例采样测序深度的测序饱和度指标。测序饱和度受测序深度和文库复杂性的影响，当所有mRNA转录本都已测序时，它接近1.0(100%)。
   曲线末端接近平滑状态说明测序达到饱和，因为继续增加测序量，检测到的转录本也不会有特别大的变化
   。

-  右边曲线图展示了不同比例采样测序深度的每个细胞的基因中位数。曲线末端接近平滑状态说明测序达到饱和，因为继续增加测序量，每个细胞检测到的基因数也不会有特别大的变化
   。

