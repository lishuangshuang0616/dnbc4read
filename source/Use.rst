使用说明
========


软件安装完成后可使用命令行和WDL，conda环境和docker、singularity都是基于命令行模式，WDL只能分析主流程。

.. _1命令行dnbc4tools:

1.命令行DNBC4tools
------------------

-  conda运行：

.. code:: bash

   /miniconda3/envs/DNBC4tools/bin/DNBC4tools

-  docker运行：

.. code:: bash

   docker run -P  -v $Database_LOCAL:/database -v $Rawdata_LOCAL:/data -v $Result_LOCAL:/result lishuangshuang3/dnbc4tools DNBC4tools
   # docker通过-v挂载目录到容器内
   # $Database_LOCAL: 将基因组数据库绝对路径挂载到容器/database目录下 
   # $Rawdata_LOCAL: 将下机原始数据绝对路径挂载到容器/data目录下
   # $Result_LOCAL: 将分析结果的绝对路径挂载到容器/result目录下
   # 可以使用 --user $(id -u):$(id -g) 使生成文件为使用用户属主和属组信息

-  singularity运行：

.. code:: bash

   export SINGULARITY_BIND=$cDNA_data,$oligo_data,$result,$database
   singularity exec dnbc4tools.sif DNBC4tools
   # 通过export SINGULARITY_BIND将目录挂载到容器内，可以挂载多个目录
   # 也可以在singularity exec -B $data,-B参数挂载目录或多个目录

.. _11-dnbc4tools-mkref-构建数据库:

1.1 DNBC4tools mkref (构建数据库)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

分析需要比对参考基因组和注释文件注释分析。在分析前需要创建对应物种的参考基因组数据库。需要准备两个文件，基因组的DNA序列文件（
FASTA格式）和基因的注释文件（GTF格式）。常用的 Ensembl和
GENECODE数据库提供了这两种格式的文件。

.. _111-查看注释文件的基因类型:

1.1.1 查看注释文件的基因类型
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

在构建数据库之前可以对 gtf文件的基因类型进行过滤，先查看 gtf中的
gene类型确定需要过滤哪些基因类型。

.. code:: bash

   DNBC4tools mkref --action stat --ingtf gene.gtf --type gene_type --outstat gtf_type.txt

-  DNBC4tools mkref --action stat 输入项：

========= ========= ====================================
参数      类型      描述
========= ========= ====================================
--ingtf   File Path 输入需要查看gene类型统计的gtf文件。
--type    String    gtf中gene类型的tag。
--outstat File Path 结果保存的文件，默认为gtf_type.txt。
========= ========= ====================================

-  输出结果\ ``gtf_type.txt``

===== ===========================================================
列名  描述
===== ===========================================================
Type  gtf中的基因类型。包括protein_coding、lncRNA、pseudogene等。
Count 每种基因类型的数目。
===== ===========================================================

可查看gtf的类型选取需要的基因类型

.. code:: bash

   $cat gtf_type.txt

   Type	Count
   protein_coding	21884
   processed_pseudogene	9999
   lncRNA	9949
   TEC	3237
   unprocessed_pseudogene	2718
   miRNA	2206
   snoRNA	1507
   snRNA	1381
   misc_RNA	562
   rRNA	354
   transcribed_processed_pseudogene	300
   transcribed_unprocessed_pseudogene	272
   IG_V_gene	218
   IG_V_pseudogene	158
   TR_V_gene	144

**Notice**\ ：--type选择需要根据gtf类型的tag来选择，如ensemble是\ ``--type gene_biotype``,genecode是\ ``--type gene_type``\ 。

.. _112-过滤注释文件:

1.1.2 过滤注释文件
^^^^^^^^^^^^^^^^^^

在构建数据库之前可以对
gtf文件的基因类型进行过滤，使其中仅包含感兴趣的基因类别，
过滤哪些基因取决于您的研究问题。

软件分析中，gtf中存在 overlap的基因将导致reads被舍弃 。
通过过滤gtf文件使其只有少量重叠的基因。

.. code:: bash

   DNBC4tools mkref --action mkgtf --ingtf gene.gtf --outgtf gene.filter.gtf \
               --attribute gene_type:protein_coding \
                           gene_type:lncRNA \
                           gene_type:IG_C_gene \
                           gene_type:IG_D_gene \
                           gene_type:IG_J_gene \
                           gene_type:IG_LV_gene \
                           gene_type:IG_V_gene \
                           gene_type:IG_V_pseudogene \
                           gene_type:IG_J_pseudogene \
                           gene_type:IG_C_pseudogene \
                           gene_type:TR_C_gene \
                           gene_type:TR_D_gene \
                           gene_type:TR_J_gene \
                           gene_type:TR_V_gene \
                           gene_type:TR_V_pseudogene \
                           gene_type:TR_J_pseudogene

