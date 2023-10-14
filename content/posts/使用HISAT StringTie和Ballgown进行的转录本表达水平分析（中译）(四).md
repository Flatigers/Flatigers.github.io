---
title: 使用HISAT StringTie和Ballgown进行的转录本表达水平分析（中译）(四)
date: 2022-03-12 17:01:35
tags:
    - 生物信息学
    - 翻译文献
    - RNA-Seq
    - 协议Protocol
categories:
    - 生物信息学习
---

### 流程

##### 将RNA-seq reads与基因组进行比对 ● 时间\<20分钟

1.  将每个样本的reads映射到参考基因组上

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1
chrX_data/samples/ERR188044_chrX_1.fastq.gz -2
chrX_data/samples/ERR188044_chrX_2.fastq.gz -S ERR188044_chrX.sam
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1
chrX_data/samples/ERR188104_chrX_1.fastq.gz -2
chrX_data/samples/ERR188104_chrX_2.fastq.gz -S ERR188104_chrX.sam
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1 
chrX_data/samples/ERR188234_chrX_1.fastq.gz -2
chrX_data/samples/ERR188234_chrX_2.fastq.gz -S ERR188234_chrX.sam
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1
chrX_data/samples/ERR188245_chrX_1.fastq.gz -2
chrX_data/samples/ERR188245_chrX_2.fastq.gz -S ERR188245_chrX.sam
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1
chrX_data/samples/ERR188257_chrX_1.fastq.gz -2
chrX_data/samples/ERR188257_chrX_2.fastq.gz -S ERR188257_chrX.sam
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1
chrX_data/samples/ERR188273_chrX_1.fastq.gz -2
chrX_data/samples/ERR188273_chrX_2.fastq.gz -S ERR188273_chrX.sam
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1
chrX_data/samples/ERR188337_chrX_1.fastq.gz -2
chrX_data/samples/ERR188337_chrX_2.fastq.gz -S ERR188337_chrX.sam
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1
chrX_data/samples/ERR188383_chrX_1.fastq.gz -2
chrX_data/samples/ERR188383_chrX_2.fastq.gz -S ERR188383_chrX.sam
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1
chrX_data/samples/ERR188401_chrX_1.fastq.gz -2
chrX_data/samples/ERR188401_chrX_2.fastq.gz -S ERR188401_chrX.sam
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1
chrX_data/samples/ERR188428_chrX_1.fastq.gz -2
chrX_data/samples/ERR188428_chrX_2.fastq.gz -S ERR188428_chrX.sam
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1
chrX_data/samples/ERR188454_chrX_1.fastq.gz -2
chrX_data/samples/ERR188454_chrX_2.fastq.gz -S ERR188454_chrX.sam
$ hisat2 -p 8 --dta -x chrX_data/indexes/chrX_tran -1
chrX_data/samples/ERR204916_chrX_1.fastq.gz -2
chrX_data/samples/ERR204916_chrX_2.fastq.gz -S ERR204916_chrX.sam
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1.  将SAM文件分类并转换为BAM文件。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ samtools sort -@ 8 -o ERR188044_chrX.bam ERR188044_chrX.sam
$ samtools sort -@ 8 -o ERR188104_chrX.bam ERR188104_chrX.sam
$ samtools sort -@ 8 -o ERR188234_chrX.bam ERR188234_chrX.sam
$ samtools sort -@ 8 -o ERR188245_chrX.bam ERR188245_chrX.sam
$ samtools sort -@ 8 -o ERR188257_chrX.bam ERR188257_chrX.sam
$ samtools sort -@ 8 -o ERR188273_chrX.bam ERR188273_chrX.sam
$ samtools sort -@ 8 -o ERR188337_chrX.bam ERR188337_chrX.sam
$ samtools sort -@ 8 -o ERR188383_chrX.bam ERR188383_chrX.sam
$ samtools sort -@ 8 -o ERR188401_chrX.bam ERR188401_chrX.sam
$ samtools sort -@ 8 -o ERR188428_chrX.bam ERR188428_chrX.sam
$ samtools sort -@ 8 -o ERR188454_chrX.bam ERR188454_chrX.sam
$ samtools sort -@ 8 -o ERR204916_chrX.bam ERR204916_chrX.sam
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

