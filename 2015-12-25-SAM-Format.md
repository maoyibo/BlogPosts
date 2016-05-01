title: SAM 格式介绍
date: 2015-12-25 14:33:13
tags: 
    - SAM
    - Format
    - Unfinished
---
注：SAM格式说明文件（Sequence Alignment/Map Format Specification）[下载地址](https://github.com/samtools/hts-specs)。

SAM格式是以以TAB分割的文本格式，包括可选的Header和Alignment部分。

<!-- more -->  

# Header部分

__Header__ 部分如果出现，必须在Alignment部分的前面。Header的每一行都需要以'@'开始。在'@'之后是一个预定义的由两个字母组成的Header记录类型编码。除了@CO，每个Header记录类型的数据区都需要遵守'TAG:VALUE'格式，TAG同样由两个字符构成，第一个自负必须是字母，第二个可以是数字，TAG定义Value的格式和内容。每个每个Header都有必选的TAG。  
SAM定义了五种Header记录类型编码：  

- @HD   首行，如果出现则出现在文件的第一行。  
    + VN*   格式版本。  
- @SQ   参考序列字典。@SQ行根据其比对排序的顺序排序。  
    + SN*   参考序列名称。此处的值将出现在alignment部分的RNAME和RNEXT中。
    + LN*   参考序列长度。
- @RG   Read组。允许有多个未排序的@RG行。
    + ID*   Read组标识符 每一个@RG行都必须有一个唯一的ID。这个ID将在Aligmane部分中的RG tag被使用。  
- @PG 软件  
    + ID* 软件标识符。每一个@PG行都需要有一个唯一的ID。
- @CO   单行的文字描述。可以有多个未排序的@CO行。
    
注：'\*'标记的TAG为必选项  
可以自由的为新的数据区添加新的TAG，推荐自己添加的TAG使用小写字母表示。

# Alignment部分

__Alignment__ 部分的每一行表示一个片段的线性比对，不以'@'开始。每一行包括11个记录比对信息的必选字段（mandatory fields），和可变数目的可选字段。必选字段以规定的顺序出现，并且必须出现，当值不可取的时候可是'0'或'\*'。  
## 必选字段
- 1.QNAME：查询的模板名称。Reads或者Segments拥有相同的QNAME被认为是来自同一个模板。如果信息不可用则用'\*'表示。在一个SAM文件中，当嵌合比对或者多mappings出现，一个read可能占据多个alignment行。
- 2.FLAG：FLAG 按位的结合。每一个bit的解释如下:  

|DEC Bit|HEX Bit|Description|
|-|-|-|
|1|0x1|template having multiple segments in sequencing|
|2|0x2|each segment properly aligned according to the aligner|
|4|0x4|segment unmapped|
|8|0x8|next segment in the template unmapped|
|16|0x10|SEQ being reverse complemented|
|32|0x20|SEQ of the next segment in the template being reverse complemented|
|64|0x40|the first segment in the template|
|128|0x80|the last segment in the template|
|256|0x100|secondary alignment|
|512|0x200|not passing filters, such as platform/vendor quality controls|
|1024|0x400|PCR or optical duplicate|
|2048|0x800|supplementary alignment|

>对于SAM文件中的每一个read/contig，需要与之相关的行中有且只有一行满足'FLAG & 0x900==0'，即有且只有一个不满足'supplementary alignment' 和 'secondary alignment'，这行被称作read的 __'主要的行(primary line)'__ 。  
>没有出现在列表里面的位被留作以后使用。在当前的软件中他们不应该被设置，在读取中应该被忽略。  
>0x1 表示模板在测序中有多个片段，如果0x1未被设置，则不能假设0x2，0x8，0x20，0x40和0x80的值(即这些位的值为0)。  
>0x4 只有在表示read是否未被mapped被设置，如果0x4 被设置，则不能假设RNAME, POS, CIGAR, MAPQ, and bits 0x2, 0x100, 和 0x800的值(即这些位的值为0)。  
>0x10 表示SEQ和QUAL是否反转，当0x4未被设置，其片段比对上的序列相一致。当0x4被设置时表示这个未被mapped的read是否以测序仪输出的原始方向储存。
>0x40 0x80 如果同时被设置，这个片段是模板的一个线性部分，但是他既不是第一个也不是最后一个read。如果0x40 和 0x80没有设置，read在模板中的索引是未知的。这可能发生在一个非线性模板或者或者索引在数据操作中丢失的情况。  
>0x100 表示当使用的软件能够识别这个bit时，这个比对在确定的分析中将不会被使用。它通常使用在标记可变的mapping，当多mappings出现在SAM中。  
>0x800 表示相应的比对行是一个嵌合比对的一部分。一个标记0x800的行被称作一个附加行。
    
- 3.RNAME：比对的参考序列名称。如果@SQ header行出现，RNAME（如果不是'\*'）必须出现在一个@SQ的SN tag 中。一个没有mapped的片段没有坐标，在此位置使用'\*'。然而一个没有mapped的片段也可能又有一个普通的坐标，这样它可以在排序后放置在合适的位置。如果 RNAME 是'\*'，不能假设POS和CIGAR的值。  
- 4.POS：1-based 最左边第一个碱基的mapping位置。参考序列的第一个碱基的坐标是1。一个没有 mapped 的read 没有坐标，POS值为0 。如果POS值为0 ,则不能假设RNAME 和 CIGAR的值。
- 5.MAPQ：mapping的质量。它等于 $ -10log_{10}Pr $\{mapping position is wrong\}，取最近的整数。值255表示mapping的质量是不可取的。  
- 6.CIGAR：CIGAR字符串，具体信息查看下表，如果CIGAR值不可取设置为'\*'：

|Op|BAM|Description| 描述 |
|-|-|-|-|
|M|0|alignment match (can be a sequence match or mismatch)|比对匹配（可以是sequence match或mismatch）|
|I|1|insertion to the reference|插入参考序列 |
|D|2|deletion from the reference|删除参考序列|
|N|3|skipped region from the reference|跳过参考序列的区域|
|S|4|soft clipping (clipped sequences present in SEQ)|柔性剪切（剪切SEQ中的序列）|
|H|5|hard clipping (clipped sequences NOT present in SEQ)|硬性剪切（剪切非SEQ中的序列）|
|P|6|padding (silent deletion from padded reference)|填充（从填充参考中沉默剪切）|
|=|7|sequence match|序列匹配|
|X|8|sequence mismatch|序列不匹配|

>+ H 只出现在操作字符串的第一位或者最后一位。
>+ S 只可以有H操作在其和CIFAR末尾之间。
>+ 对于mRNA-to-genome比对，一个N操作表示内含子。对于其它类型的比对，N的解释是未定义的。
>+ M/I/S/=/X操作的总和应该等于SEQ的长度。

- 7.RNEXT：模板的下一个read的主要比对的参考序列名。对于一个模板的最后一个read，下一个read就是下一个模板的第一个read。如果@SQ header存在，RNEXT（如果不是'\*'）必须出现在一个@SQ的SN tag 中。如果该区域是'\*'表示该信息不可取，如果是'='表示RNEXT 与 RNAME 相同。如果不是'='而且模板中的下一个read有一个主要比对，这个区域和下一个read的主要比对的RNAME相同。如果RNAME是'\*'，则不能假定PNEXT 和 0x20的值。
- 8.PNEXT：模板中下一个read的主要比对的位置。如果该信息去可取的话其值为0。此处的值等于下一个read的主要比对的POS值。如果PNEXT值是0,则不能假定RNEXT 和 0x20的值。
- 9.TLEN：有符号的观察到的模板的长度。如果所有的片段比对到相同的参考序列，无符号的观察到的模板长度等于左边mapped到的碱基到右边mapped到的碱基的碱基数目。最左边的片段标记为正号，最右边的标记为负号，中间片段的符号未定义。如果信息不可用或者是单片段模板的话其值设为0。
- 10.SEQ：片段序列，如果序列没有被储存则表示为'\*'。如果一个'\*'，序列的长度必须等于CIGAR字符串中M/I/S/=/X操作符的长度。'='表示碱基和参考序列碱基一致。没有关于字母情况的假设。
- 11.QUAL：碱基质量加上33的ASCII码表示，如果质量没有储存的话可以用'\*'表示。如果此处不是'\*'，则SEQ必须不是'\*'，并且质量字符串的长度应该和SEQ的长度相等。

## 可选字段
所有的可选字段必须满足TAG:TYPE:VALUE的格式。TAG必须是以字母开始的两位字符。每个TAG在一行中只能出现一次。小写字符表示的TAG被留给最终用户。TYPE是一个大小写敏感的字符，定义VALUE的格式。

|Type|Regexp matching VALUE|Description|描述|
|-|-|-|-|
|A|[!-~]|Printable character|可打印的字符串|
|i|[-+]?[0-9]+|Signed integer |有符号的整数|
|f|[-+]?[0-9]*\.?[0-9]+([eE][-+]?[0-9]+)?|Single-precision floating number|单精度浮点数|
|Z|[ !-~]+|Printable string, including space|可打印的字符串，包括空格|
|H|[0-9A-F]+|Byte array in the Hex format |十六进制的字节数组|
|B|[cCsSiIf]\(,[-+]?[0-9]*\.?[0-9]+([eE][-+]?[0-9]+)?\)+|Integer or numeric array|整数或数字数组|





未完待续……







