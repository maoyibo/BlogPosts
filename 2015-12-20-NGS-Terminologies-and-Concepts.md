title: NGS Terminologies and Concepts
date: 2015-12-20 17:04:40
tags: 
    - NGS
    - Terminologies
    - Concepts
---
1. __Template__  
    A DNA/RNA sequence part of which is sequenced on a sequencing machine or assembled from raw sequences.  
    一个DNA/RNA序列，此序列可以通过测序仪测出或者由原始序列组装而成。   
    _注：此Template就是分子生物学意义上的DNA模板或RNA模板，和参考序列没有关系。_
    _source:Sequence Alignment/Map Format Specification_  <!-- more -->  
2. __Segment__  
    A contiguous sequence or subsequence.  
    一个连续的序列或者子序列。  
    _source:Sequence Alignment/Map Format Specification_
3. __Read__  
    A raw sequence that comes off a sequencing machine. A read may consist of multiple segments. For sequencing data, reads are indexed by the order in which they are sequenced.  
    一个从测序仪输出的原始序列。一个read可以由多个segment构成。对于测序数据，reads通过他们被测序的顺序索引。  
    _source:Sequence Alignment/Map Format Specification_
4. __Linear alignment__  
    An alignment of a read to a single reference sequence that may include insertions, deletions, skips and clipping, but may not include direction changes (i.e. one portion of the alignment on forward strand and another portion of alignment on reverse strand). A linear alignment can be represented in a single SAM record.  
    将一个read比对到一个单一的参考序列可能包括插入、删除、跳跃和裁剪，但是不会包括方向的改变（比如：比对的一部分在正向链，另外一部分在反向链）。一个线性比对可以使用单一的SAM记录表示。  
    _source:Sequence Alignment/Map Format Specification_  
5. __Chimeric alignment__  
    An alignment of a read that cannot be represented as a linear alignment. A chimeric alignment is represented as a set of linear alignments that do not have large overlaps. Typically, one of the linear alignments in a chimeric alignment is considered the “representative” alignment, and the others are called “supplementary” and are distinguished by the supplementary alignment flag. All the SAM records in a chimeric alignment have the same QNAME and the same values for 0x40 and 0x80 flags. The decision regarding which linear alignment is representative is arbitrary.  
    一个read的比对不能被表示为一个线性比对。一个嵌合比对表现为一组没有大的重叠的线性比对。一般情况下，嵌合比对中的一个线性比对被认为是“代表性比对”，另外一个被认为是“补充比对”，两者通过附加的比对标签区别。一个嵌合比对的所有SAM记录都有相同的QNAME以及相同的 0x40 和 0x80 标签值。决定哪一个线性比对是代表性比对是随意的。  
    _source:Sequence Alignment/Map Format Specification_
6. __Read alignment__  
    A linear alignment or a chimeric alignment that is the complete representation of the alignment of the read.  
    完全表示read比对的一个线性比对或者嵌合比对。  
    _source:Sequence Alignment/Map Format Specification_
7. __Multiple mapping__  
    The correct placement of a read may be ambiguous, e.g. due to repeats. In this case, there may be multiple read alignments for the same read. One of these alignments is considered primary. All the other alignments have the secondary alignment flag set in the SAM records that represent them. All the SAM records have the same QNAME and the same values for 0x40 and 0x80 flags. Typically the alignment designated primary is the best alignment, but the decision may be arbitrary.   
    由于重复等原因，read的正确位置可能是模棱两可的。在这种情况下，对于相同的read可能有多个比对方式。这些比对方式中的一个被考虑为主要的，所有其他的比对方式在SAM记录中拥有次要比对标签集来表示他们。所有的SAM记录拥有相同的QNAME和 0x40 与 0x80 的值。一般情况下首要的比对方案是最优的比对方案，但是决定可能是随意的。  
    _source:Sequence Alignment/Map Format Specification_
8. __1-based coordinate system__  
    A coordinate system where the first base of a sequence is one. In this coordinate system, a region is specified by a closed interval. For example, the region between the 3rd and the 7th bases inclusive is [3, 7]. The SAM, VCF, GFF and Wiggle formats are using the 1-based coordinate system.  
    一个坐标系统，其中序列的第一个碱基是1，在这个坐标系统中，一个区域通过一个封闭区间指定。比如，在第三和第七碱基首尾之间的区域表示为[3,7]。SAM，VCF，GFF和Wiggle格式使用1-based 坐标系统。  
    _注：在0-based坐标系统中第一个碱基的编号是1_  
    _source:Sequence Alignment/Map Format Specification_
9. __0-based coordinate system__  
    A coordinate system where the first base of a sequence is zero. In this coordinate system, a region is specified by a half-closed-half-open interval. For example, the region between the 3rd and the 7th bases inclusive is [2, 7). The BAM, BCFv2, BED, and PSL formats are using the 0-based coordinate system.  
    一个坐标系统，其中序列的第一个碱基是0，在这个坐标系统中，一个区域通过一个半封闭半开放的区间指定。比如，在第三和第七碱基首尾之间的区域表示为[2,7)。BAM, BCFv2, BED, 和 PSL formats使用0-based 坐标系统。   
    _注：在0-based坐标系统中第一个碱基的编号是0_  
    _source:Sequence Alignment/Map Format Specification_
10. __Phred scale__  
    Given a probability {% math %}0 < p \le 1{% endmath %}, the phred scale of p equals {% math %}-10log_{10}p{% endmath %}, rounded to the closest integer.  
    对于给定的概率 {% math %}0 < p \le 1{% endmath %}，p 的 phred 刻度等于 {% math %}-10log_{10}p{% endmath %}，取最近的整数。  
    _注：和["Phred quality score"][phred]意思相同。_  
    _source:Sequence Alignment/Map Format Specification_  


未完待续……
 
[phred]:https://en.wikipedia.org/wiki/Phred_quality_score  "Phred quality score"
