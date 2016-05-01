title: 使用 Bowtie2 将reads 比对到参考基因组
date: 2015-12-24 14:27:23
tags:
    - NGS
    - Mapper
    - Manual
    - Commands
---
注：本例使用的bowtie2 版本为：version 2.2.6
bowtie2的使用需要两种输入文件：
**Reference genome data (fasta格式 .fa, .fasta, .fna)**  
本文使用的例子是Escherichia coli 参考基因组（gi|15829254|ref|NC_002695.1| Escherichia coli O157:H7 str. Sakai chromosome, complete genome）。  
文件下载后命名为ecoli.fa

<!-- more -->  

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


## 使用 bowtie2-build 对参考基因组建立索引  

bowtie2-build 参数：
```bash
$ bowtie2-build -h
Bowtie 2 version 2.2.6 by Ben Langmead (langmea@cs.jhu.edu, www.cs.jhu.edu/~langmea)
Usage: bowtie2-build [options]* <reference_in> <bt2_index_base>
    reference_in            comma-separated list of files with ref sequences
                                    #使用逗号分割的参考序列文件
    bt2_index_base          write bt2 data to files with this dir/basename
                                    #bt2数据输出文件
*** Bowtie 2 indexes work only with v2 (not v1).  Likewise for v1 indexes. ***
Options:
    -f                      reference files are Fasta (default)
    -c                      reference sequences given on cmd line (as <reference_in>)
    --large-index           force generated index to be 'large', even if ref has fewer than 4 billion nucleotides
    -a/--noauto             disable automatic -p/--bmax/--dcv memory-fitting
    -p/--packed             use packed strings internally; slower, less memory
    --bmax <int>            max bucket sz for blockwise suffix-array builder
    --bmaxdivn <int>        max bucket sz as divisor of ref len (default: 4)
    --dcv <int>             diff-cover period for blockwise (default: 1024)
    --nodc                  disable diff-cover (algorithm becomes quadratic)
    -r/--noref              don't build .3/.4 index files
    -3/--justref            just build .3/.4 index files
    -o/--offrate <int>      SA is sampled every 2^<int> BWT chars (default: 5)
    -t/--ftabchars <int>    # of chars consumed in initial lookup (default: 10)
    --seed <int>            seed for random number generator
    -q/--quiet              verbose output (for debugging)
    -h/--help               print detailed description of tool and its options
    --usage                 print this usage message
    --version               print version information and quit

```

本例使用命令：
```bash
$ bowtie2-build ecoli.fa ecoli
```
输出索引文件有6个：  
ecoli.1.bt2，ecoli.2.bt2，ecoli.3.bt2，ecoli.4.bt2，ecoli.rev.1.bt2，ecoli.rev.2.bt2

## 使用 bowtie2 将reads 比对到参考基因组