>   上述命令适用于SAMtools
>   1.3版或更新的版本。对于旧版本的SAMtools，用户应该参考Box3。

\- Box3 \| 使用SAMtools 1.2.1版及以上版本对SAM文件进行分类和转换

>   最近版本的SAMtools可以在一个步骤中把SAM文件转换成BAM文件。对于使用旧版本的人来说，需要两个步骤。如图所示。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Convert SAM files to binary BAM files:

$ samtools view -bS ERR188044_chrX.sam >ERR188044_chrX_unsorted.bam
$ samtools view -bS ERR188104_chrX.sam >ERR188104_chrX_unsorted.bam
$ samtools view -bS ERR188234_chrX.sam >ERR188234_chrX_unsorted.bam
$ samtools view -bS ERR188245_chrX.sam >ERR188245_chrX_unsorted.bam
$ samtools view -bS ERR188257_chrX.sam >ERR188257_chrX_unsorted.bam
$ samtools view -bS ERR188273_chrX.sam >ERR188273_chrX_unsorted.bam
$ samtools view -bS ERR188337_chrX.sam >ERR188337_chrX_unsorted.bam
$ samtools view -bS ERR188383_chrX.sam >ERR188383_chrX_unsorted.bam
$ samtools view -bS ERR188401_chrX.sam >ERR188401_chrX_unsorted.bam
$ samtools view -bS ERR188428_chrX.sam >ERR188428_chrX_unsorted.bam
$ samtools view -bS ERR188454_chrX.sam >ERR188454_chrX_unsorted.bam
$ samtools view -bS ERR204916_chrX.sam >ERR204916_chrX_unsorted.bam

Sort the BAM files:

$ samtools sort -@ 8 ERR188044_chrX_unsorted.bam ERR188044_chrX
$ samtools sort -@ 8 ERR188104_chrX_unsorted.bam ERR188104_chrX
$ samtools sort -@ 8 ERR188234_chrX_unsorted.bam ERR188234_chrX
$ samtools sort -@ 8 ERR188245_chrX_unsorted.bam ERR188245_chrX
$ samtools sort -@ 8 ERR188257_chrX_unsorted.bam ERR188257_chrX
$ samtools sort -@ 8 ERR188273_chrX_unsorted.bam ERR188273_chrX
$ samtools sort -@ 8 ERR188337_chrX_unsorted.bam ERR188337_chrX
$ samtools sort -@ 8 ERR188383_chrX_unsorted.bam ERR188383_chrX
$ samtools sort -@ 8 ERR188401_chrX_unsorted.bam ERR188401_chrX
$ samtools sort -@ 8 ERR188428_chrX_unsorted.bam ERR188428_chrX
$ samtools sort -@ 8 ERR188454_chrX_unsorted.bam ERR188454_chrX
$ samtools sort -@ 8 ERR204916_chrX_unsorted.bam ERR204916_chrX

// Optionally, the user might want to delete temporary files:

$ rm *.sam *unsorted.bam 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

##### 组装和量化表达的基因和转录物 ● 时间\~15分钟

1.  为每个样本组装转录本

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR188044_chrX.gtf –l ERR188044 ERR188044_chrX.bam
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR188104_chrX.gtf –l ERR188104 ERR188104_chrX.bam
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR188234_chrX.gtf –l ERR188234 ERR188234_chrX.bam
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR188245_chrX.gtf –l ERR188245 ERR188245_chrX.bam
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR188257_chrX.gtf –l ERR188257 ERR188257_chrX.bam
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR188273_chrX.gtf –l ERR188273 ERR188273_chrX.bam
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR188337_chrX.gtf –l ERR188337 ERR188337_chrX.bam
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR188383_chrX.gtf –l ERR188383 ERR188383_chrX.bam
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR188401_chrX.gtf –l ERR188401 ERR188401_chrX.bam
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR188428_chrX.gtf –l ERR188428 ERR188428_chrX.bam
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR188454_chrX.gtf –l ERR188454 ERR188454_chrX.bam
$ stringtie -p 8 -G chrX_data/genes/chrX.gtf -o
ERR204916_chrX.gtf –l ERR204916 ERR204916_chrX.bam
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1.  合并所有样本的转录本

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ stringtie --merge -p 8 -G chrX_data/genes/chrX.gtf -o stringtie_merged.gtf chrX_data/mergelist.txt
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-   错误排除