-  DNBC4tools mkref --action mkgtf 输入项:

+-------------+-----------+------------------------------------------+
| 参数        | 类型      | 描述                                     |
+=============+===========+==========================================+
| --ingtf     | File Path | 输入需要进行过滤的gtf文件。              |
+-------------+-----------+------------------------------------------+
| --outgtf    | File Path | 输出过滤后的gtf文件。                    |
+-------------+-----------+------------------------------------------+
| --attribute | File Path | 通过attribute属性来筛选基因类型          |
|             |           | ，每个组合使用tag                        |
|             |           | 对应type冒号连接，多个类型使用空格间隔。 |
+-------------+-----------+------------------------------------------+

**Notice**\ ：--type选择需要根据gtf类型的tag来选择，如ensemble是\ ``--type gene_biotype``,genecode是\ ``--type gene_type``\ 。

.. _113-构建数据库:

1.1.3 构建数据库
^^^^^^^^^^^^^^^^

使用比对软件scStar进行数据库的构建。scStar的STAR版本为
2.7.2b，基因组版本为2.7.1a，相同基因组版本的STAR构建的数据库可通用，不同的基因组版本不可互用
。数据库不向下兼容 version1版本的数据库。

.. code:: bash

   DNBC4tools mkref --action mkref --ingtf gene.filter.gtf \
               --fasta genome.fa \
               --star_dir $star_dir \
               --thread $threads

-  DNBC4tools mkref --action mkref 输入项：

========== ========= ===============================
参数       类型      描述
========== ========= ===============================
--ingtf    File Path 输入构建star数据库的gtf文件。
--fasta    File Path 输入与gtf文件配套的参考基因组。
--star_dir Directory 数据库的结果目录 。
--thread   Integer   程序运行所调用的进程数。
========== ========= ===============================

.. _114-构建数据库参考文件:

1.1.4 构建数据库参考文件
^^^^^^^^^^^^^^^^^^^^^^^^

**Ref-202203**

-  **Human(GRCh38)**

   .. code:: bash

      http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/GRCh38.primary_assembly.genome.fa.gz
      http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/gencode.v32.primary_assembly.annotation.gtf.gz

-  **Mouse(GRCm38)**

   .. code:: bash

      http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/GRCm38.primary_assembly.genome.fa.gz
      http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/gencode.vM23.primary_assembly.annotation.gtf.gz

.. _12-dnbc4tools-run-运行主程序:

1.2 DNBC4tools run (运行主程序)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

run命令为运行主程序

.. code:: bash

   # 主程序示例

   DNBC4tools run --cDNAfastq1 cDNA_R1.fastq.gz \

   		--cDNAfastq2 cDNA_R2.fastq.gz \

   		--oligofastq1 oligo1_1.fq.gz,oligo2_1.fq.gz \

   		--oligofastq2 oligo1_2.fq.gz,oligo2_2.fq.gz \

   		--starIndexDir /database/Mouse/mm10/ --gtf /database/Mouse/mm10/genes.gtf \

   		--name test --species Mouse --thread 10

分析参数如下：