bowtie2  参数
```bash
$ bowtie2 -h
Bowtie 2 version 2.2.6 by Ben Langmead (langmea@cs.jhu.edu, www.cs.jhu.edu/~langmea)
Usage: 
  bowtie2 [options]* -x <bt2-idx> {-1 <m1> -2 <m2> | -U <r>} [-S <sam>]

  <bt2-idx>  Index filename prefix (minus trailing .X.bt2).
             NOTE: Bowtie 1 and Bowtie 2 indexes are not compatible.
  <m1>       Files with #1 mates, paired with files in <m2>.
             Could be gzip'ed (extension: .gz) or bzip2'ed (extension: .bz2).
  <m2>       Files with #2 mates, paired with files in <m1>.
             Could be gzip'ed (extension: .gz) or bzip2'ed (extension: .bz2).
  <r>        Files with unpaired reads.
             Could be gzip'ed (extension: .gz) or bzip2'ed (extension: .bz2).
  <sam>      File for SAM output (default: stdout)

  <m1>, <m2>, <r> can be comma-separated lists (no whitespace) and can be
  specified many times.  E.g. '-U file1.fq,file2.fq -U file3.fq'.

Options (defaults in parentheses):

 Input:
  -q                 query input files are FASTQ .fq/.fastq (default)
  --qseq             query input files are in Illumina's qseq format
  -f                 query input files are (multi-)FASTA .fa/.mfa
  -r                 query input files are raw one-sequence-per-line
  -c                 <m1>, <m2>, <r> are sequences themselves, not files
  -s/--skip <int>    skip the first <int> reads/pairs in the input (none)
  -u/--upto <int>    stop after first <int> reads/pairs (no limit)
  -5/--trim5 <int>   trim <int> bases from 5'/left end of reads (0)
  -3/--trim3 <int>   trim <int> bases from 3'/right end of reads (0)
  --phred33          qualities are Phred+33 (default)
  --phred64          qualities are Phred+64
  --int-quals        qualities encoded as space-delimited integers

 Presets:                 Same as:
  For --end-to-end:
   --very-fast            -D 5 -R 1 -N 0 -L 22 -i S,0,2.50
   --fast                 -D 10 -R 2 -N 0 -L 22 -i S,0,2.50
   --sensitive            -D 15 -R 2 -N 0 -L 22 -i S,1,1.15 (default)
   --very-sensitive       -D 20 -R 3 -N 0 -L 20 -i S,1,0.50

  For --local:
   --very-fast-local      -D 5 -R 1 -N 0 -L 25 -i S,1,2.00
   --fast-local           -D 10 -R 2 -N 0 -L 22 -i S,1,1.75
   --sensitive-local      -D 15 -R 2 -N 0 -L 20 -i S,1,0.75 (default)
   --very-sensitive-local -D 20 -R 3 -N 0 -L 20 -i S,1,0.50

 Alignment:
  -N <int>           max # mismatches in seed alignment; can be 0 or 1 (0)
  -L <int>           length of seed substrings; must be >3, <32 (22)
  -i <func>          interval between seed substrings w/r/t read len (S,1,1.15)
  --n-ceil <func>    func for max # non-A/C/G/Ts permitted in aln (L,0,0.15)
  --dpad <int>       include <int> extra ref chars on sides of DP table (15)
  --gbar <int>       disallow gaps within <int> nucs of read extremes (4)
  --ignore-quals     treat all quality values as 30 on Phred scale (off)
  --nofw             do not align forward (original) version of read (off)
  --norc             do not align reverse-complement version of read (off)
  --no-1mm-upfront   do not allow 1 mismatch alignments before attempting to
                     scan for the optimal seeded alignments
  --end-to-end       entire read must align; no clipping (on)
   OR
  --local            local alignment; ends might be soft clipped (off)

 Scoring:
  --ma <int>         match bonus (0 for --end-to-end, 2 for --local) 
  --mp <int>         max penalty for mismatch; lower qual = lower penalty (6)
  --np <int>         penalty for non-A/C/G/Ts in read/ref (1)
  --rdg <int>,<int>  read gap open, extend penalties (5,3)
  --rfg <int>,<int>  reference gap open, extend penalties (5,3)
  --score-min <func> min acceptable alignment score w/r/t read length
                     (G,20,8 for local, L,-0.6,-0.6 for end-to-end)

 Reporting:
  (default)          look for multiple alignments, report best, with MAPQ
   OR
  -k <int>           report up to <int> alns per read; MAPQ not meaningful
   OR
  -a/--all           report all alignments; very slow, MAPQ not meaningful

 Effort:
  -D <int>           give up extending after <int> failed extends in a row (15)
  -R <int>           for reads w/ repetitive seeds, try <int> sets of seeds (2)

 Paired-end:
  -I/--minins <int>  minimum fragment length (0)
  -X/--maxins <int>  maximum fragment length (500)
  --fr/--rf/--ff     -1, -2 mates align fw/rev, rev/fw, fw/fw (--fr)
  --no-mixed         suppress unpaired alignments for paired reads
  --no-discordant    suppress discordant alignments for paired reads
  --no-dovetail      not concordant when mates extend past each other
  --no-contain       not concordant when one mate alignment contains other
  --no-overlap       not concordant when mates overlap at all

 Output:
  -t/--time          print wall-clock time taken by search phases
  --un <path>           write unpaired reads that didn't align to <path>
  --al <path>           write unpaired reads that aligned at least once to <path>
  --un-conc <path>      write pairs that didn't align concordantly to <path>
  --al-conc <path>      write pairs that aligned concordantly at least once to <path>
  (Note: for --un, --al, --un-conc, or --al-conc, add '-gz' to the option name, e.g.
  --un-gz <path>, to gzip compress output, or add '-bz2' to bzip2 compress output.)
  --quiet            print nothing to stderr except serious errors
  --met-file <path>  send metrics to file at <path> (off)
  --met-stderr       send metrics to stderr (off)
  --met <int>        report internal counters & metrics every <int> secs (1)
  --no-unal          supppress SAM records for unaligned reads
  --no-head          supppress header lines, i.e. lines starting with @
  --no-sq            supppress @SQ header lines
  --rg-id <text>     set read group id, reflected in @RG line and RG:Z: opt field
  --rg <text>        add <text> ("lab:value") to @RG line of SAM header.
                     Note: @RG line only printed when --rg-id is set.
  --omit-sec-seq     put '*' in SEQ and QUAL fields for secondary alignments.

 Performance:
  -p/--threads <int> number of alignment threads to launch (1)
  --reorder          force SAM output order to match order of input reads
  --mm               use memory-mapped I/O for index; many 'bowtie's can share

 Other:
  --qc-filter        filter out reads that are bad according to QSEQ filter
  --seed <int>       seed for random number generator (0)
  --non-deterministic seed rand. gen. arbitrarily instead of using read attributes
  --version          print version information and quit
  -h/--help          print this usage message

```