>   这里mergelist.txt是一个文本文件，其中有上一步创建的基因转移格式（GTF）文件的名称，每个文件的名称在一行中（参见ChrX_data的例子文件）。如果你不在所有GTF文件所在的同一目录下运行StringTie，那么你可能需要在mergelist.txt中的每个GTF文件名中包括其完整路径。使用任何文本编辑器来创建或编辑（纯文本）mergelist.txt文件。

1.  检查转录本与参考注释的对比情况（可选）

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ gffcompare –r chrX_data/genes/chrX.gtf –G –o merged stringtie_merged.gtf
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

选项-o指定了gffcompare将创建的输出文件的前缀。上面的命令将生成多个文件，在*gffcompare*文档中作了解释；更多细节见**BOX1**。

1.  估计转录物丰度，并为Ballgown创建表格计数

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR188044/ERR188044_chrX.gtf ERR188044_chrX.bam
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR188104/ERR188104_chrX.gtf ERR188104_chrX.bam
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR188234/ERR188234_chrX.gtf ERR188234_chrX.bam
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR188245/ERR188245_chrX.gtf ERR188245_chrX.bam
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR188257/ERR188257_chrX.gtf ERR188257_chrX.bam
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR188273/ERR188273_chrX.gtf ERR188273_chrX.bam
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR188337/ERR188337_chrX.gtf ERR188337_chrX.bam
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR188383/ERR188383_chrX.gtf ERR188383_chrX.bam
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR188401/ERR188401_chrX.gtf ERR188401_chrX.bam
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR188428/ERR188428_chrX.gtf ERR188428_chrX.bam
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR188454/ERR188454_chrX.gtf ERR188454_chrX.bam
$ stringtie –e –B -p 8 -G stringtie_merged.gtf -o
ballgown/ERR204916/ERR204916_chrX.gtf ERR204916_chrX.bam
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

##### 运行差异化表达分析协议 ● 时间\~5分钟

1.  加载相关的R包。这些包包括你将用于执行大部分分析的Ballgown包，以及其他一些有助于这些分析的包，特别是**RSkittleBrewer**（用于设置颜色）、**genefilter**（用于快速计算平均值和变异值）、**dplyr**（用于排序和排列结果）和**devtools**（用于重现性和安装软件包）。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
$ R
R version 3.2.2 (2015-08-14) -- "Fire Safety"
Copyright (C) 2015 The R Foundation for Statistical Computing
Platform: x86_64-apple-darwin13.4.0 (64-bit)
>library(ballgown)
>library(RSkittleBrewer)
>library(genefilter)
>library(dplyr)
>library(devtools)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1.  加载样本的表型数据。一个名为geuvadis_phenodata.csv的示例文件包括在该协议的数据文件中（ChrX_data）。一般来说，你将不得不自己创建这个文件。它包含了关于你的RNA-seq样本的信息，格式如这个csv（逗号分隔的值）文件中所示。文件中的每一行包含每个样本的描述，每一列应该包含一个变量。为了将这个文件读入R，我们使用read.csv命令。在这个文件中，数值是由逗号分隔的，但如果文件是以制表符分隔的，你可以使用函数read.table。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>pheno_data = read.csv("geuvadis_phenodata.csv")
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-   错误排除

1.  读入由StringTie计算的表达数据。Ballgown目前也支持从Cufflinks6和RSEM17读取数据。要做到这一点，我们使用带有以下三个参数的ballgown命令：存储数据的目录（dataDir，这里简单命名为'Ballgown'），出现在样本名称中的模式（samplePattern）和我们在前一步加载的表型信息（pData）。请注意，一旦创建了Ballgown对象，任何其他Bioconductor32软件包都可以应用于数据分析或数据可视化。这里我们提出了一个标准化的流程，可以用来进行标准的差异表达分析。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>bg_chrX = ballgown(dataDir = "ballgown", samplePattern = "ERR", pData=pheno_data)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-   错误排除

