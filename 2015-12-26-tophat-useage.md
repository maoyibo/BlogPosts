title: TopHat使用方法
date: 2015-12-26 21:53:35
tags:  
    - NGS
    - RNASeq
    - Mapper
    - Manual
    - Commands
    - Unfinished
---
TopHat是一个将RNA-Seq reads 比对到基因组上，来鉴定外显子-外显子剪接点的程序，其基于超快速短read mapping 程序 Bowtie。  
TopHat 被设计用于Illumina Genome Analyzer所产生的reads，但是用户也成功地将其用于其他技术产生的reads。此软件1.1.0版本开始可以支持 Applied Biosystems 公司的 Colorspace format。此软件针对75bp或者更长的reads进行了优化。  
注：本例使用的TopHat版本是：v2.1.0 。TopHat手册[查看此处](http://ccb.jhu.edu/software/tophat/manual.shtml "tophat manual")  

<!-- more -->  

## TopHat如何寻找剪接点
TopHat可以在没有参考注释的情况下寻找剪接点。通过首先将RNA-Seq reads比对到基因组，TopHat鉴定潜在的基因组，因为许多RNA-Seq reads 可以连续地比对到基因组上。使用这些最初的比对信息，TopHat构建可能的剪接点的数据库，然后比reads到到这些剪接点来确认他们。  
当前的测序仪可以产生100bp 或更长的reads。但是许多外显子比这个长度更短，所以这些外显子可能在第一轮的mapping中被忽视。TopHat 通过将输入reads切割成更小的片段独立比对来解决这个问题。片段的比对在程序的最后一步放在一起产生end-to-end reads 比对。  
TopHat产生可能剪接点的数据库从两个证据来源。一个剪接点的第一个也是最强的证据来源是当同一个read（最短45bp）的两个片段在同一个基因组系列比对到一个具体的距离，或者当一个内部片段不能比对表示这些reads跨越多个外显子。通过这些手段，可以从头开始发现 "GT-AG", "GC-AG" 和 "AT-AC" 内含子。第二个证据来源是“覆盖岛”的配对，他们是在初始mapping 中不同的reads 对叠区域，相邻的岛经常在转录组中连接在一起，这样TopHat寻找方法来连接他们和一个内含子。只推荐用户对短的read (< 45bp)，和小数目的reads (<= 10 million) 使用第二个选项(--coverage-search) 。这个选项只报告跨过"GT-AG" 内含子的比对。

## 使用TopHat
TopHat 的参数：
```bash
$ tophat2 -h
TopHat maps short sequences from spliced transcripts to whole genomes.
Usage:
    tophat [options] <bowtie_index> <reads1[,reads2,...]> [reads1[,reads2,...]] [quals1,[quals2,...]] [quals1[,quals2,...]]
    #bowtie_index  基因组索引的路径和basename
    #reads*  一个包含reads的文件，可以是fastq格式或者fasta格式
    #quals* 与fasta文件对应的质量文件
Options:
    -v/--version    #版本
    -o/--output-dir                <string>    [ default: ./tophat_out ] #输出文件目录，默认 ./tophat_out
    --bowtie1                                  [ default: bowtie2 ] #使用bowtie1,默认bowtie2
    -N/--read-mismatches           <int>       [ default: 2 ]
    --read-gap-length              <int>       [ default: 2 ]
    --read-edit-dist               <int>       [ default: 2 ]
    --read-realign-edit-dist       <int>       [ default: "read-edit-dist" + 1 ]
    -a/--min-anchor                <int>       [ default: 8 ]
    -m/--splice-mismatches         <0-2>       [ default: 0 ]
    -i/--min-intron-length         <int>       [ default: 50 ]
    -I/--max-intron-length         <int>       [ default: 500000  ]
    -g/--max-multihits             <int>       [ default: 20 ]
    --suppress-hits
    -x/--transcriptome-max-hits    <int>       [ default: 60 ]
    -M/--prefilter-multihits                   ( for -G/--GTF option, enable an initial bowtie search against the genome )
    --max-insertion-length         <int>       [ default: 3 ]
    --max-deletion-length          <int>       [ default: 3 ]
    --solexa-quals      #使用solexa分数
    --solexa1.3-quals                          (same as phred64-quals) #使用solexa1.3分数
    --phred64-quals                            (same as solexa1.3-quals) #使用phred64分数
    -Q/--quals      #分数
    --integer-quals     #证书分数
    -C/--color                                 (Solid - color space) #Solid-色彩空间
    --color-out     
    --library-type                 <string>    (fr-unstranded, fr-firststrand, fr-secondstrand)
    -p/--num-threads               <int>       [ default: 1 ]
    -R/--resume                    <out_dir>   ( try to resume execution )
    -G/--GTF                       <filename>  (GTF/GFF with known transcripts)
    --transcriptome-index          <bwtidx>    (transcriptome bowtie index)
    -T/--transcriptome-only                    (map only to the transcriptome)
    -j/--raw-juncs                 <filename>   #未处理的剪切点
    --insertions                   <filename>
    --deletions                    <filename>
    -r/--mate-inner-dist           <int>       [ default: 50 ]
    --mate-std-dev                 <int>       [ default: 20 ]
    --no-novel-juncs
    --no-novel-indels
    --no-gtf-juncs
    --no-coverage-search
    --coverage-search
    --microexon-search
    --keep-tmp
    --tmp-dir                      <dirname>   [ default: <output_dir>/tmp ]
    -z/--zpacker                   <program>   [ default: gzip ]
    -X/--unmapped-fifo                         [use mkfifo to compress more temporary files for color space reads]
Advanced Options:
    --report-secondary-alignments
    --no-discordant
    --no-mixed
    --segment-mismatches           <int>       [ default: 2 ]
    --segment-length               <int>       [ default: 25 ] #片段长度
    --bowtie-n                                 [ default: bowtie -v ]
    --min-coverage-intron          <int>       [ default: 50 ]
    --max-coverage-intron          <int>       [ default: 20000 ]
    --min-segment-intron           <int>       [ default: 50 ]
    --max-segment-intron           <int>       [ default: 500000 ]
    --no-sort-bam                              (Output BAM is not coordinate-sorted)
    --no-convert-bam                           (Do not output bam format. Output is <output_dir>/accepted_hits.sam)
    --keep-fasta-order
    --allow-partial-mapping
Bowtie2 related options:
  Preset options in --end-to-end mode (local alignment is not used in TopHat2) #预设的选项是 --end-to-end模式，(local alignment在TopHat2中不能被使用)
    --b2-very-fast
    --b2-fast
    --b2-sensitive
    --b2-very-sensitive
  Alignment options
    --b2-N                         <int>       [ default: 0 ]
    --b2-L                         <int>       [ default: 20 ]
    --b2-i                         <func>      [ default: S,1,1.25 ]
    --b2-n-ceil                    <func>      [ default: L,0,0.15 ]
    --b2-gbar                      <int>       [ default: 4 ]
  Scoring options
    --b2-mp                        <int>,<int> [ default: 6,2 ]
    --b2-np                        <int>       [ default: 1 ]
    --b2-rdg                       <int>,<int> [ default: 5,3 ]
    --b2-rfg                       <int>,<int> [ default: 5,3 ]
    --b2-score-min                 <func>      [ default: L,-0.6,-0.6 ]
  Effort options
    --b2-D                         <int>       [ default: 15 ]
    --b2-R                         <int>       [ default: 2 ]
Fusion related options:
    --fusion-search
    --fusion-anchor-length         <int>       [ default: 20 ]
    --fusion-min-dist              <int>       [ default: 10000000 ]
    --fusion-read-mismatches       <int>       [ default: 2 ]
    --fusion-multireads            <int>       [ default: 2 ]
    --fusion-multipairs            <int>       [ default: 2 ]
    --fusion-ignore-chromosomes    <list>      [ e.g, <chrM,chrX> ]
    --fusion-do-not-resolve-conflicts          [this is for test purposes ]
SAM Header Options (for embedding sequencing run metadata in output):
    --rg-id                        <string>    (read group ID)
    --rg-sample                    <string>    (sample ID)
    --rg-library                   <string>    (library ID)
    --rg-description               <string>    (descriptive string, no tabs allowed)
    --rg-platform-unit             <string>    (e.g Illumina lane ID)
    --rg-center                    <string>    (sequencing center name)
    --rg-date                      <string>    (ISO 8601 date of the sequencing run)
    --rg-platform                  <string>    (Sequencing platform descriptor)
    for detailed help see http://tophat.cbcb.umd.edu/manual.html

```

当使用TopHat处理双末端reads时 将 \*_1文件和 \*_2文件放到不同的以逗号分割的文件列表，两个列表中文件的顺序应当一致。
TopHat允许用户在配对reads之后使用附加的未配对的reads。TopHat在2.0.10版本之后支持混合格式输入，但是不推荐。


未完待续……