+-----------------+-----------+--------------------------------------+
| 参数            | 类型      | 描述                                 |
+=================+===========+======================================+
| --name          | String    | 必选 。样本名称 。                   |
+-----------------+-----------+--------------------------------------+
| --cDNAfastq1    | File Path | 必选                                 |
|                 |           | 。cDNA文库fastq格                    |
|                 |           | 式的R1端序列，多个文件使用逗号隔开。 |
+-----------------+-----------+--------------------------------------+
| --cDNAfastq2    | File Path | 必选                                 |
|                 |           | 。                                   |
|                 |           | cDNA文库fastq格式的R2端序列，多个文  |
|                 |           | 件使用逗号隔开，顺序与cDNAfastq1相同 |
|                 |           | 。                                   |
+-----------------+-----------+--------------------------------------+
| --oligofastq1   | File Path | 必选                                 |
|                 |           | 。oligo文库fastq格                   |
|                 |           | 式的R1端序列，多个文件使用逗号隔开。 |
+-----------------+-----------+--------------------------------------+
| --oligofastq2   | File Path | 必选                                 |
|                 |           | 。ol                                 |
|                 |           | igo文库fastq格式的R2端序列，多个文件 |
|                 |           | 使用逗号隔开，顺序与oligofastq1相同  |
|                 |           | 。                                   |
+-----------------+-----------+--------------------------------------+
| --starIndexDir  | Directory | 必选                                 |
|                 |           | 。参考基因组构建数据库索引路径。     |
+-----------------+-----------+--------------------------------------+
| --gtf           | File Path | 必选 。参考基因组注释文件gtf路径。   |
+-----------------+-----------+--------------------------------------+
| --species       | String    | 可选。样本物种名称，默认为NA。       |
+-----------------+-----------+--------------------------------------+
| --outdir        | Directory | 可选 。分析结果路径，默认为当前路径  |
|                 |           | 。                                   |
+-----------------+-----------+--------------------------------------+
| --thread        | Integer   | 可选 。程序运行时调用的进程数        |
|                 |           | ，默认为 4。                         |
+-----------------+-----------+--------------------------------------+
| --cDNAconfig    | File Path | 可选 。cDNA文库结构文件路径          |
|                 |           | ，为json格式文件，包含结构           |
|                 |           | 位置、汉明距离允许错配碱基数量和cell |
|                 |           | barcode白名单信息。                  |
+-----------------+-----------+--------------------------------------+
| --oligoconfig   | File Path | 可选 。oligo文库结构文件路径         |
|                 |           | ，为json格式文件，包含结构           |
|                 |           | 位置、汉明距离允许错配碱基数量和cell |
|                 |           | barcode白名单信息。                  |
+-----------------+-----------+--------------------------------------+
| --oligotype     | File Path | 可选 。oligo文库droplet              |
|                 |           | index白名单文件 。                   |
+-----------------+-----------+--------------------------------------+
| -calling_method | String    | 可选。默认 : emptydrops。cell        |
|                 |           | calling鉴定有效液滴内beads的         |
|                 |           | 方法，可选barcoderanks和emptydrops。 |
+-----------------+-----------+--------------------------------------+
| --expectcells   | Integer   | 可选。默认 :                         |
|                 |           | 3000。期望细胞数，仅当               |
|                 |           | calling_method为emptydrops时参数有效 |
|                 |           | 。                                   |
+-----------------+-----------+--------------------------------------+
| --forcecells    | Integer   | 可选。默认 : 0。截取 beads数量 。    |
+-----------------+-----------+--------------------------------------+
| --mtgenes       | String    | 可选。默认 :                         |
|                 |           | auto。                               |
|                 |           | 线粒体基因列表文件，auto表示选择基因 |
|                 |           | 名前缀为mt或MT的基因作为线粒体基因。 |
+-----------------+-----------+--------------------------------------+
| --process       | String    | 可选。默认 :                         |
|                 |           | data,count,anlysi                    |
|                 |           | s,report。选择需要分析的步骤，可选择 |
|                 |           | data,count,anlysis,report其中        |
|                 |           | 几项（该参数常用于已分析完成需要重新 |
|                 |           | 调整参数时使用，更改某一步骤参数后面 |
|                 |           | 的步骤也需要重新分析），用逗号分隔。 |
+-----------------+-----------+--------------------------------------+
| --no_introns    | Flag      | 比对到 intronic区域的                |
|                 |           | reads不纳入进表达量矩阵计算。        |
+-----------------+-----------+--------------------------------------+
| --mixseq        | Flag      | 该参数用于cDNA文库和                 |
|                 |           | oligo文库混合测序样本，添加          |
|                 |           | 该参数OligoBarcode默认使用DNBelabC4  |
|                 |           | _scRNA_oligomix_readStructure.json。 |
+-----------------+-----------+--------------------------------------+
| --no_bam        | Flag      | 添加该参数则不会将02.count中的       |
|                 |           | anno_decon_sorted.bam和              |
|                 |           | anno_decon_sorted.bam.bai移动到      |
|                 |           | output目录中。后续使用 DNBC4tools    |
|                 |           | clean时则会删除该                    |
|                 |           | bam文件减少存储占用。                |
+-----------------+-----------+--------------------------------------+
| --dry           | Flag      | 不进行流                             |
|                 |           | 程分析。只打印分析步骤的shell文件。  |
+-----------------+-----------+--------------------------------------+

对参数的详细说明：