1.  使用筛选器去除低丰度基因。RNA-seq数据的一个常见问题是，某些基因经常有非常少的计数或零计数。一个常见的步骤是过滤掉其中的一些。另一种被用于基因表达分析的方法是应用一个方差过滤器。在这里，我们删除所有跨样本方差小于1的转录本。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>bg_chrX_filt = subset(bg_chrX,"rowVars(texpr(bg_chrX)) >1",genomesubset=TRUE)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1.  识别在统计学上显示组间差异的转录本。我们可能想做的一件事是确保我们考虑到其他变量引起的表达变化。作为一个例子，我们将寻找在性别间有差异表达的转录本，同时校正由于群体变量造成的任何表达差异。我们可以使用Ballgown的stattest函数来做这件事。我们设置了**getFC=TRUE**参数，这样我们就可以查看两组之间经混杂因素调整后的折叠变化。
    请注意，Ballgown的统计测试是一个基于标准线性模型的比较。对于小样本量（每组n\<4），通常最好进行正则化。这可以用*Bioconductor*中的limma包来完成。其他正则化方法，如DESeq和edgeR可以应用于基因或外显子计数，但它们不适合直接应用于FPKM丰度估计。统计测试使用累积的上四分位数规范化(*cumulative
    upper quartile normalization*)。

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>results_transcripts = stattest(bg_chrX_filt,
feature="transcript",covariate="sex",adjustvars = c("population"), 
getFC=TRUE, meas="FPKM")
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1.  识别组间有统计学差异的基因。为此，我们可以运行与识别差异表达的转录本相同的函数，在这里我们在stattest命令中设置feature="gene"

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>results_genes = stattest(bg_chrX_filt, feature="gene",
covariate="sex", adjustvars = c("population"), getFC=TRUE,
meas="FPKM")
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1.  在results_transcripts数据框中添加基因名称和基因ID

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>results_transcripts =
data.frame(geneNames=ballgown::geneNames(bg_chrX_filt),
geneIDs=ballgown::geneIDs(bg_chrX_filt), results_transcripts)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1.  将结果从最小的P值排序到最大

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>results_transcripts = arrange(results_transcripts,pval)
>results_genes = arrange(results_genes,pval)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

-   两性之间差异表达的转录物（q值\<5%） 基因名\|基因ID\|转录本名\|ID\|Fold
    Change\|P值\|q值 :--:\|:--:\|:--:\|:--:\|:--:\|:--:\|:--: XIST
    MSTRG.506\|NR_001564\|1729\|0.003255\|7.0447e-10\|1.6160e-06
    \|MSTRG.506\|MSTRG.506.1\|1728\|0.016396\|1.2501e-08\|1.4339e-05
    TSIX\|MSTRG.505\|NR_003255\|1726\|0.083758\|2.4939e-06\|1.9070e-03
    \|MSTRG.506\|MSTRG.506.2\|1727\|0.047965\|3.7175e-06\|2.1319e-03
    \|MSTRG.585\|MSTRG.585.1\|1919\|7.318925\|9.3945e-06\|3.7715e-03
    PNPLA4\|MSTRG.56\|NM_004650\|203\|0.466647\|9.8645e-06\|3.7715e-03
    \|MSTRG.506\|MSTRG.506.5\|1731\|0.046993\|2.1350e-05\|6.9968e-03
    \|MSTRG.592\|MSTRG.592.1\|1923\|9.186257\|3.5077e-05\|1.0058e-02
    \|MSTRG.518\|MSTRG.518.1\|1744\|11.972859\|4.4476e-05\|1.1336e-02

>   Fold
>   Change是指女性与男性的表达比例；因此，低于1的数值意味着转录物在男性中的表达水平较低。

1.  将结果写入一个可以共享和分发的csv文件

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>write.csv(results_transcripts, "chrX_transcript_results.csv",
row.names=FALSE)
>write.csv(results_genes, "chrX_gene_results.csv",
row.names=FALSE) 
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

1.  识别q值\<0.05的转录物和基因

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
>subset(results_transcripts,results_transcripts$qval<0.05)
>subset(results_genes,results_genes$qval<0.05)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

每个命令的Ballgown输出都会出现在屏幕上；我们在下面的**Table3**（转录物）和**Table4**（基因）中显示了结果。如表所示，X号染色体有9个转录本在两性之间有差异表达（使用q值阈值为0.05），其中3个对应于已知基因的异构体（XIST、TSIX和PNPLA4）。在基因水平上（表4），X染色体在相同的q值临界点上有10个差异表达的基因。(如果使用不同版本的程序，这些表格可能包含略有不同的结果）。

-   错误排除

-   错误排除

-   错误排除
