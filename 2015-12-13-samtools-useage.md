title: 使用samtools发现SNPs
date: 2015-12-13 12:29:23
tags:   
    - NGS 
    - SNP 
    - SNV 
    - Manual
    - Commands
---
## 前言
samtools 是一个操作sam/bam文件的软件包，其最新的1.3版本和经典的0.1.19版本软件参数不同，使用前需注意使用的软件版本。  
注：本文使用的samtools版本是 0.1.19-44428cd，定义命令为samtools19，其中附带的bcftools版本是 0.1.19-44428cd，定义命令为bcftools19，vcfutils命令定义为vcfutils19。  
使用的文件是“使用 bwa 将 reads 比对到参考基因组”例子中产生的paired-end.sam文件。

<!-- more -->  

## 使用samtools faidx 对参考序列进行索引
samtools faidx 参数  
```bash
$ samtools19 faidx
        Usage: faidx <in.fasta> [<reg> [...]]
```
本例使用命令：
```bash
$ samtools19 faidx ecoli.fa
```
输出结果： ecoli.fa.fai

## 使用samtools view 将sam格式的结果转为bam格式
 samtools view 参数：
```bash
$ samtools19 view
Usage:   samtools view [options] <in.bam>|<in.sam> [region1 [...]]
Options: 
         -b             output BAM
                    #输出BAM
         -h             print header for the SAM output
         -H             print header only (no alignments)
         -S             input is SAM
                    #输入SAM
         -u             uncompressed BAM output (force -b)
         -1             fast compression (force -b)
         -x             output FLAG in HEX (samtools-C specific)
         -X             output FLAG in string (samtools-C specific)
         -c             print only the count of matching records
         -B             collapse the backward CIGAR operation
         -@ INT     number of BAM compression threads [0]
         -L FILE        output alignments overlapping the input BED FILE [null]
         -t FILE        list of reference names and lengths (force -S) [null]
         -T FILE        reference sequence file (force -S) [null]
         -o FILE        output file name [stdout]
                        #指定输出文件名而不是标准输出
         -R FILE        list of read groups to be outputted [null]
         -f INT         required flag, 0 for unset [0]
         -F INT         filtering flag, 0 for unset [0]
         -q INT         minimum mapping quality [0]
         -l STR         only output reads in library STR [null]
         -r STR         only output reads in read group STR [null]
         -s FLOAT     fraction of templates to subsample; integer part as seed [-1]
         -?                 longer help
```

本例使用命令：

```bash
$ samtools19 view -b -S -o paired-end.bam paired-end.sam
```

## 使用 samtools sort 对bam结果进行排序
samtools sort参数：
```bash
$samtools19 sort
Usage:   samtools sort [options] <in.bam> <out.prefix>
Options: 
         -n            sort by read name
         -f             use <out.prefix> as full file name instead of prefix
         -o            final output to stdout
         -l INT      compression level, from 0 to 9 [-1]
         -@ INT    number of sorting and compression threads [1]
         -m INT    max memory per thread; suffix K/M/G recognized [768M]
```

本例使用命令
```bash
$ samtools19 sort paired-end.bam paired-end.sorted
```

## 使用 samtools index 对排序后的bam文件建立索引

```bash
$samtools19 index
Usage: samtools index <in.bam> [out.index]
```

本例命令
```bash
$ samtools19 index paired-end.sorted.bam
```

## 使用 samtools mpileup 结合 bcftools view 生成二进制bcf 格式文件

**samtools mpileup 参数**

