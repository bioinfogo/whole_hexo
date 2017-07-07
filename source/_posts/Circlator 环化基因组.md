---
title: Circlator 环化拼接的基因组
date: 2017-07-07 09:35:05
categories: 基因组学

toc: true
---

# 介绍
Circlator 是一个自动化的基因组环化工具，其利用长序列作为数据基础，根据拼接的结果对序列进行环化，并修正其中可能的错误。
值得注意的是，该工具并不是ab inito 的拼接工具，而是处理拼接的结果，产生正确的环形DNA 结构。
其优势是，在甚至没有overlap 的情况下，也可以进行环化。
相关的文章发表在Genome Biology，详见[这里](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-015-0849-0)。

该程序通过将reads 比对到已有的拼接序列末端上，形成新的拼接，然后在其中鉴定可能的环化序列，并对其中可能的重复部分进行清除。最后，根据提供的基因，确定其起点。

# 安装与配置
## 依赖
该程序依赖以下的工具包：
1. BWA version >= 0.7.12
2. prodigal version >= 2.6
3. SAMtools (versions 0.1.9 to 1.3)
4. MUMmer version >= 3.23
5. Canu and/or SPAdes. SPAdes version 3.6.2 or higher is required, but 3.7.1 is recommended (marginally gave the best results on NCTC data from the Circlator publication, tested on all SPAdes versions 3.6.2-3.9.0).

其中 BWA 是常见的序列比对工具，prodigal 是原核生物的基因预测工具，SAMtools 是常用的比对和序列处理工具，MUMmer 是老牌的长序列比对工具，Canu/SPAdes 是三代测序的拼接工具。
## 安装
下载github中的最新release，然后解压后测试一下：
```
python3 setup.py test
```
如果没有问题的话，进行安装：
```
python3 setup.py install
#检查依赖包是否都可用？
circlator progcheck
```
在安装prodigal 过程中，如果遇到“FATAL: kernel too old”的错误，重新下载工具的source 文件，重新编译即可。
此外，要注意的是，python的版本不能低于3.0，否则会报错。如果系统版本低于3.0，可以重新安装一个新版的python
Python 的安装过程很简单，可以参考这个[教程](http://www.csuldw.com/2016/05/06/2016-05-06-python-and-pip/)。
## 实践与问题
在一个redhat服务器上安装的时候，在安装其中一个工具包pysam 的时候，总是报错：
> cc1: error: unrecognized command line option "-Wno-error=declaration-after-statement"

查阅相关的[解决方案](http://bugs.python.org/issue21121)，依然无法解决问题，所以无法配置。
而在另外一台较新的centOS服务器上配置的时候，除了遭遇过无权限安装python 包（更改prefix 即可）之外，没有遇到大的问题，比较顺利的完成。
推测还是因为gcc 的版本较低引起的问题，尽管在redhat上已经配置了gcc47。
# 程序运行
该程序需要的参数较少，简单来说，仅仅需要以下三项：
1. 拼接的基因组文件，fasta 格式
2. 修正过后的reads 文件，fasta或者fastq 格式均可。该文件在多数的三代测序分析工具如HGAP等中会产生，并作为中间文件。
3. 输出目录

该程序可以分阶段运行，如果运行"all" 的话，则是运行全部的流程。

默认情况下，该程序是采用SPAdes 进行拼接处理。
值得注意的是，在最后的起点锚定阶段，该程序默认采用dnaA的序列作为起点。但是该基因是一个细菌中较为保守且靠近复制起点的基因，如果分析的对象是真核生物该基因就不适用了。我采用的ATP6（真菌，cox2 也可以），其他物种请查资料确定。
该基因的序列可以利用以下参数输入：
> --genes_fa FILENAME

另外一个需要注意的问题是，这里要提供的拼接基因组序列，我认为最好预处理一下，对于真核生物而言，很多序列是已知的染色体序列片段，并不需要进行环化分析，可以剔除这些序列，仅仅保留那些可能的线粒体序列即可。
当然，对于具有环形基因组的原核生物则不需要处理。

一个常见的运行命令：
```
circlator all --threads 15 assembly.fa corrected.fastq output_circlator
```