-  ``--cDNAconfig``\ 和\ ``--oligoconfig``\ 在默认情况下会调用\ ``DNBC4toos/config``\ 中的\ ``DNBelabC4_scRNA_beads_readStructure.json``\ 和\ ``DNBelabC4_scRNA_oligo_readStructure.json``,对json格式的说明请参考常见问题说明的配置文件json。在测序时，cDNA文库和oligo文库分别在不同芯片上测序，且cDNA的R1和oligo的R1R2都进行了暗反应即使用默认参数。如果cDNA文库和oligo文库上机同一张芯片，且只对R1端进行了暗反应，则可使用\ ``DNBelabC4_scRNA_beads_readStructure.json``\ 和\ ``DNBelabC4_scRNA_oligomix_readStructure.json``\ ，或者直接添加--mixseq参数。如果不适用暗反应与其他文库混测，则需要自定义json文件。

-  ``--callling_method``\ ，在默认情况下会使用emptydrops，如果对结果不满意也可以尝试barcoderanks。两种cell
   calling方法的原理请参考常见问题说明。

-  ``--mtgenes``\ ，默认为auto，表示选择基因名前缀为mt或MT的基因作为线粒体基因。也可以使用自定义mtgenes的列表文件。文件内容如下：

   .. code:: 

      mt-Nd1
      mt-Nd2
      mt-Co1
      mt-Co2
      mt-Atp8
      mt-Atp6
      mt-Co3

-  ``--no_introns``\ ，分析中默认会将比对到内含子的reads加入到表达量矩阵分析。虽然不推荐，但用户可以使用这个参数将内含子数据丢弃。

-  ``--species``\ ，信息会展示在结果报告中，如果信息为Human或者Mouse会进行细胞群体注释分析。

-  ``--process``\ ，默认 :
   data,count,anlysis,report。选择需要分析的步骤，可选择
   data,count,anlysis,report其中几个步骤。每个步骤可单独使用DNBC4tools去分析，具体见以下步骤。

.. _121-dnbc4tools-data:

1.2.1 DNBC4tools data
^^^^^^^^^^^^^^^^^^^^^

提取 barcode和 UMI序列，并对下机数据进行质控与参考基因组进行比对注释
，获取所有 beads的原始表达量矩阵 。

参数保持与DNBC4tools run一致。

.. _122-dnbc4tools-count:

1.2.2 DNBC4tools count
^^^^^^^^^^^^^^^^^^^^^^

确定有效液滴内 beads，合并同一个液滴内的多个 beads计算细胞表达矩阵。

分析参数如下：

+---------------------+-----------+----------------------------------+
| 参数                | 类型      | 描述                             |
+=====================+===========+==================================+
| --bam               | File Path | 必选，data步骤生成的             |
|                     |           | final.bam文件。                  |
+---------------------+-----------+----------------------------------+
| --raw_matrix        | Directory | 必选，data步骤生成的             |
|                     |           | raw_matrix矩阵目录。             |
+---------------------+-----------+----------------------------------+
| --cDNAbarcodeCount  | File Path | 必选，data步骤生成的             |
|                     |           | c                                |
|                     |           | DNA_barcode_counts_raw.txt文件。 |
+---------------------+-----------+----------------------------------+
| --Indexreads        | File Path | 必选，data步骤生成的             |
|                     |           | Index_reads.fq.gz文件。          |
+---------------------+-----------+----------------------------------+
| --oligobarcodeCount | File Path | 必选，data步骤生成的             |
|                     |           | Ind                              |
|                     |           | ex_barcode_counts_raw.txt.文件。 |
+---------------------+-----------+----------------------------------+
| --minumi            | Integer   | 可选，默认1000。 cell calling中  |
|                     |           | emptydrops方法中可获取的         |
|                     |           | beads最小的 umi数量 。           |
+---------------------+-----------+----------------------------------+

.. _123-dnbc4tools-analysis:

1.2.3 DNBC4tools analysis
^^^^^^^^^^^^^^^^^^^^^^^^^

对细胞表达矩阵进行质控，过滤低质量的细胞根据表达矩阵进行细胞聚类分析和标记基因筛选。

分析参数如下：

