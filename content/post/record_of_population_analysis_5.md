---
title: "群体遗传进化学学习笔记（五）选择清除分析运行实操"
date: 2023-02-14T14:37:53+08:00
draft: false
categories : [
  "学习",
]
tags : [
  "群体遗传学",
  "生物信息学",
]
typora-root-url: ./
---

从群体遗传学的各个概念，到群体遗传学结构分析的各个步骤，可以看出，这些值的计算基本都是基于vcf文件，因此重测序分析的学习也很重要，不过既然我们已经有了vcf文件，那么本篇实操一下**选择清除分析**的流程，当然，还是用基因组学大讲堂的流程，但是我在运行过程中也发现了一些脚本需要修改一下才能正确显示出图。

数据目前的目录结构:我所有的脚本和命令都拷到了linux的docker 里面，因此工作路径还是比较清楚的，如何安装docker和镜像的教程可以自己上网查查。组学大讲堂里好像也有。

` docker pull omicsclass/pop-evol-gwas:latest #下载镜像`

` docker run --rm -it -m 4G --cpus 1 -v /share/work/huangls/pop_evo:/work omicsclass/pop-evol-gwas:latest`

![dirstr](/img/dirstr.png)

### Fst、pi值的计算

```shell
######################################################
#选择消除分析  fst pi ROD  基于多样性降低
########################################################
cd $workdir  #回到工作目录
mkdir 05.select_sweep
cd 05.select_sweep

#不同群体的样本分开放到不同的文件中
cat $GROUP |grep wild |cut -f 1 >wild_popid.txt
cat $GROUP |grep cultivated|grep -v "Indian"|grep -v "Xishuangbanna" |cut -f 1 >cultivated_popid.txt

#fst  pi  tajimaD分析
mkdir fst_pi_ROD
cd fst_pi_ROD
#设置输入的vcf文件与计算的窗口和步长
gzvcf=$workdir/00.filter/clean.vcf.gz
window=100000
step=10000

#pi 多样性
vcftools  --gzvcf $gzvcf \
    --window-pi $window --window-pi-step  $step  \
    --keep ../wild_popid.txt   --out pi.wild
vcftools  --gzvcf $gzvcf \
    --window-pi $window --window-pi-step  $step  \
    --keep ../cultivated_popid.txt  --out pi.cultivated

#pi多样性绘图输出
pi_manhattan_plot.r -i pi.wild.windowed.pi -F $FAI -f 19226500 -n pi.wild
pi_smooth_line_plot.r -i pi.wild.windowed.pi -F $FAI -f 19226500  -n pi.wild.smoothline

#Fst  群体间多样性差异
vcftools  --gzvcf $gzvcf --fst-window-size $window --fst-window-step $step  \
    --weir-fst-pop  ../wild_popid.txt --weir-fst-pop ../cultivated_popid.txt --out  Fst.wild.cultivated

#fst绘图输出
fst_manhattan_plot.r -i Fst.wild.cultivated.windowed.weir.fst -F $FAI -f 19226500 -n Fst.wild.cultivated
fst_manhattan_plot.r -i Fst.wild.cultivated.windowed.weir.fst -F $FAI -f 19226500  -n Fst.wild.cultivated_vline --vline
fst_smooth_line_plot.r -i Fst.wild.cultivated.windowed.weir.fst -F $FAI -f 19226500  -n Fst.wild.cultivated_smoothline

#fst 与 pi 联合 筛选受选择区域
Rscript $scriptdir/fst_pi_select_sweep.r --fst Fst.wild.cultivated.windowed.weir.fst \
    --pi1 pi.wild.windowed.pi --pi2 pi.cultivated.windowed.pi --zscore --log2 \
    -A wild -B cultivated -c 0.05 -n fst-pi.wild-vs-cultivated  -f pdf

Rscript $scriptdir/fst_pi_select_sweep.r --fst Fst.wild.cultivated.windowed.weir.fst \
    --pi1 pi.wild.windowed.pi --pi2 pi.cultivated.windowed.pi --zscore --log2 \
    -A wild -B cultivated -c 0.05 -n fst-pi.wild-vs-cultivated  -f png
```

### ROD值的计算及绘图

```shell
#ROD计算与展示
rod_calculate.r --wild pi.wild.windowed.pi --domesticated pi.cultivated.windowed.pi -p ROD.wild.cultivated

#绘图输出
rod_manhattan_plot.r -i ROD.wild.cultivated.txt -F $FAI -f 19226500  -n ROD.wild.cultivated 
rod_smooth_line_plot.r -i ROD.wild.cultivated.txt -F $FAI -f 19226500  -n ROD.wild.cultivated.smoothline
```

