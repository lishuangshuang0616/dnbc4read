=========
软件安装
=========
DNBelab_C_Series_HT_scRNA-analysis-software分析软件基于conda环境运行


安装需求
===================
 - Language Python3(>=3.8.*),R scripts
 - Hardware/Software requirements
 - x86-64 compatible processors.
 - require at least 50GB of RAM and 4 CPU.
 - centos 7.x 64-bit operating system (Linux kernel 3.10.0, compatible with higher software and hardware configuration).


安装conda
===================
Conda是一个开源的软件包管理系统和环境管理系统，可在Windows，macOS和Linux上运行。Conda 可快速安装、运行和更新包及其依赖项。Conda 可轻松创建、保存、加载本地计算机上的环境并在环境之间切换。它是为Python程序创建的，但它可以为任何语言打包和分发软件。

Conda作为软件包管理器可以帮助您查找和安装软件包。如果您需要需要不同版本的Python的包，则无需切换到其他环境管理器，因为conda也是环境管理器。只需几个命令，您就可以设置一个完全独立的环境来运行不同版本的Python，同时继续在正常环境中运行通常版本的Python。

.. csv-table:: Miniconda3 of Linux
   :header: Python version,Name,Size,SHA256 hash
   :widths: 5, 10, 5, 80

   Python 3.9,`Miniconda3 Linux 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py39_4.12.0-Linux-x86_64.sh>`_,73.1 MiB,``78f39f9bae971ec1ae7969f0516017f2413f17796670f7040725dd83fcff5689``
   ,`Miniconda3 Linux-aarch64 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py39_4.12.0-Linux-aarch64.sh>`_,75.3 MiB,``5f4f865812101fdc747cea5b820806f678bb50fe0a61f19dc8aa369c52c4e513``
   ,`Miniconda3 Linux-ppc64le 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py39_4.12.0-Linux-ppc64le.sh>`_,74.3 MiB,``1fe3305d0ccc9e55b336b051ae12d82f33af408af4b560625674fa7ad915102b``
   ,`Miniconda3 Linux-s390x 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py39_4.12.0-Linux-s390x.sh>`_,69.2 MiB,``ff6fdad3068ab5b15939c6f422ac329fa005d56ee0876c985e22e622d930e424``
   Python 3.8,`Miniconda3 Linux 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py38_4.12.0-Linux-x86_64.sh>`__,72.6 MiB,``3190da6626f86eee8abf1b2fd7a5af492994eb2667357ee4243975cdbb175d7a``
   ,`Miniconda3 Linux-aarch64 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py38_4.12.0-Linux-aarch64.sh>`__,64.4 MiB,``0c20f121dc4c8010032d64f8e9b27d79e52d28355eb8d7972eafc90652387777``
   ,`Miniconda3 Linux-ppc64le 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py38_4.12.0-Linux-ppc64le.sh>`__,65.9 MiB,``4be4086710845d10a8911856e9aea706c1464051a24c19aabf7f6e1a1aedf454``
   ,`Miniconda3 Linux-s390x 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py38_4.12.0-Linux-s390x.sh>`__,68.7 MiB,``3125961430c77eae81556fa59fe25dca9e5808f76c05f87092d6f2d57f85e933``
   Python 3.7,`Miniconda3 Linux 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py37_4.12.0-Linux-x86_64.sh>`__,100.1 MiB,``4dc4214839c60b2f5eb3efbdee1ef5d9b45e74f2c09fcae6c8934a13f36ffc3e``
   ,`Miniconda3 Linux-aarch64 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py37_4.12.0-Linux-aarch64.sh>`__,101.7 MiB,``47affd9577889f80197aadbdf1198b04a41528421aaf0ec1f28b04a50b9f3ab8``
   ,`Miniconda3 Linux-ppc64le 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py37_4.12.0-Linux-ppc64le.sh>`__,101.4 MiB,``c99b66a726a5116f7c825f9535de45fcac9e4e8ae825428abfb190f7748a5fd0``
   ,`Miniconda3 Linux-s390x 64-bit <https://repo.anaconda.com/miniconda/Miniconda3-py37_4.12.0-Linux-s390x.sh>`__,95.9 MiB,``8401eb61094297cc53709fec4654695d59652b3adde241963d3d993a6d760ed5``

