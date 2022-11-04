使用说明
========

软件安装完成后可使用命令行或者WDL流程进行分析。

conda环境和docker、singularity是基于命令行模式。

WDL流程只有主流程分析功能，不能进行数据库构建等其他功能。

命令行DNBC4tools
----------------

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

.. _1-dnbc4tools-mkref-构建数据库:

1. DNBC4tools mkref (构建数据库)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

分析需要比对参考基因组和注释文件注释分析。在分析前需要创建对应物种的参考基因组数据库。需要准备两个文件，基因组的DNA序列文件（FASTA格式）和基因的注释文件（GTF格式）。常用的Ensembl和GENECODE数据库提供了这两种格式的文件。

.. _11-查看注释文件的基因类型:

1.1 查看注释文件的基因类型
^^^^^^^^^^^^^^^^^^^^^^^^^^

在构建数据库之前可以对gtf文件的基因类型进行过滤，先查看gtf中的gene类型确定需要过滤哪些基因类型。

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

.. _12-过滤注释文件:

1.2 过滤注释文件
^^^^^^^^^^^^^^^^

在构建数据库之前可以对gtf文件的基因类型进行过滤，使其中仅包含感兴趣的基因类别，过滤哪些基因取决于您的研究问题。

软件分析中，gtf中存在overlap的基因将导致reads被舍弃。通过过滤gtf文件使其只有少量重叠的基因。

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

.. _13-构建数据库:

1.3 构建数据库
^^^^^^^^^^^^^^

使用比对软件scStar进行数据库的构建。scStar的STAR版本为
2.7.2b，基因组版本为2.7.1a，相同基因组版本的STAR构建的数据库可通用，不同的基因组版本不可互用
。数据库不向下兼容 v1版本的数据库。

.. code:: bash

   DNBC4tools mkref --action mkref --ingtf gene.filter.gtf \
               --fasta genome.fa \
               --genomeDir $star_dir \
               --thread $threads

-  DNBC4tools mkref --action mkref 输入项：

+--------------------------+-----------+---------------------------+
| 参数                     | 类型      | 描述                      |
+==========================+===========+===========================+
| --ingtf                  | File Path | 输入                      |
|                          |           | 构建star数据库的gtf文件。 |
+--------------------------+-----------+---------------------------+
| --fasta                  | File Path | 输入与                    |
|                          |           | gtf文件配套的参考基因组。 |
+--------------------------+-----------+---------------------------+
| --genomeDir              | Directory | 构建数据库的结果目录。    |
+--------------------------+-----------+---------------------------+
| --thread                 | Integer   | 程序运行时                |
|                          |           | 所调用的进程数，默认为4。 |
+--------------------------+-----------+---------------------------+
| --limitGenomeGenerateRAM | Integer   | 程序运行时所调用的内存大  |
|                          |           | 小，默认为125000000000。  |
+--------------------------+-----------+---------------------------+

.. _14-构建数据库参考文件:

1.4 构建数据库参考文件
^^^^^^^^^^^^^^^^^^^^^^

**Ref-202203**

-  **Human(GRCh38)**

   .. code:: bash

      http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/GRCh38.primary_assembly.genome.fa.gz
      http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/gencode.v32.primary_assembly.annotation.gtf.gz

-  **Mouse(GRCm38)**

   .. code:: bash

      http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/GRCm38.primary_assembly.genome.fa.gz
      http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/gencode.vM23.primary_assembly.annotation.gtf.gz

.. _2-dnbc4tools-run-运行主程序:

2. DNBC4tools run (运行主程序)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

run命令为运行主程序

.. code:: bash

   # 主程序示例
   DNBC4tools run --cDNAfastq1 cDNA_R1.fastq.gz \
   		--cDNAfastq2 cDNA_R2.fastq.gz \
   		--oligofastq1 oligo1_1.fq.gz,oligo2_1.fq.gz \
   		--oligofastq2 oligo1_2.fq.gz,oligo2_2.fq.gz \
   		--genomeDir /database/Mouse/mm10/ --gtf /database/Mouse/mm10/genes.gtf \
   		--name test --species Mus_musculus --thread 10

分析参数如下：

-  必选参数

