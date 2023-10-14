---
title: 使用HISAT StringTie和Ballgown进行的转录本表达水平分析（中译）(三)
date: 2022-03-12 11:47:17
tags:
    - 生物信息学
    - 翻译文献
    - RNA-Seq
    - 协议Protocol
categories:
    - 生物信息学习
---
### 材料
##### 配置
- 数据（本方案中使用的RNA-seq reads、索引和基因注释的例子可在[链接](ftp://ftp.ccb.jhu.edu/pub/RNAseq_protocol)；详情见设备设置）
- HISAT2软件（[链接1](http://ccb.jhu.edu/software/hisat2)或[链接2](http://github.com/infphilo/hisat2)），2.0.1或更高版本
- StringTie软件([链接1](http://ccb.jhu.edu/software/stringtie)或[链接2](https://github.com/gpertea/stringtie),  1.2.2 或更高版本)
- SAMtools （[链接](http://samtools.sourceforge.net), 版本0.1.19或更高)
- R（[链接](https://www.r-project.org)，版本3.2.2或更高）。
- 硬件（64位计算机，运行Linux或Mac OS X（10.7 Lion或更高版本）；4GB内存（最好是8GB）；见**设备设置**)

##### 设备配置

###### 所需的数据  
该协议用一个来自智人X染色体的例子实验来说明，你可以通过分析来熟悉全套软件。你需要的所有数据都可以在文件**chrX_data.tar.gz**中找到，该文件将在下面说明。要使用HISAT2和StringTie进行基因表达分析，你必须使用一个基因组已被测序的生物体。这两个工具也可以使用基因和转录物的注释文件，尽管这是可选的。HISAT2需要一个基因组的索引，这在数据下载文件中提供。在HISAT2网站上可以找到其他多个物种的预建索引。在没有可用索引的情况下，**Box2**提供了如何创建一个HISAT2索引的说明。
> 重要提醒 协议中使用的命令都应该从Unix shell提示下的终端窗口运行，直到在R环境中进行差异表达式分析。我们鼓励用户创建一个单一的目录（例如，"my_rnaseq_exp"），在其中存储所有的例子数据和分析所创建的文件。所有的命令都是在假设用户是在这个目录下工作的情况下描述的。该协议还包括在R统计计算环境中运行的小部分代码。拟在Unix shell（如bash或csh）中执行的命令以"$"字符为前缀。要从R脚本或R交互式shell中运行的命令前缀为">"字符。


- Box2 | 创建HISAT2索引文件
  >人类基因组和许多模式生物的预建HISAT2索引可以从HISAT网站下载。如果所需的索引不在那里，用户可能需要建立一个自定义的索引。这里我们描述了如何为人类X号染色体建立索引；要为其他基因组建立索引，应该用对应于该基因组及其注释的文件分别替换基因组文件CHRX.FA和注释文件CHRX.gtf。
  首先，使用HISAT2软件包中的python脚本，从基因注释文件中提取剪接位点和外显子信息(Line:01,02)，然后再创建一个索引。
  ```
  $ extract_splice_sites.py chrX_data/genes/chrX.gtf >chrX.ss
  $ extract_exons.py chrX_data/genes/chrX.gtf >chrX.exon
  $ hisat2-build --ss chrX.ss --exon chrX.exon chrX_data/genome/chrX.fa chrX_tran
  ```
  > 如果没有注释，上述命令中的--ss和--exon选项可以省略。请注意，建立索引的步骤比对齐步骤需要更多的内存，而且可能无法在台式电脑上执行。例如，为X染色体建立索引需要9GB内存，为整个人类基因组建立索引需要160GB内存。如果省略注释信息，内存量要小得多。使用一个CPU核心对X染色体进行索引 需要<10分钟。使用8个CPU核为整个人类基因组建立索引应该需要2小时。


###### 下载和整理所需数据
解压数据文件并检查内容
``` 
$ tar xvzf chrX_data.tar.gz
```
假设我们将数据存储在my_rnaseq_exp/，软件包解压后包含一个chrX_data/文件夹，其结构如下：*samples/ indexes/ genome/ genes/*（包含四个目录）。

样本目录包含12个样本的成对端RNA-seq reads(*paired-end RNA-seq reads*)，其中6个样本来自两个人群，GBR（来自英格兰的英国人）和YRI（来自尼日利亚伊巴丹的约鲁巴人）。每个人群都有三个样本，分别来自男性和女性受试者（每个样本的人口和性别见**Table2**）。
- Table2|协议中使用的样本
  样本ID|性别|人群
  :---:|:---:|:---:
  ERR188245|Female|GBR
  ERR188428|Female|GBR
  ERR188337|Female|GBR
  ERR188401|Male|GBR
  ERR188257|Male|GBR
  ERR188383|Male|GBR
  ERR204916|Female|YRI
  ERR188234|Female|YRI
  ERR188273|Female|YRI
  ERR188454|Male|YRI
  ERR188104|Male|YRI
  ERR188044|Male|YRI



所有的序列都是压缩的'fastq'格式，它将每个read存储为四行。每个样本依次包含在两个文件中：一个是read 1，另一个是read 2。因此，例如，样本*ERR188245*包含在文件*ERR188245_ chrX_1.fastq.gz*和*ERR188245_chrX_2.fastq.gz*中，每个文件包含 *855,983*条reads。请注意，原始数据文件包含映射到整个基因组的*reads*，而不仅仅是X染色体，这些文件大约要大25倍。如果读者有兴趣，可以在[链接](ftp://ftp.ccb.jhu.edu/pub/RNAseq_protocol)下的README文件查看关于下载完整数据集的说明。请注意，HISAT2还提供了一个选项，称为-sra-acc，可以通过互联网直接使用**NCBI序列**阅读档案（SRA）数据。这消除了手动下载*SRA reads*并将其转换为fasta/fastq格式的需要，却不会显著影响到运行时间。例如，下面的HISAT2命令将下载并对齐本协议中使用的12个样本中的1个（*ERR188245*）。
```
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran --sra-acc ERR188245 –S ERR188245_chrX.sam
```
indexes目录包含X染色体的HISAT2索引，包括*chrX_tran.1.ht2, chrX_tran.2.ht2, chrX_tran.3.ht2, chrX_tran.4.ht2
chrX_tran.5.ht2, chrX_tran.6.ht2, chrX_tran.7.ht2 和 chrX_tran.8.ht2*.

基因组目录包含一个文件*chrX.fa*，它是人类X染色体的序列（*GRCh38 build 81*）。注意，如果你使用的是全基因组，这个目录将有一个包含所有染色体的文件。基因目录包含一个 *chrX.gtf*，它存储你了来自RefSeq数据库的GRCh38的人类基因注释。
chrX_data目录还包含另外两个文件：*mergelist.txt*和*geuvadis_phenodata.csv*。这些文件是运行这个协议必须的文本文件，如下文所述。通常情况下，你会自己用文本编辑器创建这些文件，但我们在这里提供它们作为例子。

###### 下载安装软件
先创建一个目录来存储本协议中使用的所有的程序（如果还没有）
```
$ mkdir $HOME/bin
```
将上述目录添加到你的PATH环境变量中
```
$ export PATH=$HOME/bin:$PATH
```
要安装SAMtools，请从[链接](http://samtools.sourceforge.net)下载，解压*SAMtools tar*文件，然后cd到SAMtools源目录（该协议将适用于从0.1.19开始的所有SAMtools版本）。
```
$ tar jxvf samtools-0.1.19.tar.bz2
$ cd samtools-0.1.19
$ make
$ cd .
// Copy the samtools binary to a directory in your PATH:
$ cp samtools-0.1.19/samtools $HOME/bin
```
要安装HISAT2，请从[链接1](http://ccb.jhu.edu/software/hisat2)或[链接2](http://github.com/infphilo/hisat2)，下载最新的二进制程序包，解压HISAT2的压缩包，并cd到解压后的目录。
```
$ unzip hisat2-2.0.1-beta-OSX_x86_64.zip
```
接下来，将HISAT2的可执行文件复制到你的PATH中的一个目录。
```
$ cp hisat2-2.0.1-beta/hisat2* hisat2-2.0.1-beta/*.py $HOME/bin
```
要安装StringTie，请从[链接](http://ccb.jhu.edu/software/stringtie)，下载最新的二进制包，解压StringTie tar文件，然后cd到解压后的目录。
```
$ tar xvzf stringtie-1.2.2.OSX_x86_64.tar.gz
```
(注意，这个指令使用了Mac OS X的可执行文件。如果你使用的是Linux，可以用这个可执行文件代替：stringtie-1.2.2.Linux_x86_64.tar.gz）。接下来，将StringTie包的可执行文件复制到你的PATH中的某个目录。
```
$ cp stringtie-1.2.2.OSX_x86_64/stringtie $HOME/bin
```
StringTie也可以从[链接](https://github.com/gpertea/stringtie)下载。要安装*gffcompare*，请从以下网站下载最新的二进制软件包[链接](http://github.com/gpertea/gffcompare)，并按照 *README.md*文件中的说明。要安装Ballgown，请先启动R语言会话界面。
```
$ R
R version 3.2.2 (2015-08-14) -- "Fire Safety"
Copyright (C) 2015 The R Foundation for
Statistical Computing
Platform: x86_64-apple-darwin13.4.0 (64-bit)
```
'>'前的文本为R的介绍信息结束。安装 **Ballgown**软件包和其他R的依赖项，如下所示。
```
>install.packages("devtools",repos="http://cran.us.r-project.org")
>source("http://www.bioconductor.org/biocLite.R")
>biocLite(c("alyssafrazee/RSkittleBrewer","ballgown","genefilter","dplyr","devtools"))
```
运行本协议需要Bioconductor 3.0或更高版本，以及R 3.1版本。补充软件包括一个shell脚本和其他依赖文件，可以用来运行本协议的所有步骤。还包括一个README文件，解释了如何运行该脚本。这些文件也可以从[链接](ftp://ftp.ccb.jhu.edu/pub/RNAseq_protocol)获取。