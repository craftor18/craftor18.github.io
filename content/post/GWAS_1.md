---
title: "GWAS_1"
date: 2023-11-26T21:21:26+08:00
draft: false
categories : [
  "学习",
]
tags : [
  "群体遗传学",
  "GWAS分析"
]
typora-root-url: ./
---
基因型数据：WGS数据首先经过bwa比对、samtools排序、GATK Call SNP 和vcftools等程序过滤，最后只要提取出SNP位点进行后续的GWAS分析。芯片数据我不太熟悉，但是应该是类似数据，但是可能中间多一步基因型填充用于填补缺失的基因型。由于做的是非模式生物的，所以vcf注释采用的是snpEFF进行注释。首先build构建好注释用的索引，然后java一下运行一下注释，这一步主要是为了将SNP位点与基因对应，每个有注释的SNP位点都应该在某个基因的内含子区域或者5-端上游或者3-端下游等区域。最后使用一个python脚本（网上大佬的）将ID替换成染色体_position，得到最终可以用于做GWAS分析的高质量SNP位点数据集

```python
#author_zukunft
#data_202210

import vcf
import argparse

parser = argparse.ArgumentParser(description='add ID to VCF_file: ".">>"chr*-POS"')
parser.add_argument('--input', type=str, nargs='?',
                    help='input file')

parser.add_argument('--output', type=str, nargs='?',
                    help='output file')

args = parser.parse_args()

input_file = args.input
output_file = args.output


vcf_reader = vcf.Reader(filename=input_file)
vcf_writer = vcf.Writer(open("tsp.2-8.rb17.dp5_indv2_maf5.impute.clear_site.snp.nochr.ID.vcf", 'w'), vcf_reader)

if __name__ == "__main__":
    line=0
    for record in vcf_reader:
        # print(record.ID)
        record.ID = str(record.CHROM) + "_"+ str(record.POS) #自由修改
        # print(record.ID)
        line+=1
        # print("正在执行第" + line + "SNP")
        vcf_writer.write_record(record)

vcf_writer.close()
print("done,The numbers of SNPs:")
print(line)
```

usage:

```bash
tools_py=/path/vcf_add_ID.py #vcf_add_ID.py见下面

input_vcf=/path/**.vcf.gz  #**.vcf也可
output_vcf=/path/**.ID.vcf.gz

time python3 $tools_py --input $input_vcf --output $output_vcf
```

表型数据：

其实就是首先将数据整理好，FID、IID、trait1...，这些。然后用r语言做一下统计，看看是否符合类正态分布这些啥的，要做标准化的话就做一下，不做也行。。。除非表型数据特别奇怪，否则应该都是能用的吧。。。

![image-20231126213906037](/C:/Users/13694/AppData/Roaming/Typora/typora-user-images/image-20231126213906037.png)

差不多像上面这样的图弄一下看看就行。

GWAS：经典GEMMA，启动！！！！！！