+---------------+-----------+----------------------------------------+
| 参数          | 类型      | 描述                                   |
+===============+===========+========================================+
| --name        | String    | 样本名称。                             |
+---------------+-----------+----------------------------------------+
| --cDNAfastq1  | File Path | cDNA文库fastq                          |
|               |           | 格式的R1端序列，多个文件使用逗号隔开。 |
+---------------+-----------+----------------------------------------+
| --cDNAfastq2  | File Path | cDNA文库fastq格式的R2端序列，多个      |
|               |           | 文件使用逗号隔开，顺序与cDNAfastq1相同 |
|               |           | 。                                     |
+---------------+-----------+----------------------------------------+
| --oligofastq1 | File Path | oligo文库fastq                         |
|               |           | 格式的R1端序列，多个文件使用逗号隔开。 |
+---------------+-----------+----------------------------------------+
| --oligofastq2 | File Path | oligo文库fastq格式的R2端序列，多个文件 |
|               |           | 使用逗号隔开，顺序与oligofastq1相同。  |
+---------------+-----------+----------------------------------------+
| --genomeDir   | Directory | 参考基因组构建数据库索引路径。         |
+---------------+-----------+----------------------------------------+
| --gtf         | File Path | 参考基因组注释文件gtf路径。            |
+---------------+-----------+----------------------------------------+

-  可选参数

+------------------+-----------+-------------------------------------+
| 参数             | 类型      | 描述                                |
+==================+===========+=====================================+
| --species        | String    | 样本物种名称，默认为undefined。只   |
|                  |           | 有物种名为Homo_sapiens,Mus_musculu  |
|                  |           | s,Human,Mouse时可进行细胞注释分析。 |
+------------------+-----------+-------------------------------------+
| --outdir         | Directory | 分析结果路径，默认为当前路径 。     |
+------------------+-----------+-------------------------------------+
| --thread         | Integer   | 程序运行时调用的进程数，默认为4。   |
+------------------+-----------+-------------------------------------+
| --calling_method | String    | 默认：emptydrops。cell              |
|                  |           | calling鉴定有效液滴内beads的        |
|                  |           | 方法，可选barcoderanks,emptydrops。 |
+------------------+-----------+-------------------------------------+
| --expectcells    | Integer   | 默认：3000。期望细胞数，仅当calling |
|                  |           | method为emptydrops时参数有效        |
|                  |           | 。期望细胞数建议按照投入活细胞数量  |
|                  |           | 的50%去设置（细胞捕获率约为50%）。  |
+------------------+-----------+-------------------------------------+
| --forcecells     | Integer   | 默认：0。截取beads数量进行分析。    |
+------------------+-----------+-------------------------------------+
| --chemistry      | String    | 默认：aut                           |
|                  |           | o。试剂版本，建议自动获取试剂版本。 |
|                  |           | 该参数需要和--darkreaction一起使用  |
|                  |           | 。试剂版本包括scRNAv1HT,scRNAv2HT。 |
+------------------+-----------+-------------------------------------+
| --darkreaction   | String    | 默认                                |
|                  |           | ：auto。暗反应设置，建议自动获取测  |
|                  |           | 序是否使用暗反应。该参数需要和--che |
|                  |           | mistry一起使用。参数格式为”cDNA,oli |
|                  |           | go“，中间使用逗号分隔，比如"R1,R1R2 |
|                  |           | "代表cDNA的R1使用了暗反应，oligo的R |
|                  |           | 1和R2都使用了暗反应。包括"R1,R1R2", |
|                  |           | "R1,R1", "unset,unset"等等。        |
+------------------+-----------+-------------------------------------+
| --customize      | String    | 使用自定义的文库结构文件进          |
|                  |           | 行分析。文件为json格式，包含结构位  |
|                  |           | 置、汉明距离允许错配碱基数量和cell  |
|                  |           | barcode白名单信息。                 |
|                  |           | 参数格式为”cDNA,oligo“，比如”scRNA  |
|                  |           | _beads_readStructure.json,scRNA_oli |
|                  |           | go_readStructure.json“。customize的 |
|                  |           | 优先级高于chemistry和darkreaction。 |
+------------------+-----------+-------------------------------------+
| --process        | String    | 默认：data,count,anlysis            |
|                  |           | ,report。选择需要分析的步骤，可选择 |
|                  |           | data,count,anlysis,report其中几项（ |
|                  |           | 该参数常用于已分析完成需要重新调整  |
|                  |           | 参数时使用，更改某一步骤参数后面的  |
|                  |           | 步骤也需要重新分析），用逗号分隔。  |
+------------------+-----------+-------------------------------------+
| --mtgenes        | String    | 默认：auto。线粒                    |
|                  |           | 体基因列表文件，auto表示选择基因名  |
|                  |           | 前缀为mt或MT的基因作为线粒体基因。  |
+------------------+-----------+-------------------------------------+