### 选择消除分析 

```shell
#################################################
#选择消除分析   XP-CLR  tajima'D 分析  基于频率差异  
#################################################
cd  $workdir/05.select_sweep
mkdir xpclr
cd xpclr


## xpclr软件每次只能运行1个染色体
## 原始的XPCLR分析：https://www.omicsclass.com/article/1334

#python 版本的xpclr,结果说明见：  https://github.com/hardingnj/xpclr
#不同的染色体分别计算
for i in Chr1 Chr2 Chr3 Chr4 Chr5 Chr6 Chr7;do (xpclr --format vcf --input $workdir/00.filter/clean.vcf.gz --samplesA ../wild_popid.txt --samplesB ../cultivated_popid.txt --out ./$i.xpclr --chr $i --size $window  --step $step --maxsnps 200 --minsnps 5 &);done
#python 版本的xpclr参数使用说明：
#--format 指定输入文件格式
#--input 指定输入文件
#--samplesA or B 指定参与分析的个体
#--chr 指明运行的染色体
#--phased 指明vcf文件是phased后的（如果没有phased就去掉）
#--ld 指定用于加权的连锁不衡阈值
#--maxsnp 窗口内参与计算的SNP数量
#--size 指定滑窗的大小（物理距离）
#--step 指定步长
#--out 指定输出文件名
#--rrate 指定每bp的重组率


#awk合并文件 保留第一个文件的表头
awk 'FNR==1 && NR!=1{next;}{print}' *.xpclr >all.chr.xpclr
xpclr_manhattan_plot.r -i all.chr.xpclr -F $FAI -f 19226500  -n all.chr.xpclr --vline
xpclr_smooth_line_plot.r -i all.chr.xpclr -F $FAI -f 19226500  -n all.chr.xpclr.smoothline


cd  $workdir/05.select_sweep
mkdir TajimaD
cd TajimaD

#Tajima's D 
#vcftools无法用滑动窗口
vcftools --gzvcf $gzvcf --TajimaD  100000  --keep  ../wild_popid.txt  --out wild
vcftools --gzvcf $gzvcf --TajimaD  100000  --keep  ../cultivated_popid.txt  --out cultivated


tajimaD_manhattan_plot.r -i cultivated.Tajima.D -F $FAI -f 19226500  -n TajimaD.cultivated 
tajimaD_smooth_line_plot.r -i cultivated.Tajima.D -F $FAI -f 19226500  -n TajimaD.cultivated.smoothline

tajimaD_manhattan_plot.r -i wild.Tajima.D -F $FAI -f 19226500  -n TajimaD.wild 
tajimaD_smooth_line_plot.r -i wild.Tajima.D -F $FAI -f 19226500  -n TajimaD.wild.smoothline


####################
#xpehh  https://www.mdpi.com/2076-2615/11/1/157/htm
#########################
#需要遗传图或者phase


###############################################
#筛选候选区域基因
##################################################
cd  $workdir/05.select_sweep

#以Fst top 5%的区域作为候选区域，筛选区域里面的基因为例
#合并区域
bedtools merge -i fst_pi_ROD/Fst.wild.cultivated_selected_region_top0.05.txt >fst_top0.5_selected_region.txt

#gff中获取基因位置
awk '$3=="gene"' $GFF > gene.gff
#筛选受选择区域内基因
bedtools  intersect -wo -F 0.1 -nonamecheck -a  fst_top0.5_selected_region.txt -b gene.gff | sort -u  > Fst.top0.05.gene.txt
```

### 今日の总结

主要学习了如何利用vcftools根据vcf文件中info和样本haplotype里有关基因频率参数来计算fst和pi以及ROD等各项与群体遗传进化学上重要值，计算过程则无需我们担心，因为vcftools会自动识别，但是有关结果的结构和转换是我们需要知道的，因为这能帮助我们有效筛选符合条件的SNP和绘图时设置一些参数从而避免绘图结果不好看或者筛选出一些假阳性的结果。总的来说，等位基因频率与中性进化假说是我们计算这三个数值时需要考虑到的，根据这些来设置有关滑动窗口以及steps从而得到合理的计算结果，避免错过一些重要SNP位点。在学习的过程中我主要参考其利用bedtools提取对应基因位置的过程，以此来编写自己提取SNP位置的过程，因为SNP位置的提取似乎只需要以来vcf文件即可，无需参考基因组注释文件，但是需要是snpeff注释过得vcf文件。
