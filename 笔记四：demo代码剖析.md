# 笔记四：Demo代码剖析

> 样例代码存放在demo文件夹内。  
> 使用说明详见样例代码的Readme.md.


## IndexFiles

```
for (int i = 0; i < args.length; ++i) {
	if ("-index".equals(args[i])) {
		indexPath = args[i + 1];
		i++;
	} else if ("-docs".equals(args[i])) {
		docsPath = args[i + 1];
		i++;
	} else if ("-update".equals(args[i])) {
		create = false;
	}
}
```

- 对输入参数进行字符串解析，获得相关量indexPath, docsPath, create.

```
Directory dir = FSDirectory.open(Paths.get(indexPath));
Analyzer analyzer = new StandardAnalyzer();
IndexWriterConfig iwc = new IndexWriterConfig(analyzer);
```

- 创建Directory对象，指向索引文件夹目录。
- 创建Analyzer对象，设置为标准分词器。事实上还有其他分词器，可以适用于不同语言、不同情形。
- 创建IndexWriterConfig对象，以分词器为配置内容。

```
if (create) {
	iwc.setOpenMode(OpenMode.CREATE);
} else {
	iwc.setOpenMode(OpenMode.CREATE_OR_APPEND);
}
```

- 由create布尔值判断是重做索引还是增加索引。（可以视为全量/增量）

```
IndexWriter writer = new IndexWriter(dir, iwc);
```

- 创建IndexWriter对象，准备写入索引。参数为路径以及配置。

```
indexDocs(writer, docDir);
```

- 调用indexDocs，本质是一个爬虫，爬遍路径中的所有文件。

```
Document doc = new Document();
doc.add(new StringField("path", file.toString(), Field.Store.YES));
doc.add(new LongField("modified", lastModified, Field.Store.NO));
doc.add(new TextField("contents", new BufferedReader(new InputStreamReader(stream, StandardCharsets.UTF_8))));
```

- 创建Document对象，创建Field对象。其中，Document相当于一张表，Field相当于表中的字段。
- `doc.add(field)`即为在表中添加不同字段。字段有其数据类型[^Field]，例如`StringField`, `LongField`, `TextField`。此处添加了三个字段，分别是*路径*，*最后修改时间*，*文本内容*。

[^Field]: [Class Field](https://lucene.apache.org/core/5_2_1/core/org/apache/lucene/document/Field.html)

```
if (writer.getConfig().getOpenMode() == OpenMode.CREATE) {
	System.out.println("adding " + file);
	writer.addDocument(doc);
} else {
	System.out.println("updating " + file);
	writer.updateDocument(new Term("path", file.toString()), doc);
}
```

- 最后根据OpenMode决定是创建还是更新文档索引。
- `writer.addDocument(doc)`即为实质写索引过程。
- `writer.updateDocument(term, doc)`同理，更新文档索引。

## SearchFiles

*参数表：*

- `[-index dir]`索引路径
- `[-field f]` 查询字段，默认为contents
- `[-repeat n]` 重复n次查询，测试查询性能
- `[-queries file]`	以文档方式存储的批量查询语句
- `[-query string]`	单条查询语句，可用&等符号连接查询条件
- `[-raw]`	搜索结果原始信息，会显示文档得分
- `[-paging hitsPerPage]`   每页展示的命中数，默认为10.

```
IndexReader reader = DirectoryReader.open(FSDirectory.open(Paths.get(index)));
IndexSearcher searcher = new IndexSearcher(reader);
Analyzer analyzer = new StandardAnalyzer();
```
- 创建IndexReader对象，指向索引文件夹目录。
- 创建IndexSearcher对象，对IndexReader搜索。
- 创建Analyzer对象，设置为标准分词器。

```
BufferedReader in = null;
if (queries != null) {
	in = Files.newBufferedReader(Paths.get(queries), StandardCharsets.UTF_8);
} else {
	in = new BufferedReader(new InputStreamReader(System.in,StandardCharsets.UTF_8));
}
``` 
- 设置输入流in

```
QueryParser parser = new QueryParser(field, analyzer);
```
- 创建QueryParser对象，对field进行语法分析。

```
if (queries == null && queryString == null) {
	System.out.println("Enter query: ");
}
String line = queryString != null ? queryString : in.readLine();
if (line == null || line.length() == -1) {
	break;
}
line = line.trim();	
if (line.length() == 0) {
	break;
}
```
- 提醒输入查询语句。
- 获得line语句，得到查询字符串。
- `line = line.trim();`对字符按串进行修整，去掉头尾的空格。

```
Query query = parser.parse(line);
System.out.println("Search for: " + query.toString(field));
```
- 生成Query对象，准备查询。

```
if (repeat > 0) {
	Date start = new Date();
	for (int i = 0; i < repeat; ++i) {
		searcher.search(query, 100);
	}
	Date end = new Date();
	System.out.println("Time: " + (end.getTime() - start.getTime()) + "ms");
}
```

- `Searcher.search(Query, int)`对Query执行查询操作，返回前int个命中结果。
- 这一段代码，乍一看没有任何意义，search返回的结果没有保存，直接输出了总时间。
- 这是一段性能测试，对于同一查询，通过大量重复查询操作，测试它的平均查询时间。

```
doPagingSearch(in, searcher, query, hitsPerPage, raw, queries == null && queryString == null);
```
- 调用`doPagingSearch`方法查询并分页显示。

```
TopDocs results = searcher.search(query, 5 * hitsPerPage);
ScoreDoc[] hits = results.scoreDocs;
```
- 创建TopDocs保存查询结果
- scoreDocs是类TopDocs的一个Field，本质是一个ScoreDoc数组。[^TopDocs]
- ScoreDoc记录文档评分。[^ScoreDOc]

[^TopDocs]:[Class TopDocs](https://lucene.apache.org/core/5_2_1/core/org/apache/lucene/search/TopDocs.html)
[^ScoreDoc]:[Class ScoreDoc](https://lucene.apache.org/core/5_2_1/core/org/apache/lucene/search/ScoreDoc.html)

```
if (raw) {
	System.out.println("doc= " + hits[i].doc + " score=" + hits[i].score);
	continue;
}
```
- 这里是在raw状态下，输出原始信息。

```
Document doc = searcher.doc(hits[i].doc);
String path = doc.get("path");
if (path != null) {
	System.out.println((i + 1) + ". " + path);
	String title = doc.get("title");
	if (title != null) {
		System.out.println("   Title: " + doc.get("title"));
	}
} else {
	System.out.println((i + 1) + ". " + "No path for this document");
}
```
- 这里即为输出格式化的结果。
- 其它代码主要为分页显示的格式化所用，不再冗述。

2015年8月7日  
©copyright 慕瑜