-  flag参数

+--------------+------+----------------------------------------------+
| 参数         | 类型 | 描述                                         |
+==============+======+==============================================+
| --no_introns | Flag | 比对到                                       |
|              |      | intronic区域的reads不纳入进表达量矩阵计算。  |
+--------------+------+----------------------------------------------+
| --no_bam     | Flag | 添加该参数则不会将02.count中                 |
|              |      | 的anno_decon_sorted.bam和anno_decon_sorted.b |
|              |      | am.bai移动到output目录中。后续使用DNBC4tools |
|              |      | clean时则会删除该bam文件减少存储占用。       |
+--------------+------+----------------------------------------------+
| --dry        | Flag | 不进行流程分析。只打印分析步骤的shell文件。  |
+--------------+------+----------------------------------------------+

对参数的详细说明：

-  ``--chemistry``\ 和\ ``--darkreaction``\ 需要一起使用。建议使用自动检测的试剂版本和测序暗反应。当测序时R1没有进行暗反应才可检测到软件版本，scRNAv1HT,scRNAv2HT的cDNA和oligo的R1端在有暗反应的情况下结构是一样的。

-  ``--customize``\ ，在使用customize参数时，chemistry和darkreaction参数是无法起作用的。json文件的格式内容可参考常见问题说明。

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

-  ``--species``\ ，信息会展示在结果报告中，如果信息为Homo_sapiens,Mus_musculus,Human,Mouse会进行细胞群体注释分析。

-  ``--process``\ ，默认：data,count,anlysis,report。选择需要分析的步骤，可选择data,count,anlysis,report其中几个步骤。

每个步骤可单独使用DNBC4tools去分析，具体见以下步骤。

**DNBC4tools data**

提取 barcode和 UMI序列，并对下机数据进行质控与参考基因组进行比对注释
，获取所有 beads的原始表达量矩阵 。

参数保持与DNBC4tools run一致。

**DNBC4tools count**

确定有效液滴内 beads，合并同一个液滴内的多个 beads计算细胞表达矩阵。

分析参数如下：

+---------------------+-----------+----------------------------------+
| 参数                | 类型      | 描述                             |
+=====================+===========+==================================+
| --bam               | File Path | 必选                             |
|                     |           | ，data步骤生成的final.bam文件。  |
+---------------------+-----------+----------------------------------+
| --raw_matrix        | Directory | 必选，da                         |
|                     |           | ta步骤生成的raw_matrix矩阵目录。 |
+---------------------+-----------+----------------------------------+
| --cDNAbarcodeCount  | File Path | 必选，data步骤生成的c            |
|                     |           | DNA_barcode_counts_raw.txt文件。 |
+---------------------+-----------+----------------------------------+
| --Indexreads        | File Path | 必选，data步                     |
|                     |           | 骤生成的Index_reads.fq.gz文件。  |
+---------------------+-----------+----------------------------------+
| --oligobarcodeCount | File Path | 必选，data步骤生成的Ind          |
|                     |           | ex_barcode_counts_raw.txt.文件。 |
+---------------------+-----------+----------------------------------+
| --minumi            | Integer   | 可选，默认1000。cell             |
|                     |           | calling中emptydrops方            |
|                     |           | 法中可获取的beads最小的umi数量。 |
+---------------------+-----------+----------------------------------+

**DNBC4tools analysis**

对细胞表达矩阵进行质控，过滤低质量的细胞根据表达矩阵进行细胞聚类分析和标记基因筛选。

分析参数如下：

