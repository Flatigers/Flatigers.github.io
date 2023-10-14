---
title: 运行一个RNA-Seq流程
date: 2022-04-18 10:20:02
tags:
    - 生物信息学
    - RNA-Seq
categories:
    - 生物信息学习
---

## 运行一个RNA-Seq流程
---
> 此流程根据《使用HISAT StringTie和Ballgown进行的转录本表达水平分析》重复文中的RNA-Seq实验

### 流程设计
本流程分为如下步骤：
1. 生物信息服务器的搭建
2. 生信环境的配置
3. 在生信环境上操作RNA-seq实验
4. 统计数据及结果分析

### 生信服务器的搭建
生物信息学分析所使用的数据量极大，往往需要数百G以上的存储空间及大量内存，这样的需求在一般PC上较难实现。而且生信分析往往需要众人分工协作，由此搭建一个公用服务器十分重要。

本次重复的实验使用的数据较小，在个人PC上完全可以操作，需要的硬盘存储空间约**20G**以内，由于条件的限制，作者本次实验在个人Linux系统上进行操作。

生物信息学实验使用服务器是必不可少的，在今后的操作中，读者需熟练掌握Linux操作系统及shell命令编程的使用，熟练使用云Linux服务器进行操作。国内云计算提供商有**腾讯云，阿里云，华为云**等，新用户均有数月的云服务器体验，读者可根据自己的实际情况自行选择。

### 生信环境的配置
#### 系统环境
根据此流程需要进行的操作，本次实验需要在Linux系统上配置的环境如下：
- R语言环境
- R包：alyssafrazee/RSkittleBrewer, **ballgown**, genefilter, dplyr, devtools
- 分析软件：Hisat2, StringTie, **Samtools**， gffcompare

安装hisat2和StringTie:
```
sudo apt update
sudo apt install hisat2
sudo apt install stringtie
```
安装Conda并使用其安装Samtools, gffcompare:
```
wget -c https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
# 适合于linux 64位
chmod 777 Miniconda3-latest-Linux-x86_64.sh
#给执行权限
bash Miniconda3-latest-Linux-x86_64.sh
#运行安装程序，所有选项均选'yes'

conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda/
#配置conda频道，这里选择清华Tuna镜像

conda install samtools
conda install gffcompare
#使用conda安装Samtools和gffcompare
#这里使用conda安装是因为作者使用apt安装samtools时出现问题
#conda是专用于生物信息学分析的环境配置器
#Hisat, StringTie以及其他R包均可使用conda安装
```

安装 R 和 R包Ballgown：
```
conda create -n Rspcae
#创建名为Rspace的conda环境，可以替换成自己需要的名称
source activate Rspace
#激活此环境

conda install r-base=4.1.3
#安装4.1.3的R，可以在conda官网包管理页面查询r包的最新版本
conda install r-ballgown
#conda 安装r包需要在包名前 添加 r-
#此处作者使用conda安装r包，也是因为在R中安装遇到问题
#若无问题，可以直接在R中安装任何R包，具体为：
#install.packages(ballgown) 或
#BiocManager::install("ballgown")
#使用BiocManager需先安装，参加下一步

conda deactivate
#安装完成后退出当前conda环境
```

安装其他R包：
```
R        
#进入R环境
install.packages(“devtools”)

install.packages("BiocManager")
library("BiocManager")
#安装并导入BiocManager


BiocManager::install(c("alyssafrazee/RSkittleBrewer","ballgown","genefilter","dplyr"))

#source("http://www.bioconductor.org/biocLite.R")
#BiocManager::biocLite(c("alyssafrazee/RSkittleBrewer","ballgown","genefilter","dplyr"))
#安装R包,此项为旧版本用法
#此处也可使用：
#install.packages()
```


#### 资源
此处所需资源为文章中提供的测序数据及参考序列等资料：
```
cd /home/username/RNA
nohup wget ftp://ftp.ccb.jhu.edu/pub/RNAseq_protocol/chrX_data.tar.gz &
tar zxvf chrX_data.tar.gz
#先cd到工作环境下文件夹
#下载并解压
```

