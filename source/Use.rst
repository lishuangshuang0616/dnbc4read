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

-  | **Human(GRCh38)**
   | http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/GRCh38.primary_assembly.genome.fa.gz
   | http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_32/gencode.v32.primary_assembly.annotation.gtf.gz

-  | **Mouse(GRCm38)**
   | http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/GRCm38.primary_assembly.genome.fa.gz
   | http://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M23/gencode.vM23.primary_assembly.annotation.gtf.gz

.. _12-dnbc4tools-run-运行主程序:

1.2 DNBC4tools run (运行主程序)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
