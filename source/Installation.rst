软件安装
========

DNBelab_C_Series_HT_scRNA-analysis-software分析软件可基于conda环境、docker/singularity容器。

安装需求
--------

-  Language Python3(>=3.8.*),R scripts

-  x86-64 compatible processors.

-  require at least 50GB of RAM and 4 CPU.

-  centos 7.x 64-bit operating system (Linux kernel 3.10.0, compatible
   with higher software and hardware configuration).

1. Conda
-----

.. _1-conda安装:

1.1 Conda安装
~~~~~~~~~~~~

Conda是一个开源的软件包管理系统和环境管理系统，可在Windows，macOS和Linux上运行。Conda
可快速安装、运行和更新包及其依赖项。Conda
可轻松创建、保存、加载本地计算机上的环境并在环境之间切换。它是为Python程序创建的，但它可以为任何语言打包和分发软件。

.. code:: bash

   ### 下载miniconda3安装包
   wget https://repo.anaconda.com/miniconda/Miniconda3-py38_4.12.0-Linux-x86_64.sh -O Miniconda3-latest-Linux-x86_64.sh
   ### 安装miniconda3
   ## Regular installation
   bash Miniconda3-latest-Linux-x86_64.sh
   ## Installing in silent mode
   bash Miniconda3-latest-Linux-x86_64.sh -b -p $HOME/miniconda

.. _2-dnbc4tools环境安装:

1.2 dnbc4tools环境安装
~~~~~~~~~~~~~~~~~~~~~

.. _21-clone-repo:

1.2.1 Clone repo
^^^^^^^^^^^^^^^

`github <https://github.com/MGI-tech-bioinformatics/DNBelab_C_Series_HT_scRNA-analysis-software>`__\ 地址，下载

.. code:: bash

   git clone https://github.com/MGI-tech-bioinformatics/DNBelab_C_Series_HT_scRNA-analysis-software.git
   chmod 755 -R DNBelab_C_Series_HT_scRNA-analysis-software
   cd DNBelab_C_Series_HT_scRNA-analysis-software

.. _22-create-dnbc4tools-environment:

1.2.2 Create DNBC4tools environment
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

基于conda创建dnbc4tools环境

.. code:: bash

   source /mgi/miniconda3/bin/activate
   conda env create -f DNBC4tools.yaml -n DNBC4tools
   Rscript -e "devtools::install_github(c('chris-mcginnis-ucsf/DoubletFinder','ggjlab/scHCL','ggjlab/scMCA'),force = TRUE);"

如果使用wdl去运行流程需要下载cromwell

.. code:: bash

   wget https://github.com/broadinstitute/cromwell/releases/download/35/cromwell-35.jar

.. _3-更新:

1.3 更新
~~~~~~~

在版本更新时，不需要对环境进行重新安装，只需要更新python包dnbc4tools

.. code:: bash

   # 如果只使用命令行模式，只需要更新dnbc4tools环境的dnbc4tools
   source activate DNBC4tools
   pip install --upgrade -i https://pypi.tuna.tsinghua.edu.cn/simple dnbc4tools
   # 如果还需要使用wdl，则还需要重新更新repo
   git clone https://github.com/MGI-tech-bioinformatics/DNBelab_C_Series_HT_scRNA-analysis-software.git
   chmod 755 -R DNBelab_C_Series_HT_scRNA-analysis-software

2. 基于容器技术
------------

.. _1-docker:

2.1 docker
~~~~~~~~~

Docker
是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的镜像中，然后发布到任何流行的
Linux或Windows操作系统的机器上，也可以实现虚拟化。

服务器安装docker可参考\ `Install Docker Engine \| Docker
Documentation <https://docs.docker.com/engine/install/>`__

.. code:: bash

   # 下载docker镜像
   docker pull lishuangshuang3/dnbc4tools

.. _2-singularity:

2.2 singularity
~~~~~~~~~~~~~~

singularity 是一个容器平台。 Singularity
旨在以简单、可移植和可重现的方式在 HPC 集群上运行复杂的应用程序。

singularity安装可参考\ https://docs.sylabs.io/guides/3.5/user-guide/introduction.html

.. code:: bash

   # 创建sif文件
   singularity build dnbc4tools.sif docker://lishuangshuang3/dnbc4tools