+---------------------+-----------+----------------------------------+
| 参数                | 类型      | 描述                             |
+=====================+===========+==================================+
| --matrix            | Directory | 必选，count步骤生成              |
|                     |           | 的filter_matrix表达量矩阵目录。  |
+---------------------+-----------+----------------------------------+
| --qcdim             | String    | 可选，默认20。DoubletFin         |
|                     |           | der的PCs参数显著的主成分的数量。 |
+---------------------+-----------+----------------------------------+
| --clusterdim        | Integer   | 可选，默认20。用于PCA降维后的    |
|                     |           | 降维聚类使用的显著主成分的数量。 |
+---------------------+-----------+----------------------------------+
| --doubletpercentage | Float     | 可选，默认：0.05。预测双胞比例。 |
+---------------------+-----------+----------------------------------+
| --mitpercentage     | Integer   | 可选                             |
|                     |           | ，默认：15。过滤线粒体基因比例。 |
+---------------------+-----------+----------------------------------+
| --minfeatures       | Integer   | 可选，默认：2                    |
|                     |           | 00。细胞含有的基因数目的最小值。 |
+---------------------+-----------+----------------------------------+
| --PCusage           | Integer   | 可选，默认：                     |
|                     |           | 50。用于PCA降维的主成分的数量。  |
+---------------------+-----------+----------------------------------+
| --resolution        | Integer   | 可选，默认：0.5。细胞聚类分      |
|                     |           | 辨率。该参数设置下游聚类的细胞群 |
|                     |           | 体数量，增加该值导致更多的分群。 |
+---------------------+-----------+----------------------------------+

**DNBC4tools report**

数据汇总和可视化网页报告生成。

参数保持与DNBC4tools run一致。

**Notice**\ ：在data,count,analysis,report中有些参数在主程序
run中没有。通常情况下这些参数使用默认值分析即可。如果需要修改这些参数，可使用
data,count,analysis,report模块进行分析，再使用 run
-process参数将后续的结果分析。例如，使用
run得到分析结果和报告后，对细胞分群的结果不满意，可使用 DNBC4tools
analysis –resolution调整分群的分辨率，分析完成后在使用 DNBC4tools run
–process report完成后续的 report分析。

.. _3-dnbc4tools-multi-对多个样本生成dnbc4tools-run:

3. DNBC4tools multi (对多个样本生成DNBC4tools run)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: bash

   # 分析示例
   DNBC4tools multi --list samplelist
   		--genomeDir /database/Mouse/mm10/ --gtf /database/Mouse/mm10/genes.gtf \
   		--thread 10

其中samplelist的格式如下：

.. code:: bash

   test1 cDNA1_L01_1.fq.gz;cDNA1_L01_2.fq.gz    oligo1_L01_1.fq.gz,oligo1_L02_1.fq.gz;/oligo1_L01_2.fq.gz,oligo1_L02_2.fq.gz Mouse
   test2 cDNA2_L01_1.fq.gz,cDNA2_L02_1.fq.gz;cDNA1_L01_2.fq.gz,cDNA2_L02_2.fq.gz   oligo2_L01_1.fq.gz;/oligo2_L01_2.fq.gz  Mouse
   test3 cDNA3_L01_1.fq.gz;cDNA3_L01_2.fq.gz    oligo3_L01_1.fq.gz,oligo3_L02_1.fq.gz;/oligo3_L01_2.fq.gz,oligo3_L02_2.fq.gz Mouse

-  文件包含四列，使用水平制表符(\t)进行分隔

-  不设置表头，第一列为样本名称，第二列为cDNA文库信息，第三列为oligo文库信息，第四列为物种名称。

-  cDNA文库和oligo文库，多个fastq以逗号分隔，R1和R2以分号分隔。R1和R2中的多个fastq顺序需保持一致。

-  分析样本物种名须保持一致，因为\ ``--genomeDir``\ 和\ ``--gtf``\ 只能分析一个物种。

.. _4-dnbc4tools-clean-分析完成后清理中间文件:

4. DNBC4tools clean (分析完成后清理中间文件)
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
| --combine | Flag      | 对                                         |
|           |           | 选择的样本的统计文件metrics_summary.xls进  |
|           |           | 行合并且将样本的网页报告拷贝到result目录中 |
|           |           | 。                                         |
+-----------+-----------+--------------------------------------------+