bowtie2 默认执行 end-to-end read 比对，且本例使用Paired-end reads 使用命令
```bash
$ bowtie2 -x ecoli -1 SRR2982241_1_filtered.fastq -2 SRR2982241_2_filtered.fastq -S SRR2982241.ETE.sam
#程序输出
1340783 reads; of these:
  1340783 (100.00%) were paired; of these:
    967025 (72.12%) aligned concordantly 0 times
    357153 (26.64%) aligned concordantly exactly 1 time
    16605 (1.24%) aligned concordantly >1 times
    ----
    967025 pairs aligned concordantly 0 times; of these:
      485924 (50.25%) aligned discordantly 1 time
    ----
    481101 pairs aligned 0 times concordantly or discordantly; of these:
      962202 mates make up the pairs; of these:
        834252 (86.70%) aligned 0 times
        73901 (7.68%) aligned exactly 1 time
        54049 (5.62%) aligned >1 times
68.89% overall alignment rate

```

bowtie2 执行 Local read 比对，使用命令
```bash
$ bowtie2 --local -x ecoli -1 SRR2982241_1_filtered.fastq -2 SRR2982241_2_filtered.fastq -S SRR2982241.LOCAL.sam
#程序输出
1340783 reads; of these:
  1340783 (100.00%) were paired; of these:
    918570 (68.51%) aligned concordantly 0 times
    384568 (28.68%) aligned concordantly exactly 1 time
    37645 (2.81%) aligned concordantly >1 times
    ----
    918570 pairs aligned concordantly 0 times; of these:
      492349 (53.60%) aligned discordantly 1 time
    ----
    426221 pairs aligned 0 times concordantly or discordantly; of these:
      852442 mates make up the pairs; of these:
        724379 (84.98%) aligned 0 times
        44099 (5.17%) aligned exactly 1 time
        83964 (9.85%) aligned >1 times
72.99% overall alignment rate
```

对与Single-end reads 使用命令：
```bash
#End-to-end:
bowtie2 -x ecoli -U SRR2982241.fastq -S SRR2982241.ETE.sam

#Local:
bowtie2 --local -x ecoli -U SRR2982241.fastq -S SRR2982241.LOCAL.sam
```