### 在生信环境上操作RNA-seq实验
#### hisat2:将reads与genome匹配
```
for i in *1.fastq.gz; 
do
i=${i%1.fastq.gz*}; 
nohup hisat2 -p 8 --dta -x /home/username/RNA/chrX_data/indexes/chrX_tran -1 ${i}1.fastq.gz -2 ${i}2.fastq.gz -S /home/username/RNA/chrX_data/result/${i}align.sam /home/username/RNA/chrX_data/result/${i}align.log & 
done

#此步骤将在/ home/username/RNA/chrX_data/result/ 下生成sample reads的sam文件
```
#### samtools:将sam文件转换为bam文件
```
for i in *.sam;
do 
i=${i%_align.sam*};
nohup samtools sort -@ 8 -o ${i}.bam ${i}_align.sam & 
done

#此步骤将在同目录下生成bam文件
```
#### stringtie:组装转录本并定量表达基因
```
for i in *.bam; 
do 
i=${i%.bam*};
nohup stringtie -p 8 -G /home/username/RNA/chrX_data/genes/chrX.gtf -o ${i}.gtf ${i}.bam & 
done

#此步骤将在同目录下生成gtf文件
```
#### stringtie:合并所有转录本
```
stringtie --merge -p 8 -G  /home/username/RNA/chrX_data/genes/chrX.gtf -o stringtie_merged.gtf  /home/username/RNA/chrX_data/mergelist.txt

#此步骤将在同目录下生成 stringtie_merged.gtf 文件
```
#### gffcompare:检测相对于注释文件，转录本的组装情况
```
gffcompare -r /home/username/RNA/chrX_data/genes/chrX.gtf  -G -o merged stringtie_merged.gtf

#此步骤将生成如下文件：
#gffcmp.annotated.gtf
#它在每个转录本上添加一个 “类代码”（描述参见流程文档）和注释文件中的转录本名称
#gffcmp.stats
#输出文件中计算不同基因特征的灵敏度和精确度（P值）统计
```
#### stringtie:重新组装转录本并估算基因表达丰度，并为ballgown创建读入文件
```
for i in *_chrX.bam; 
do 
i=${i%_chrX.bam*};
nohup stringtie -e -B -p 8 -G stringtie_merged.gtf -o ballgown/${i}/${i}_chrX.gtf  ${i}_chrX.bam & 
done
#此步骤将在 /home/username/RNA/chrX_data/ballgown/ 下生成
#以sample名为文件夹名的的ballgown读取文件
```

### 统计数据及结果分析
此步骤将在R环境中进行统计步骤及结果分析

#### 数据处理
- 进入R并加载R包
```
library(ballgown)
library(RSkittleBrewer)
library(genefilter)
library(dplyr)
library(devtools)
```
- 加载表型数据
```
setwd("/home/username/RNA/chrX_data/")
#设置R工作环境目录
pheno_data = read.csv("geuvadis_phenodata.csv")
```
- 读入StringTie的计算结果
```
bg_chrX = ballgown(dataDir = "/home/username/RNA/chrX_data/result/ballgown", samplePattern = "ERR", pData=pheno_data)
```
- 滤去低丰度基因（样本间差异少于一个转录本）
```
bg_chrX_filt = subset(bg_chrX,"rowVars(texpr(bg_chrX)) >1",genomesubset=TRUE)
```
- 得到组间有差异的转录本和基因
```
results_transcripts = stattest(bg_chrX_filt, feature="transcript" covariate="sex",adjustvars = c("population"),  getFC=TRUE, meas="FPKM")

results_genes = stattest(bg_chrX_filt, feature="gene", covariate="sex", adjustvars = c("population"), getFC=TRUE, meas="FPKM")
```

- 对转录本结果添加基因名
```
results_transcripts = data.frame(geneNames=ballgown::geneNames(bg_chrX_filt), geneIDs=ballgown::geneIDs(bg_chrX_filt), results_transcripts)
```
- 转录本与基因结果按P值排序
```
results_transcripts=arrange(results_transcripts,pval)
results_genes=arrange(results_genes,pval)
```
- 将排序结果写入文件
```
write.csv(results_transcripts, "chrX_transcript_results.csv",row。names=FALSE)
write.csv(results_genes, "chrX_gene_results.csv",row.names=FALSE)

#此步骤生成 chrX_transcript_results.csv, chrX_gene_results.csv
#两个文字表格文件
```

-  识别q值小于0.05的转录物和基因（性别间表达有差异）
```
subset(results_transcripts,results_transcripts$qval<0.05)
subset(results_genes,results_genes$qval<0.05)
```

#### 结果可视化

- FPKM为参考值，性别为区分条件作图
```
palette(rainbow(5))
#为调色板指定颜色

fpkm = texpr(bg_chrX,meas="FPKM")
fpkm = log2(fpkm+1)
#提取FPKM值并做log转换
boxplot(fpkm,col=as.numeric(pheno_data$sex),las=2,ylab='log2(FPKM+1)')

```

- 单个样本在转录组上的分布
```
ballgown::transcriptNames(bg_chrX)[12]

ballgown::geneNames(bg_chrX)[12]


# 绘制箱线图
plot(fpkm[12,] ~ pheno_data$sex, border=c(1,2),
    main=paste(ballgown::geneNames(bg_chrX)[12],' : ',
    ballgown::transcriptNames(bg_chrX)[12]),pch=19, xlab="Sex",
    ylab='log2(FPKM+1)')
points(fpkm[12,] ~ jitter(as.numeric(pheno_data$sex)),
    col=as.numeric(pheno_data$sex))
```


- 查看某一基因位置上所有转录组
```
plotTranscripts(ballgown::geneIDs(bg_chrX)[2263], bg_chrX, main=c('Gene XIST in sample ERR188234'), sample=c('ERR188234'))
#根据ERR188234.gtf的文件，查出XIST基因所在位置编码2263，进行绘图
```

- 以性别为区分查看特定基因表达情况
```
plotMeans('MSTRG.56', bg_chrX_filt,groupvar="sex",legend=FALSE)
```

---
由于作者精力有限，编写时难免会出现错误和过失，对您的阅读和操作造成的不便请您谅解。如果方便，还请把错误处邮件至本人邮箱，或于公众号处留言，十分感谢！