WDL流程
-------

工作流描述语言（Workflow Description
Language），简称WDL，是一门开源的、标准化的以及人类可读写的用于描述任务和工作流的编程语言。

.. _1-准备配置文件:

1. 准备配置文件
~~~~~~~~~~~~~~~

config.json文件包含一下内容，可拷贝/example/wdl/config.json文件然后修改：

-  必选参数

+-------------------+-----------+------------------------------------+
| 参数              | 类型      | 描述                               |
+===================+===========+====================================+
| main.Outdir       | Directory | 输出结果目录的路径 。              |
+-------------------+-----------+------------------------------------+
| main.SampleName   | String    | 样本名，不允许有空格。             |
+-------------------+-----------+------------------------------------+
| main.cDNA_Fastq1  | Fastq     | cDNA文库fastq格式                  |
|                   |           | 的R1端序列，多个文件使用逗号隔开。 |
+-------------------+-----------+------------------------------------+
| main.cDNA_Fastq2  | Fastq     | cDNA文库fas                        |
|                   |           | tq格式的R2端序列，多个文件使用逗号 |
|                   |           | 隔开，顺序与main.cDNA_Fastq1相同。 |
+-------------------+-----------+------------------------------------+
| main.Oligo_Fastq1 | Fastq     | oligo文库fastq格式                 |
|                   |           | 的R1端序列，多个文件使用逗号隔开。 |
+-------------------+-----------+------------------------------------+
| main.Oligo_Fastq2 | Fastq     | oligo文库fas                       |
|                   |           | tq格式的R2端序列，多个文件使用逗号 |
|                   |           | 隔开，顺序与main.oligo_Fastq1相同  |
|                   |           | 。                                 |
+-------------------+-----------+------------------------------------+
| main.BeadsBarcode | JSON file | cDNA文库结构文                     |
|                   |           | 件路径，为json格式文件，包含结构位 |
|                   |           | 置、汉明距离允许错配碱基数量和cell |
|                   |           | barcode白名单信息。                |
+-------------------+-----------+------------------------------------+
| main.OligoBarcode | JSON file | oligo文库结构文                    |
|                   |           | 件路径，为json格式文件，包含结构位 |
|                   |           | 置、汉明距离允许错配碱基数量和cell |
|                   |           | barcode白名单信息。                |
+-------------------+-----------+------------------------------------+
| main.Root         | Directory | DNBelab C4分析流程路径 。          |
+-------------------+-----------+------------------------------------+
| main.Refdir       | Directory | 参考基因组构建数据库索引路径。     |
+-------------------+-----------+------------------------------------+
| main.Gtf          | File Path | 参考基因组注释文件gtf路径。        |
+-------------------+-----------+------------------------------------+
| main.Species      | String    | 样本物种名。                       |
+-------------------+-----------+------------------------------------+

-  可选参数