+---------------------+-----------+----------------------------------+
| 参数                | 类型      | 描述                             |
+=====================+===========+==================================+
| --matrix            | Directory | 必选， count步骤 生成的          |
|                     |           | filter_matrix表达量矩阵目录。    |
+---------------------+-----------+----------------------------------+
| --qcdim             | String    | 可选，默认20。 DoubletFinder的   |
|                     |           | PCs参数显著的主成分的数量。      |
+---------------------+-----------+----------------------------------+
| --clusterdim        | Integer   | 可选，默认20。用于               |
|                     |           | PCA降维后的                      |
|                     |           | 降维聚类使用的显著主成分的数量。 |
+---------------------+-----------+----------------------------------+
| --doubletpercentage | Float     | 可选，默认 :                     |
|                     |           | 0.05。预测双胞比例。             |
+---------------------+-----------+----------------------------------+
| --mitpercentage     | Integer   | 可选，默认 :                     |
|                     |           | 15。过滤线粒体基因比例。         |
+---------------------+-----------+----------------------------------+
| --minfeatures       | Integer   | 可选，默认 : 200。               |
|                     |           | 细胞含有的基因数目的最小值 。    |
+---------------------+-----------+----------------------------------+
| --PCusage           | Integer   | 可选，默认 : 50。 用于           |
|                     |           | PCA降维的主成分的数量。          |
+---------------------+-----------+----------------------------------+
| --resolution        | Integer   | 可选，默认 :                     |
|                     |           | 0.5。细胞聚类分辨率。            |
|                     |           | 该参数设置下游聚类的细胞群       |
|                     |           | 体数量，增加该值导致更多的分群。 |
+---------------------+-----------+----------------------------------+

.. _124-dnbc4tools-report:

1.2.4 DNBC4tools report
^^^^^^^^^^^^^^^^^^^^^^^

数据汇总和可视化网页报告生成。

参数保持与DNBC4tools run一致。

**Notice**\ ：在data,count,analysis,report中有些参数在主程序
run中没有。通常情况下这些参数使用默认值分析即可。如果需要修改这些参数，可使用
data,count,analysis,report模块进行分析，再使用 run
-process参数将后续的结果分析。例如，使用
run得到分析结果和报告后，对细胞分群的结果不满意，可使用 DNBC4tools
analysis –resolution调整分群的分辨率，分析完成后在使用 DNBC4tools run
–process report完成后续的 report分析。

.. _13-dnbc4tools-multi对多个样本生成dnbc4tools-run:

1.3 DNBC4tools multi(对多个样本生成DNBC4tools run)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

   # 分析示例

   DNBC4tools multi --list samplelist

   		--starIndexDir /database/Mouse/mm10/ --gtf /database/Mouse/mm10/genes.gtf \

   		--thread 10

其中samplelist的格式如下：

.. code:: bash

   test1 cDNA1_L01_1.fq.gz;cDNA1_L01_2.fq.gz    oligo1_L01_1.fq.gz,oligo1_L02_1.fq.gz;/oligo1_L01_2.fq.gz,oligo1_L02_2.fq.gz Mouse

   test2 cDNA2_L01_1.fq.gz,cDNA2_L02_1.fq.gz;cDNA1_L01_2.fq.gz,cDNA2_L02_2.fq.gz   oligo2_L01_1.fq.gz;/oligo2_L01_2.fq.gz  Mouse

   test3 cDNA3_L01_1.fq.gz;cDNA3_L01_2.fq.gz    oligo3_L01_1.fq.gz,oligo3_L02_1.fq.gz;/oligo3_L01_2.fq.gz,oligo3_L02_2.fq.gz Mouse

-  文件包含四列，使用水平制表符(\t)进行分隔

-  不设置表头，第一列为样本名称，第二列为cDNA文库信息，第三列为oligo文库信息，第四列为物种名称。

-  cDNA文库和oligo文库，多个fastq以逗号分隔，R1和R2以分号分隔。R1和R2中的多个fastq顺序需保持一致。

-  分析样本物种名须保持一致，因为\ ``--starIndexDir``\ 和\ ``--gtf``\ 只能填写一个物种。

.. _14-dnbc4tools-clean分析完成后清理中间文件:

1.4 DNBC4tools clean(分析完成后清理中间文件)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

对分析中的存储较大的中间文件进行清除。确定不需要对结果重新分析时使用 。

.. code:: bash

   ### 删除该目录下所有样本的中间大文件

   DNBC4tools clean

   

   ### 删除该目录下样本sampleA的中间大文件

   DNBC4tools clean --name sampleA

分析参数如下：

+-----------+-----------+--------------------------------------------+
| 参数      | 类型      | 描述                                       |
+===========+===========+============================================+
| --name    | String    | 可选，默认该目录下的所有样本。需要进行中   |
|           |           | 间文件清楚的样本名，多个样本使用逗号连接。 |
+-----------+-----------+--------------------------------------------+
| --outdir  | Directory | 可选，默认当前路径。分析结果的输出目录。   |
+-----------+-----------+--------------------------------------------+
| --combine | Flag      | 对选择的样本的统计文件                     |
|           |           | metrics_summary.xls进                      |
|           |           | 行合并且将样本的网页报告拷贝到result目录中 |
|           |           | 。                                         |
+-----------+-----------+--------------------------------------------+

.. _2-wdl流程:

2. WDL流程
----------