```bash
$ samtools19 mpileup

Usage: samtools mpileup [options] in1.bam [in2.bam [...]]

Input options:
  -6, --illumina1.3+      quality is in the Illumina-1.3+ encoding
  -A, --count-orphans     do not discard anomalous read pairs
  -b, --bam-list FILE     list of input BAM filenames, one per line
  -B, --no-BAQ            disable BAQ (per-Base Alignment Quality)
  -C, --adjust-MQ INT     adjust mapping quality; recommended:50, disable:0 [0]
                    #调整mapping质量，推荐50,0为关闭
  -d, --max-depth INT     max per-BAM depth; avoids excessive memory usage [250]
  -E, --redo-BAQ          recalculate BAQ on the fly, ignore existing BQs
  -f, --fasta-ref FILE    faidx indexed reference sequence file
                    #faidx 索引过的参考序列
  -G, --exclude-RG FILE   exclude read groups listed in FILE
  -l, --positions FILE    skip unlisted positions (chr pos) or regions (BED)
  -q, --min-MQ INT        skip alignments with mapQ smaller than INT [0]
  -Q, --min-BQ INT        skip bases with baseQ/BAQ smaller than INT [13]
  -r, --region REG        region in which pileup is generated
  -R, --ignore-RG         ignore RG tags (one BAM = one sample)
  --rf, --incl-flags STR|INT  required flags: skip reads with mask bits unset []
  --ff, --excl-flags STR|INT  filter flags: skip reads with mask bits set
                                            [UNMAP,SECONDARY,QCFAIL,DUP]
  -x, --ignore-overlaps   disable read-pair overlap detection

Output options:
  -o, --output FILE       write output to FILE [standard output]
  -g, --BCF               generate genotype likelihoods in BCF format
                    #使用BCF格式产生基因型可能性
  -v, --VCF               generate genotype likelihoods in VCF format
                    #使用VCF格式产生基因型可能性

Output options for mpileup format (without -g/-v):
  -O, --output-BP         output base positions on reads
  -s, --output-MQ         output mapping quality

Output options for genotype likelihoods (when -g/-v is used):
  -t, --output-tags LIST  optional tags to output: DP,DPR,DV,DP4,INFO/DPR,SP []
  -u, --uncompressed      generate uncompressed VCF/BCF output
                    #产生非压缩的VCF/BCF 输出

SNP/INDEL genotype likelihoods options (effective with -g/-v):
  -e, --ext-prob INT      Phred-scaled gap extension seq error probability [20]
  -F, --gap-frac FLOAT    minimum fraction of gapped reads [0.002]
  -h, --tandem-qual INT   coefficient for homopolymer errors [100]
  -I, --skip-indels       do not perform indel calling
  -L, --max-idepth INT    maximum per-sample depth for INDEL calling [250]
  -m, --min-ireads INT    minimum number gapped reads for indel candidates [1]
  -o, --open-prob INT     Phred-scaled gap open seq error probability [40]
  -p, --per-sample-mF     apply -m and -F per-sample for increased sensitivity
  -P, --platforms STR     comma separated list of platforms for indels [all]

Notes: Assuming diploid individuals.

```

 **bcftools view参数**

```bash
$ bcftools19 view

Usage: bcftools view [options] <in.bcf> [reg]

Input/output options:

       -A        keep all possible alternate alleles at variant sites
       -b        output BCF instead of VCF
       -D FILE   sequence dictionary for VCF->BCF conversion [null]
       -F        PL generated by r921 or before (which generate old ordering)
       -G        suppress all individual genotype information
       -l FILE   list of sites (chr pos) or regions (BED) to output [all sites]
       -L        calculate LD for adjacent sites
       -N        skip sites where REF is not A/C/G/T
       -Q        output the QCALL likelihood format
       -s FILE   list of samples to use [all samples]
       -S        input is VCF
       -u        uncompressed BCF output (force -b)

Consensus/variant calling options:

       -c        SNP calling (force -e)
       -d FLOAT  skip loci where less than FLOAT fraction of samples covered [0]
       -e        likelihood based analyses
       -g        call genotypes at variant sites (force -c)
       -i FLOAT  indel-to-substitution ratio [-1]
       -I        skip indels
       -m FLOAT  alternative model for multiallelic and rare-variant calling, include if P(chi^2)>=FLOAT
       -p FLOAT  variant if P(ref|D)<FLOAT [0.5]
       -P STR    type of prior: full, cond2, flat [full]
       -t FLOAT  scaled substitution mutation rate [0.001]
       -T STR    constrained calling; STR can be: pair, trioauto, trioxd and trioxs (see manual) [null]
       -v        output potential variant sites only (force -c)

Contrast calling and association test options:

       -1 INT    number of group-1 samples [0]
       -C FLOAT  posterior constrast for LRT<FLOAT and P(ref|D)<0.5 [1]
       -U INT    number of permutations for association testing (effective with -1) [0]
       -X FLOAT  only perform permutations for P(chi^2)<FLOAT [0.01]

```

本例使用参数：
```bash
$ samtools19 mpileup -u -f ecoli.fa paired-end.sorted.bam |bcftools19 view -bvcg - > paired-end.sorted.raw.bcf
```

##使用 bcftools view 结合 vcfutils 转换并生成 vcf 结果。

vcfutils
```bash
$ $ vcfutils19  varFilter
Usage:   vcfutils.pl varFilter [options] <in.vcf>
Options: -Q INT    minimum RMS mapping quality for SNPs [10]
         -d INT    minimum read depth [2]
         -D INT    maximum read depth [10000000]
         -a INT    minimum number of alternate bases [2]
         -w INT    SNP within INT bp around a gap to be filtered [3]
         -W INT    window size for filtering adjacent gaps [10]
         -1 FLOAT  min P-value for strand bias (given PV4) [0.0001]
         -2 FLOAT  min P-value for baseQ bias [1e-100]
         -3 FLOAT  min P-value for mapQ bias [0]
         -4 FLOAT  min P-value for end distance bias [0.0001]
		 -e FLOAT  min P-value for HWE (plus F<0) [0.0001]
         -p        print filtered variants

Note: Some of the filters rely on annotations generated by SAMtools/BCFtools.

```
本例使用命令：
```bash
bcftools19 view paired-end.sorted.raw.bcf | vcfutils19 varFilter -D 200 >paired-end.sorted.flt.vcf

```