#笔记三：Lucene的索引文件格式
### 基本概念

>参考[Lucene 5.0 File Format](http://lucene.apache.org/core/5_2_0/core/org/apache/lucene/codecs/lucene50/package-summary.html)，不同版本的文件格式有一定差异。

**术语层次：**

* **索引** *Index*：一个索引置于一个文件夹中，同一文件夹的所有文件构成一个索引，一个索引包含了多个文档
* **段** *Segment*：Index可能由多个Sub-Index, 也即Segment组成，每个Segment本质上是一个独立索引，可以被单独检索，一般由新增文档或合并已有的Segment产生
* **文档** *Document*：一个文档包含了多个域 
* **域** *Field*：一个域包含了一系列词
* **词** *Term*：词由一系列byte组成

**每个Segment索引包含了下列信息：[^structure]**

[^structure]:[*Index Structure Overview*](http://lucene.apache.org/core/5_2_0/core/org/apache/lucene/codecs/lucene50/package-summary.html#Overview)

* 段信息 *Segment info*
* 域名称 *Field names*
* 存储的域的值 *Stored Field values*
* 词典 *Term dictionary*
* 词频数据 *Term Frequency data*
* 位置信息 *Term Proximity data*
* 标准化因子 *Normalizaiton factors*
* 文档向量 *Term Vectors*
* 文档信息概览 *Per-document alues*
* 存活文档 *Live documents*

一些规范：

* 同一Segment的文件，共用相同的名称，有不同扩展名。[^Extensions]  
* 一个Index中的所有Segment在同一目录下。  
* 文档名依照数字顺序排列，从0开始。

[^Extensions]:[*Summary of File Extensions*](http://lucene.apache.org/core/5_2_0/core/org/apache/lucene/codecs/lucene50/package-summary.html#file-names)

### 编码技巧

###### 前缀编码Prefix
相同的前缀不重复编码，保存前缀在词中的偏移量，以及后缀。
###### 差值规则Delta
文档的编号等，数字会很大。利用保存前后差值，避免直接保存大整数。
###### 跟随判断
索引结构中，可能某个值A后跟着另一个值B，也可能不跟，如果单独存储这个信息，需要一个Byte。可以通过把A左移一位，用最后一个Bit去保存这个信息，更节约空间。  
是否采用这样的规则，作为配置信息保存在配置文件中。
###### 跳跃表Skip list
跳跃表是如图所示的一种数据结构。其具有下列特点：

* 元素按序排列。在Lucene中，按字典序或从小到大排列。
* 有间隔*interval*。每次跳跃的元素数是事先配置好的间隔。
* 有层次*level*。跳跃表本身可以构建多层。

![image](img/跳跃表.jpg)

<br />

*注：*

* 一些术语定义可能有差异，如间隔数，层次数。
* Lucene的具体实现也与理论有一定不同。

### 其他

关于索引文件中的具体数据类型，以及各文件如何保存索引信息，比较繁杂，不再冗述。[觉先的博客](http://www.cnblogs.com/forfuture1978/archive/2010/06/13/1757479.html)对此阐述较为详细。有需求时可以一览。

<br />

2015年8月6日  
©copyright 慕瑜