+-----------------------+-----------+---------------------------+
| 参数                  | 类型      | 描述                      |
+=======================+===========+===========================+
| main.expectCellNum    | Integer   | 默认：3000。期            |
|                       |           | 望细胞数，仅当calling_met |
|                       |           | hod为emptydrops时参数有效 |
|                       |           | 。                        |
+-----------------------+-----------+---------------------------+
| main.calling_method   | String    | 默认：emptydrops。cell    |
|                       |           | calling鉴定有效           |
|                       |           | 液滴内beads的方法，可选b  |
|                       |           | arcoderanks和emptydrops。 |
+-----------------------+-----------+---------------------------+
| main.forceCellNum     | Integer   | 默认：0。截取beads数量 。 |
+-----------------------+-----------+---------------------------+
| main.Intron           | Boolean   | 默认                      |
|                       |           | ：true。是否将比对到intr  |
|                       |           | onic区域的reads纳入分析。 |
+-----------------------+-----------+---------------------------+
| main.mtgenes          | String    | 默认：auto                |
|                       |           | 。线粒体基因列表文件，aut |
|                       |           | o表示选择基因名前缀为mt或 |
|                       |           | MT的基因作为线粒体基因。  |
+-----------------------+-----------+---------------------------+
| main.Oligo_type8      | File Path | 默认：D                   |
|                       |           | NBC4tools/config/oligo_ty |
|                       |           | pe8.txt。oligo文库droplet |
|                       |           | index白名单信息文件路径   |
|                       |           | 。                        |
+-----------------------+-----------+---------------------------+
| main.Adapter          | File Path | 默认：                    |
|                       |           | DNBC4tools/config/adapter |
|                       |           | .txt。Adapter列表文件路径 |
+-----------------------+-----------+---------------------------+
| main.clusterdim       | Integer   | 默认：20                  |
|                       |           | 。用于PCA降维后的降维聚类 |
|                       |           | 使用的显著主成分的数量。  |
+-----------------------+-----------+---------------------------+
| main.doublepercentage | Float     | 默                        |
|                       |           | 认：0.05。预测双胞比例。  |
+-----------------------+-----------+---------------------------+
| main.mitpercentage    | Integer   | 默认：                    |
|                       |           | 15。过滤线粒体基因比例。  |
+-----------------------+-----------+---------------------------+
| main.minfeatures      | Integer   | 默认：200。细             |
|                       |           | 胞含有的基因数目的最小值  |
+-----------------------+-----------+---------------------------+
| main.PCusage          | Integer   | 默认：50。                |
|                       |           | 用于PCA降维的主成分的数量 |
|                       |           | 。                        |
+-----------------------+-----------+---------------------------+
| main.resolution       | Integer   | 默认：0.5。细胞           |
|                       |           | 聚类分辨率。该参数设置下  |
|                       |           | 游聚类的细胞群体数量，增  |
|                       |           | 加该值能得到更多的分群。  |
+-----------------------+-----------+---------------------------+

准备主分析脚本run.sh：

.. code:: bash

   # export环境变量，将所有路径替换成真实分析路径
   export PATH=/miniconda3/envs/DNBC4tools/bin:$PATH
   export LD_LIBRARY_PATH=/miniconda3/envs/DNBC4tools/lib:$LD_LIBRARY_PATH
   java -jar /pipeline/wdl/cromwell-35.jar run -i config.json /pipeline/wdl/DNBC4_scRNA.wdl

针对多个样本，使用samplelist样本列表文件，参考\ *DNBC4tools
multi*\ ，修改scripts目录下的\ ``wdl.json``\ ，替换成真实路径，使用如下命令：

.. code:: bash

   /miniconda3/envs/DNBC4tools/bin/python creat_wdl_json.py --infile samplelist --outdir outdir

.. _2-运行分析流程:

2. 运行分析流程
~~~~~~~~~~~~~~~

.. code:: bash

   ### 运行分析流程
   sh run.sh

-  执行\ ``sh run.sh``\ 后，会在当前目录下生成 DNBelab
   C4分析的执行目录\ ``cromwell-executions``\ 该目录下为主进程\ ``main``\ 目录，在主进程\ ``main``\ 目录下包含
   workflow id，每次运行run.sh都会自动生成一个workflow id。每个 workflow
   id下包含4个功能模块对应的任务运行脚本及运行日志。

-  ``symbol``\ 记录每个功能步骤是否完成的标志文件。重新分析时如果symbol目录中存在该步骤完成标志文件，则该分析步骤跳过。

   -  ``01.oligoparse_sigh.txt``\ ，oligo文库数据质控过滤。

   -  ``02.cDNAAnno_sigh.txt``\ ，cDNA文库数据质控过滤、比对和注释生成所有beads的表达量矩阵。

   -  ``03.M280UMI_stat_sigh.txt``\ ，cell
      calling获取有效液滴内beads、合并同一液滴内beads。

   -  ``04.count_matrix_sigh.txt``\ ，生成细胞表达量矩阵。

   -  ``04.saturation_sigh.txt``\ ，饱和度分析。

   -  ``05.Cluster_sigh.txt``\ ，过滤后细胞降维聚类注释。

   -  ``05.QC_sigh.txt``\ ，对细胞进行过滤。

   -  ``06.splice_matrix_sigh.txt``\ ，exonic区域的表达量矩阵、RNA
      velocity分析的表达量矩阵。

   -  ``07.report_sigh.txt``\ ，分析结果整理，生成网页报告。
