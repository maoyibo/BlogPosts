title: 使用 bwa 将 reads 比对到参考基因组
date: 2015-12-12 22:02:04
tags:  
        - NGS
        - Mapper
        - Manual
        - Commands
-------------------------
## 前言

Bwa (Burrows-Wheeler Alignment Tool) 是一个将低分散序列比对到人类基因组等大参考基因组的软件包。它由三个算法组成：BWA-backtrack，BWA-SW 和 BWA-MEM。第一个算法BWA-backtrack针对最长 100bp 的 Illumina sequence reads。另外两个算法BWA-SW 和 BWA-MEM针对从70bp 到 1Mbp 的长序列。BWA-SW 和 BWA-MEM 具有相同的特征，比如长read的支持，支持分裂比对，但是BWA-MEM是最新的，更快，更精准，推荐用于高质量比对。 BWA-MEM 和 BWA-backtrack 相比处理70-100bp Illumina reads 拥有更好的性能。  
对于所有的算法，BWA首先需要构建参考基因组的FM-index（使用 index 命令）。比对算法通过调用不同的子命令实现：BWA-backtrack 算法使用aln/samse/sampe，BWA-SW算法使用 bwasw，BWA-MEM 算法使用mem。  
注：本例使用的bwa版本为：0.7.12-r1039。BWA手册[查看此处](http://bio-bwa.sourceforge.net/bwa.shtml)。

<!-- more -->  

bwa的使用需要两种输入文件：  
**Reference genome data (fasta格式 .fa, .fasta, .fna)**  
本文使用的例子是Escherichia coli 参考基因组（gi|15829254|ref|NC_002695.1| Escherichia coli O157:H7 str. Sakai chromosome, complete genome）。  
文件下载后命名为ecoli.fa

**Short reads data (fastaq格式 .fastaq, .fq)**  
本文使用的read数据是从SRA获得的 SRR2982241 reads数据。  

- Name: NexteraXT
- Instrument: Illumina MiSeq
- Strategy: WGS
- Source: GENOMIC
- Selection: RANDOM
- Layout: PAIRED
- Construction protocol: NexteraXT

首先使用sratoolkit中的fastq-dump 对下载的 SRR2982241.sra 格式文件转换格式为fastq格式文件，并将序列分开。
```bash
$ fastq-dump --split-files SRR2982241.sra
```
生成文件 SRR2982241_1.fastq 和 SRR2982241_2.fastq


## 使用 bwa index 对参考基因组建立索引  

bwa index 索引fasta格式的参考基因组序列，其参数如下：
```
$ bwa index
Usage:   bwa index [options] <in.fasta>
Options: 
        -a STR    BWT construction algorithm: bwtsw or is [auto]
                    #BWT构建算法bwtsw 或 is ，默认自动选择
        -p STR    prefix of the index [same as fasta name]  
                    #输出的索引的前缀，默认和FASTA文件名相同
        -b INT    block size for the bwtsw algorithm (effective with -a bwtsw) [10000000]   
                    #bwtsw算法的块大小，当使用bwtsw算法时生效，默认10000000
        -6        index files named as <in.fasta>.64.* instead of <in.fasta>.*  
                    #输出的索引文件名中有.64.标记
                    
Warning: `-a bwtsw' does not work for short genomes, while `-a is' and `-a div' do not work not for long genomes.
#警告：`-a bwtsw'不适用于短基因组，而`-a is'  和 `-a div' 不适用于长基因组
```

本例使用的是大肠杆菌基因组，属于short genomes，使用`-a is'参数。使用命令：

```bash
$ bwa index -a is ecoli.fa 
```
默认生成文件的前缀和输入文件相同，输出文件有:ecoli.fa.ann， ecoli.fa.pac， ecoli.fa.amb， ecoli.fa.bwt， ecoli.fa.sa。

## 使用 bwa aln 定位序列的阵列坐标 （SA coordinates）

bwa aln 参数：
```
$ bwa aln
Usage:   bwa aln [options] <prefix> <in.fq>
Options: 
        -n NUM      max #diff (int) or missing prob under 0.02 err rate (float) [0.04]
        -o INT         maximum number or fraction of gap opens [1]
        -i INT          do not put an indel within INT bp towards the ends [5]
        -d INT         maximum occurrences for extending a long deletion [10]
        -l INT          seed length [32]
        -k INT         maximum differences in the seed [2]
        -m INT        maximum entries in the queue [2000000]
        -t INT          number of threads [1] 
                        #指定进程数
        -M INT        mismatch penalty [3]
        -O INT        gap open penalty [11]
        -E INT         gap extension penalty [4]
        -R INT         stop searching when there are >INT equally best hits [30]
        -q INT         quality threshold for read trimming down to 35bp [0]
        -f FILE         file to write output to instead of stdout 
                        #指定输出文件名而不是标准输出
        -B INT         length of barcode
        -L                 log-scaled gap penalty for long deletions
        -N                non-iterative mode: search for all n-difference hits (slooow)
        -I                 the input is in the Illumina 1.3+ FASTQ-like format
                        #输入文件是 Illumina 1.3+FASTQ-like格式
        -b                the input read file is in the BAM format
                        #输入的read是BAM格式
        -0                use single-end reads only (effective with -b)
                        #只使用单末端，使用-b时有效
        -1                use the 1st read in a pair (effective with -b)
                        #使用末端配对的第一个read，使用-b时有效
        -2                use the 2nd read in a pair (effective with -b)
                        #使用末端配对的第二个read，使用-b时有效
        -Y                filter Casava-filtered sequences
```

对于Paired-end reads，使用8进程运行，使用命令：

```bash
$ bwa aln -t 8 -f SRR2982241_1.sai ecoli.fa SRR2982241_1.fastq 
$ bwa aln -t 8 -f SRR2982241_2.sai ecoli.fa SRR2982241_2.fastq 
```
对于Single reads，使用命令：

```bash
$ bwa aln -t 8 -f SRR1234567.sai refseq.fa SRR1234567.fastq 

```

## 使用 bwa sampe 或 bwa samse 转换序列阵列坐标并输出sam

bwa sampe 和 bwa samse 参数 
```bash
Paired-end reads:
$ bwa sampe
Usage:   bwa sampe [options] <prefix> <in1.sai> <in2.sai> <in1.fq> <in2.fq>
Options: 
        -a INT   maximum insert size [500]
        -o INT   maximum occurrences for one end [100000]
        -n INT   maximum hits to output for paired reads [3]
        -N INT   maximum hits to output for discordant pairs [10]
        -c FLOAT prior of chimeric rate (lower bound) [1.0e-05]
        -f FILE  sam file to output results to [stdout]
        #定输出sam文件而不是标准输出
        -r STR   read group header line such as `@RG\tID:foo\tSM:bar' [null]
        -P       preload index into memory (for base-space reads only)
        -s       disable Smith-Waterman for the unmapped mate
        -A       disable insert size estimate (force -s)
Notes:
    1. For SOLiD reads, <in1.fq> corresponds R3 reads and <in2.fq> to F3.
    2. For reads shorter than 30bp, applying a smaller -o is recommended get a sensible speed at the cost of pairing accuracy.

Single reads:
$ bwa samse
Usage: bwa samse [-n max_occ] [-f out.sam] [-r RG_line] <prefix> <in.sai> <in.fq>

```

本例是Paired-end reads，使用命令：

```
$ bwa sampe -f paired-end.sam ecoli.fa SRR2982241_1.sai SRR2982241_2.sai SRR2982241_1.fastq SRR2982241_2.fastq
```

对于Single reads，使用命令：  

```
$ bwa samse -f single_reads.sam refseq.fa SRR1234567.sai SRR1234567.fastq 
```

