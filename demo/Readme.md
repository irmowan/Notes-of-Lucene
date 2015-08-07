## Demo


该演示代码是重写的官方演示代码。

官方代码地址为:

[IndexFiles.java](https://lucene.apache.org/core/5_2_1/demo/src-html/org/apache/lucene/demo/IndexFiles.html)

[SearchFiles.java](https://lucene.apache.org/core/5_2_1/demo/src-html/org/apache/lucene/demo/SearchFiles.html)

---

#### 使用说明

以Eclipse为集成开发环境。  
首先，需要下载[*Lucene 5.2.1*](http://lucene.apache.org)（Maven导入也可），导入以下jar包：

* lucene-core-5.2.1.jar
* lucene-analyzers-common-5.2.1.jar
* lucene-queryparser-5.2.1.jar

创建工程，写入这两个文件。  
目录自定，此处参考官方代码，设置包为org.apache.lucene.demo。[^package]

[^package]:此时的文件目录即为src/org/apache/lucene/demo, 目录不符合时Eclipse会自动提示移动文件.

* **IndexFiles**	创建索引  
* **SearchFiles**	查询索引


两个文件各有一个main函数，执行时，需要携带参数表args.  
可以在Run菜单中，选择Run Configurations, main class指定IndexFiles或SearchFiles[^1]  
在Arguments中，输入Program Arguments后执行。

[^1]: 官方代码中，文件被打包在org.apache.lucene.demo中，则需要指定完整的包目录，如org.apache.lucene.demo.IndexFiles.

IndexFile参数表格式如下：

    java org.apache.lucene.demo.IndexFiles [-index INDEX_PATH] [-docs DOCS_PATH] [-update]
    
    // The document in DOCS_PATH, creating a Lucene index in INDEX_PATH that can be searched with SearchFiles.
    // e.g.: java IndexFiles -index D:\LuceneDemo\Index -docs D:\LuceneDemo\Data
    
SearchFile参数表格式如下：

	java org.apache.lucene.demo.SearchFiles [-index dir] [-field f] [-repeat n] [-queries file] [-query string] [-raw] [-paging hitsPerPage]
	
	//See http://lucene.apache.org/core/5_2_1/demo for details.
	// e.g.: java SearchFiles -index D:\LuceneDemo\Index -query Demo
	
	
执行后，即可在命令行输出中看到当前状态或查询结果。

2015年8月6日  
©copyright 慕瑜