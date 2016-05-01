title: VCF 4.1版本 格式介绍
date: 2015-12-28 16:54:56
tags: 
    - VCF
    - Format
    - Unfinished
---
注：VCF 4.1版本 格式说明文件（The Variant Call Format (VCF) Version 4.1 Specification）[下载地址](https://github.com/samtools/hts-specs)。

VCF 是一个文本文件格式，通常以压缩文件格式储存。包含元信息行，一个header行，然后是数据行，每个数据行包含一个基因组上位置的信息。VCF 格式也能够包含每个位置样品的基因型的信息。

<!-- more -->  

## 元信息行
文件的元信息行以##开始，而且必须是键=值对。推荐在元信息部分包含INFO, FILTER 和 FORMAT等出现在VCF文件body部分的条目的描述信息行。虽然这些是可选的，但是如果他们出现则需要被完全符合规则。
#### fileformat
一个单独的文件格式区域是必须的，而且必须出现在文件的第一行，描述VCF格式的版本。此行应该是：
```
##fileformat=VCFv4.1
```
### INFO
INFO 区域应该应该如下描述（所有的键都是必须的）：
```
##INFO=<ID=ID,Number=number,Type=type,Description="description">
```
INFO区域的类型可能是：整型，浮点，标记，符号和字符串。Number条目的值是整数，表示INFO区域可以包含的值的个数，如果INFO区域包含一个单独的数字则Number条目为'1'，如果包含一对数字则为'2' ，如果对于每一个等位基因均有一个值则应为'A'，如果对于每一个可能的基因型均有一个值，则应为'G'，如果可能值的个数是变化的，或者未帮顶的，应为'.'。'Flag'类型表示INFO区域不包含一个值条目，Number条目应该是0。Description值应该被双引号包围，双引号中的双引号应使用\转意。
### FILTER
应用于数据的过滤方法应该如下描述：
```
##FILTER=<ID=ID,Description="description">
```
### FORMAT
在FORMAT区域指定的基因型信息应该如下描述：
```
##FORMAT=<ID=ID,Number=number,Type=type,Description="description">
```
FORMAT区域可能的类型是：整数，浮点，字符和字符串。

### ALT

```
##ALT=<ID=type,Description=description>
```

## Header