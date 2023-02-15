---
title: "群体遗传进化分析笔记（二）"
date: 2023-02-11T17:56:29+08:00
draft: false
categories : [
  "学习",
]
tags : [
  "群体遗传",
  "生物信息学",
]
typora-root-url: ./
---

本篇为跟着组学大讲堂的群体遗传学课程的跟练记录，上一篇记录了相关分析背景知识，因此这一篇来实操一下。

*原始数据目录结构*

<img src="/img/data_tree.png" alt="data_tree" style="zoom:67%;" />

接下来进行一些环境变量的赋值方便后续分析

```shell
workdir=/work/pop_cucu #设置工作路径
refdir=$workdir/ref
datadir=$workdir/data
scriptdir=$workdir/scripts
export PATH=$scriptdir:$PATH    #组学大讲堂提供的脚本 添加到环境变量中方便使用

#一些关键文件所在的路径变量
GFF=$refdir/Cucumber_v2.gff3
REF=$refdir/Cucumber_v2.chr.fa
FAI=$refdir/Cucumber_v2.chr.fa.fai
GROUP=$datadir/pop_group.txt
```

### 对数据进行过滤

```shell
cd $workdir  #回到工作目录
mkdir 00.filter
cd 00.filter
```

两步，第一步根据indel进行硬过滤，随后使用vcftools进行软过滤。

```shell
#过滤1：vcfutils.pl  过滤掉indel附近的snp, 因此VCF数据中必须包含INDEL
# -w INT    SNP within INT bp around a gap to be filtered [3]
# -W INT    window size for filtering adjacent gaps [10]

vcfutils.pl varFilter -w 5 -W 10 "gzip -dc $datadir/cucu.vcf.gz|" |gzip - >all.varFilter.vcf.gz

#过滤2：vcftools
#--max-missing Exclude sites on the basis of the proportion of missing data 
#(defined to be between 0 and 1, where 0 allows sites that are completely missing 
#and 1 indicates no missing data allowed).

vcftools --gzvcf all.varFilter.vcf.gz --recode --recode-INFO-all --stdout     --maf 0.05  --max-missing 0.8  --minDP 2  --maxDP 1000      \
    --minQ 30 --minGQ 0 --min-alleles 2  --max-alleles 2 --remove-indels |gzip - > clean.vcf.gz  
```

两步过滤之后最终得到`clean.vcf.gz`

### 进化树分析

系统发育树（phylogenetic tree），也叫进化树。  通常系统发育树分为**有根树**和**无根树**。 常用的建树方法有基于距离的方法比如NJ或者UPGMA ；而基于特征的方法则有MP、ML、BI等方法。常用的 分析工具Phylip 、Iqtree 、Fasttree、Raxml这些，它们有些软件可能有win和linux两种版本，通常建树需求较高或者样本量多的话建议还是在linux操作系统下的集群服务器上跑。

```shell
cd $workdir  #回到工作目录
mkdir 01.phylo_tree
cd 01.phylo_tree
#文件格式转换
run_pipeline.pl  -Xmx5G -importGuess  $workdir/00.filter/clean.vcf.gz  \
    -ExportPlugin -saveAs supergene.phy -format Phylip_Inter

#最大似然法构建进化树
#方法1：fasttree 构建进化树
fasttree -nt -gtr  supergene.phy   >  fasttree.nwk
#方法2：iqtree 构建进化树  设置boots值，最大似然法
iqtree2 -s supergene.phy -st DNA -T 2  -mem 8G \
    -m  GTR  -redo \
    -B 1000 -bnni \
    --prefix iqtree 
    
#方法3：raxml 构建进化树 最大似然法
#raxml-ng  -msa supergene.phy --model GTR  --prefix raxml_tree \
    #    --threads 2 --seed 123   1>raxml.log 2>raxml.err
## with bootstrap
#raxml-ng -all  -msa supergene.phy --model GTR --bs-trees 1000 \
    #    --prefix raxml_tree_bootstrap --threads 2 --seed 123   1>raxml_bs.log 2>raxml_bs.err

#进化树绘制
ggtree.r -t fasttree.nwk -f $GROUP -g Group -n fasttree
ggtree.r -t fasttree.nwk -f $GROUP -g Group -n fasttree.unrooted -l unrooted

ggtree.r -t iqtree.treefile -f $GROUP -g Group -n iqtree
ggtree.r -t iqtree.treefile -f $GROUP -g Group -n iqtree.unrooted -l unrooted

ggtree.r -t raxml_tree.raxml.bestTree -f $GROUP -g Group -n raxml
ggtree.r -t raxml_tree.raxml.bestTree -f $GROUP -g Group -n raxml.unrooted -l unrooted
```

### PCA分析

**主成分分析**（PCA Principal Component  Analysis）：是一种降维分析方法，将数据主要的信 息以主成分形式保留下来，忽略掉数据中不重要的次要信息。这些主成分保留原始数据的绝大部分信息。

```shell
cd $workdir  #回到工作目录
mkdir 02.PCA
cd 02.PCA
#方法1：
## plink分析PCA
plink --vcf  $workdir/00.filter/clean.vcf.gz --pca 10 --out  plink_pca   \
    --allow-extra-chr --set-missing-var-ids @:#    --vcf-half-call missing

#绘图

pca_plink_plot.r -i plink_pca.eigenvec -f $GROUP -g Group --name plink_pca
```

PCA分析可以用来进行查看分群信息、检测离群样本和推断亚群间进化关系等，是一种数据降维分析的方式，在机器学习中也是一种数据的降维方式，在聚类分析和支持向量机等非监督学习很基础。



### 今日の总结

学习了如何从原始的vcf文件过滤得到clean过得vcf文件以及利用vcf文件进行基于SNP的进化树绘制和进行PCA分析，本质上都是为了后续的SNP位点的筛选。
