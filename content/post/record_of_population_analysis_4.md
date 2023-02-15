---
title: "群体遗传学学习笔记（四） 关于群体结构STRCTURE的一些分析实操记录"
date: 2023-02-13T18:07:11+08:00
draft: false
categories : [
  "学习",
]
tags : [
  "群体遗传",
  "生物信息学"
]
typora-root-url: ./
---

接着我们上次做的PCA分析，可以看到，每个不同的分析都是放在不同的文件夹下进行，这次做两个分析，一个是admixture群体结构分析，一个是LD连锁不平衡分析，关于这两个分析代表的含义，admixture主要是对群体进行亚分类，看看k值设定不同的情况下，不同亚类的分布情况，LD连锁不平衡分析则是用来判断群体进化过程中的受选择程度，其计算主要基于连锁群中等位基因的频率，LD，当位于某一座位的特定等位基因与另一座位的某一等位基因同时出现的概率大于群体中因随机分布的两个等位基因同时出现的概率时，就称这两个座位处于**连锁不平衡状态（linkage disequilibrium）**。

下面的命令主要在linux命令行或者docker命令行下运行，不同的参数可能会导致结果有所不同，因此需要设置合理的参数。

```shell
cd $workdir  #回到工作目录
mkdir 03.STRUCTURE
cd 03.STRUCTURE

## filter LD 

#50 10 0.2   50个SNP 窗口  step 10个  r2 0.2  ; 50 5 0.4
plink --vcf  $workdir/00.filter/clean.vcf.gz  --indep-pairwise 50 10 0.2 --out ld   \
    --allow-extra-chr --set-missing-var-ids @:# 
plink --vcf  $workdir/00.filter/clean.vcf.gz  --make-bed --extract ld.prune.in  \
    --out LDfiltered --recode vcf-iid  --keep-allele-order  --allow-extra-chr --set-missing-var-ids @:#  

#转换成plink格式
vcftools --vcf LDfiltered.vcf --plink \
    --out plink
#转换成admixture要求的bed格式
plink --noweb --file plink  --recode12 --out admixture \
     --allow-extra-chr  --keep-allele-order

#admixture 群体结构分析
for k in {2..10};do
    admixture -j2 -C 0.01 --cv admixture.ped $k >admixture.log$k.out
done

#绘图展示 
structure_plot.r  -d ./ -s admixture.nosex 
structure_plot.r  -d ./ -s admixture.nosex -f $GROUP -g Group  #按照分组顺序显示

#确定最佳K，CV值最小时对应的K值为最佳K

grep "CV error" *out

##########################################################
#连锁不平衡分析 LDdecay
########################################################

cd $workdir  #回到工作目录
mkdir 04.LDdecay
cd 04.LDdecay

#分组：根据自己的分组名称进行拆分

cat $GROUP |grep EastAsian |cut -f 1 >EastAsian_popid.txt
cat $GROUP |grep Eurasian |cut -f 1 >Eurasian_popid.txt
cat $GROUP |grep Xishuangbanna |cut -f 1 >Xishuangbanna_popid.txt
cat $GROUP |grep Indian |cut -f 1 >Indian_popid.txt


for i in EastAsian Eurasian Xishuangbanna  Indian;do 

    PopLDdecay -InVCF  $workdir/00.filter/clean.vcf.gz \
        -SubPop  ${i}_popid.txt -MaxDist 500 -OutStat ${i}.stat
    Plot_OnePop.pl -inFile ${i}.stat.gz -output ${i}.ld
done 

#多个群体放在一张图中
echo EastAsian.stat.gz EastAsian >ld_stat.list
echo Eurasian.stat.gz Eurasian >>ld_stat.list
echo Xishuangbanna.stat.gz Xishuangbanna >>ld_stat.list
echo Indian.stat.gz Indian >>ld_stat.list
Plot_MultiPop.pl -inList ld_stat.list -output ld_stat.multi
```

### 今日の学习总结

今天主要跟着上面的流程跑了一遍两个分析，感觉对于群体进化的速率这个概念和群体中亚分类的标准有了一定的认识，我们在研究群体的时候往往不能只考虑地理分布的环境，还应该考虑到当地这个群体是否是由别的群体迁徙而来以及一些进化历史中的大事件对其造成的影响等等因素。因此，群体进化的研究用GWAS分析比较合适，GWAS检测到显著关联的区间，可以通过进一步绘制局部的LD单体型块图，来进一步判断显著相关的SNP和目标基因间是否存在强LD关系。
