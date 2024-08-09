---
title: Lucene search engine
date: 2024-08-10
tags: 
    - Lucene
categories: Summary
copyright: true
---

> :pushpin:**Elasticsearch**是一个基于**Lucene**的搜索引擎，提供了一个分布式多用户能力的全文搜索引擎，基于`RESTful API`接口，解决了原生`Lucene`使用上的缺陷。为了更好的理解`Elasticsearch`，先对`Lucene`进行了解。

<!--more-->

## :books: 全文检索

- 结构化数据，具有固定格式或有限长度的数据，如数据库表中的字段和元数据。
  - 数据库用`SQL`语句搜索；元数据用类型、属性、标签等搜索。

- 非结构化数据，不定长或无固定格式的数据，如邮件、Word 文档、PDF 文档等，又称**全文数据**。
  - **顺序扫描法(Serial Scanning)**，对每个文档从头到尾扫描，非结构化数据在数据量大时效率低下，不适合全文检索。
  - **全文检索(Full-text Search)**，把非结构化数据转换为结构化数据，再用结构化数据的检索方法。首先将要查询的目标数据中的词提取出来，组成索引，通过查询索引达到搜索目标数据的目的。

StringField 和 TextField 是 Apache Lucene 中用于存储和索引文档字段的两种不同类型。  

- StringField：用于存储不需要进行全文搜索的字段。它不会被分词器处理，适用于精确匹配查询。例如，ID、日期、分类等字段。
- TextField：用于存储需要进行全文搜索的字段。它会被分词器处理，适用于需要进行全文搜索的文本内容。例如，文章内容、描述等字段。
示例代码：

## :mag_right:`Lucene`基本原理

### 倒排索引(Inverted index)

> 快速查询某个词在文档中的位置，将文档集合中的每个词都映射到出现在该次的文档列表，而不是将文档映射到词的列表

1. 文档分词：将文档进行分词，得到词项列表，这里可以还有大小写转化、去停用词等操作。
2. 构建索引表：将词项列表构建倒排索引表，记录每个词项出现在哪些文档中。遍历所有文档，对每个词将其映射到文档列表，可以用哈希表、树等数据结构。
3. 优化索引：对索引表进行优化，如压缩、排序、合并等操作。
4. 查询处理：用户输入一个检索词时，对查询进行分词，得到查询词项列表，然后在索引表中查找每个词项对应的文档列表，最后对文档列表进行筛选、排序、交集、并集等操作，得到最终的查询结果。

### 段落索引(Segment index)

> 为了提高搜索效率，将索引分为多个段落，每个段落都是一个独立的索引单元，每个段落都有自己的倒排索引表。

1. 分割文本：分词器的组件将文本分割成若干个段落，可以按照具体的需求来定义分割规则，例如空格、标点符号等。
2. 建立词汇表：Lucene对词汇表进行排序等以提高搜索效率。
3. 构建倒排索引：对每个段落构建倒排索引表，记录每个词项出现在哪些文档中。倒排索引的数据结构包括索引词项(Term)、文档编号(Document ID)、位置(Position)等。
4. 搜索：用户输入一个检索词时，Lucene在词汇表中查找，并获取到包含该单词的段落列表，然后根据这些段落列表计算相关度得分，并按照相关度排序返回搜索结果。

### 文本预处理

> 在构建索引之前，需要对文本进行预处理，包括分词、大小写转化、去停用词等操作。

1. 小写转换(Lowercasing)：将文本转换为小写，避免大小写不一致导致的搜索问题。
2. 去除停用词(Stopwords)：去除常用词，如“的”、“是”、“在”、“and”、“the”等，避免这些词对搜索结果的影响。
3. 词根提取(Stemming)：将词汇还原为词根，如“running”还原为“run”。
4. 同义词扩展(Synonyms)：将同义词映射到同一个词项，如“car”和“automobile”映射到同一个词项。

## :bulb:搜索算法

### BM25算法

> BM25算法是一种基于概率检索模型的搜索算法，它是一种基于词频和文档长度的搜索算法，适用于大型文本数据集的搜索，考虑了文档中各项词的权重、词频、文档长度等因素，根据**文档和查询**之间的关系来评估文档的相关性，并按照相关性进行排序。

评分公式：$$\text{score}(D, Q) = \sum_{i=1}^{n} \text{IDF}(q_i) \cdot \frac{f(q_i, D) \cdot (k_1 + 1)}{f(q_i, D) + k_1 \cdot (1 - b + b \cdot \frac{|D|}{\text{avgdl}})}$$

其中：$D$表示文档，$Q$表示查询，$n$表示查询中的词项数量，$q_i$表示查询中的第$i$个词项，$f(q_i, D)$表示词项$q_i$在文档$D$中的词频，$k_1$和$b$是调节参数，$avgdl$表示文档集合的平均长度。

### TF-IDF算法(Term Frequency-Inverse Document Frequency)

> TF-IDF算法是一种用于评估一个词对于一个文件集或一个语料库中某个文件的重要程度的统计方法，基本思想是如果某个词在一篇文章中出现的频率高，并且在其他文章中出现的频率低，则认为这个词具有很好的区分能力，是这篇文章的关键词。

- TF(Term Frequency)：词频，表示某个词在文档中出现的频率，计算公式为$$\text{TF}(t, d) = \frac{f(t, d)}{\sum_{t' \in d} f(t', d)}$$
- IDF(Inverse Document Frequency)：逆文档频率，表示某个词在文档集合中的区分能力，计算公式为$$\text{IDF}(t, D) = \log \frac{N}{\text{DF}(t, D)}$$
- TF-IDF：综合考虑词频和逆文档频率，计算公式为$$\text{TF-IDF}(t, d, D) = \text{TF}(t, d) \cdot \text{IDF}(t, D)$$

### 空间向量模型(Vector Space Model)

> 空间向量模型是一种基于向量空间的搜索算法，将文档和查询表示为向量，通过计算向量之间的相似度来评估文档的相关性。

#### 余弦相似度(Cosine Similarity)

- 基于向量的夹角来衡量文档和查询的相似度，计算文档向量和查询向量的内积除以两个向量的模长的乘积，计算公式为$$\text{Cosine Similarity}(D, Q) = \frac{D \cdot Q}{\|D\| \cdot \|Q\|}$$，其中$D$表示文档向量，$Q$表示查询向量，取值范围为$[-1, 1]$，值越大表示相似度越高。

#### Jaccard相似度(Jaccard Similarity)

- 基于向量的交集和并集来衡量文档和查询的相似度，计算文档向量和查询向量的交集除以两个向量的并集，计算公式为$$\text{Jaccard Similarity}(D, Q) = \frac{|D \cap Q|}{|D \cup Q|}$$，取值范围为$[0, 1]$，值越大表示相似度越高。

## :computer:Coding

### 导入依赖

``` xml
        <!-- lucene核心包 -->
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-core</artifactId>
            <version>${lunece.version}</version>
        </dependency>
        <!-- lucene查询解析包 -->
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-queryparser</artifactId>
            <version>${lunece.version}</version>
        </dependency>
        <!-- lucene分词器包 -->
        <dependency>
            <groupId>org.apache.lucene</groupId>
            <artifactId>lucene-analyzers-common</artifactId>
            <version>${lunece.version}</version>
        </dependency>
        <!-- test -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
        </dependency>
```

### 从文件构建索引和检索

#### 文档

- 在资源目录下创建一个`data`目录，用于存放文档数据，这里用了雅思的三段话作为示例数据$1.txt$，$2.txt$，$3.txt$。

``` bash
mkdir -p ~/JavaProjects/luceneDemo/src/main/resources/data

echo "Many procedures are available for obtaining data about a language. They range from a carefully planned, intensive field investigation in a foreign country to a casual introspection about one’s mother tongue carried out in an armchair at home." > ~/JavaProjects/luceneDemo/src/main/resources/data/1.txt

echo "In all cases, someone has to act as a source of language data – an informant. Informants are (ideally) native speakers of a language, who provide utterances for analysis and other kinds of information about the language (e.g. translations, comments about correctness, or judgements on usage). Often, when studying their mother tongue, linguists act as their own informants, judging the ambiguity, acceptability, or other properties of utterances against their own intuitions. The convenience of this approach makes it widely used, and it is considered the norm in the generative approach to linguistics. But a linguist’s personal judgements are often uncertain, or disagree with the judgements of other linguists, at which point recourse is needed to more objective methods of enquiry, using non-linguists as informants. The latter procedure is unavoidable when working on foreign languages, or child speech." > ~/JavaProjects/luceneDemo/src/main/resources/data/2.txt

echo "Many factors must be considered when selecting informants – whether one is working with single speakers (a common situation when languages have not been described before), two people interacting, small groups or large-scale samples. Age, sex, social background and other aspects of identity are important, as these factors are known to influence the kind of language used. The topic of conversation and the characteristics of the social setting (e.g. the level of formality) are also highly relevant, as are the personal qualities of the informants (e.g. their fluency and consistency). For larger studies, scrupulous attention has been paid to the sampling theory employed, and in all cases, decisions have to be made about the best investigative techniques to use." > ~/JavaProjects/luceneDemo/src/main/resources/data/3.txt
```

#### 构建索引

``` java
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;

import java.io.File;
import java.io.FileReader;
import java.io.IOException;
import java.nio.file.Paths;

public class Indexer {
    private final IndexWriter writer;

    public Indexer(String indexDir) throws IOException {
        Directory dir = FSDirectory.open(Paths.get(indexDir));
        // StandardAnalyzer标准分词器，将忽略is，a，the等单词
        IndexWriterConfig config = new IndexWriterConfig(new StandardAnalyzer());
        // 实例化写索引对象
        writer = new IndexWriter(dir, config);
    }

    // 关闭索引写入
    public void close() throws IOException {
        writer.close();
    }

    // 索引dataDir目录下的所有文件
    public int indexALl(String dataDir) throws Exception {
        File[] files = new File(dataDir).listFiles();
        assert files != null;
        for (File file : files) {
            indexFile(file);
        }
        return writer.numDocs();
    }

    // 索引单个文件
    private void indexFile(File file) throws Exception {
        System.out.println("Indexing " + file.getCanonicalPath());
        Document doc = getDocument(file);
        // 将文档写入索引
        writer.addDocument(doc);
        writer.commit();
    }

    // 获取某个文件的文档
    private Document getDocument(File file) throws Exception {
        Document doc = new Document();
        doc.add(new TextField("contents", new FileReader(file)));
        doc.add(new TextField("fileName", file.getName(), TextField.Store.YES));
        doc.add(new TextField("fullPath", file.getCanonicalPath(), TextField.Store.YES));
        return doc;
    }

    public static void main(String[] args) {
        String indexDir = "~/JavaProjects/luceneDemo/src/main/resources/Index";
        String dataDir = "~/JavaProjects/luceneDemo/src/main/resources/data";
        Indexer indexer = null;
        int indexNum = 0;
        long startTime = System.currentTimeMillis();
        try {
            indexer = new Indexer(indexDir);
            indexNum = indexer.indexALl(dataDir);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                assert indexer != null;
                indexer.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        long endTime = System.currentTimeMillis();
        System.out.println("Index " + indexNum + " files, cost " + (endTime - startTime) + " ms");
    }
}
```

#### 检索

``` java
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexReader;
import org.apache.lucene.queryparser.classic.QueryParser;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;

import java.nio.file.Paths;

public class Searcher {
    public static void search(String indexDir, String q) throws Exception {
        // 索引所在路径
        Directory dir = FSDirectory.open(Paths.get(indexDir));
        IndexReader reader = DirectoryReader.open(dir);
        IndexSearcher searcher = new IndexSearcher(reader);
        // 创建查询解析器
        QueryParser parser = new QueryParser("contents", new StandardAnalyzer());
        Query query = parser.parse(q);

        long start = System.currentTimeMillis();
        // 查询并记录前十条数据
        TopDocs hits = searcher.search(query, 10);
        long end = System.currentTimeMillis();
        System.out.println("Found " + hits.totalHits + " document(s) (in " + (end - start) + " milliseconds) that matched query '" + q + "':");

        // 从结果中取数据
        for (ScoreDoc scoreDoc : hits.scoreDocs) {
            // scoreDoc.doc即docID，根据这个docID来获取文档
            Document doc = searcher.doc(scoreDoc.doc);
            System.out.println(doc.get("fullPath"));
        }
        reader.close();
    }

    public static void main(String[] args) {
        String indexDir = "~/JavaProjects/luceneDemo/src/main/resources/Index";
        String q = "speakers";
        try {
            search(indexDir, q);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

#### 增删改查测试

``` java
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.StringField;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.*;
import org.apache.lucene.queryparser.classic.QueryParser;
import org.apache.lucene.search.*;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
import org.junit.jupiter.api.Test;

import java.nio.file.Paths;

public class IndexingTest {
    private Directory dir;

    private final String[] ids = {"1", "2", "3"};
    private final String[] tools = {"Lucene", "ElasticSearch", "Solr"};
    private final String[] positions = {"Java Engineer", "DevOps Engineer", "Data Engineer"};
    private final String[] descs = {
            "Lucene is a full-featured text search engine library written entirely in Java.",
            "ElasticSearch is a distributed, RESTful search and analytics engine capable of solving a growing number of use cases.",
            "Solr is the popular, blazing-fast, open source enterprise search platform built on Apache Lucene."
    };

    @Test
    public void testIndexAdd() throws Exception {
        IndexWriter writer = getWriter();
        for (int i = 0; i < ids.length; i++) {
            Document doc = new Document();
            doc.add(new StringField("id", ids[i], StringField.Store.YES));
            doc.add(new StringField("tool", tools[i], StringField.Store.YES));
            doc.add(new TextField("position", positions[i], TextField.Store.YES));
            // TextField field = new TextField("position", poitions[i], StringField.Store.YES);
            // if("DevOps Engineer".equals(positions[i])){
            //     field.setBoost(1.5f);
            // }
            // doc.add(field);
            doc.add(new TextField("desc", descs[i], StringField.Store.YES));
            writer.addDocument(doc);
        }
        writer.commit();
        writer.close();
    }

    private IndexWriter getWriter() throws Exception {
        String indexDir = "~/JavaProjects/luceneDemo/src/main/resources/testIndex";
        dir = FSDirectory.open(Paths.get(indexDir));
        Analyzer analyzer = new StandardAnalyzer();
        IndexWriterConfig config = new IndexWriterConfig(analyzer);
        IndexWriter writer = new IndexWriter(dir, config);
        return writer;
    }

    @Test
    public void testIndexWriterNum() throws Exception {
        IndexWriter writer = getWriter();
        System.out.println("Total number of documents in this testIndexAdd: " + writer.numDocs());
    }

    @Test
    public void testDeleteBeforeMerge() throws Exception {
        IndexWriter writer = getWriter();
        System.out.println("Total number of documents in this testIndexAdd before delete: " + writer.numDocs());
        writer.deleteDocuments(new Term("id", "1"));
        writer.commit();
        System.out.println("Max number of documents in this testIndexAdd after delete: " + writer.maxDoc());
        System.out.println("Total number of documents in this testIndexAdd after delete: " + writer.numDocs());
        writer.close();
    }

    @Test
    public void testDeleteAfterMerge() throws Exception {
        IndexWriter writer = getWriter();
        System.out.println("Total number of documents in this testIndexAdd before delete: " + writer.numDocs());
        writer.deleteDocuments(new Term("id", "1"));
        writer.forceMergeDeletes();
        writer.commit();
        System.out.println("Max number of documents in this testIndexAdd after delete: " + writer.maxDoc());
        System.out.println("Total number of documents in this testIndexAdd after delete: " + writer.numDocs());
        writer.close();
    }

    @Test
    public void testUpdate() throws Exception {
        IndexWriter writer = getWriter();

        // new a document
        Document doc = new Document();
        doc.add(new StringField("id", "1", StringField.Store.YES));
        doc.add(new StringField("tool", "Lucene", StringField.Store.YES));
        doc.add(new TextField("position", "Java Engineer", StringField.Store.YES));
        doc.add(new TextField("desc", "Lucene is a full-text search engine library", StringField.Store.YES));
        writer.updateDocument(new Term("id", "1"), doc);

        writer.commit();
        writer.close();
        System.out.println(doc.getField("desc"));
    }

    @Test
    public void testIndexReader() throws Exception {
        String indexDir = "~/JavaProjects/luceneDemo/src/main/resources/testIndex";
        dir = FSDirectory.open(Paths.get(indexDir));
        IndexReader reader = DirectoryReader.open(dir);
        System.out.println("Total number of documents in this testIndexAdd: " + reader.numDocs());
        System.out.println("Total number of deleted documents in this testIndexAdd: " + reader.numDeletedDocs());
        System.out.println("Total number of max docs in this testIndexAdd: " + reader.maxDoc());
        reader.close();
    }

    @Test
    public void testIndexSearch() throws Exception {
        String indexDir = "~/JavaProjects/luceneDemo/src/main/resources/testIndex";
        dir = FSDirectory.open(Paths.get(indexDir));
        IndexReader reader = DirectoryReader.open(dir);
        IndexSearcher searcher = new IndexSearcher(reader);

        // Query
        QueryParser parse = new QueryParser("desc", new StandardAnalyzer());
        Query query = parse.parse("search");

        // 权重搜索排序
        // 创建TermQuery
        TermQuery termQuery = new TermQuery(new Term("position", "DevOps".toLowerCase()));
        // 提升TermQuery的权重
        BoostQuery boostQuery = new BoostQuery(termQuery, 1.5f);
        // 创建BooleanQuery
        BooleanQuery.Builder booleanQuery = new BooleanQuery.Builder();
        booleanQuery.add(query, BooleanClause.Occur.MUST);
        booleanQuery.add(boostQuery, BooleanClause.Occur.SHOULD);

        TopDocs hits = searcher.search(booleanQuery.build(), 10);


        System.out.println("Total number of hits: " + hits.totalHits);

        for (ScoreDoc scoreDoc : hits.scoreDocs) {
            Document doc = searcher.doc(scoreDoc.doc);
            System.out.println(doc.get("id"));
            System.out.println(doc.get("tool"));
            System.out.println(doc.get("position"));
            System.out.println(doc.get("desc"));
        }

        reader.close();
    }
}
```

#### :mag:Attention

- `TextField`和`StringField`的区别
  - `StringField`：不会被分词器处理，适用于精确匹配查询，如ID、日期、分类等字段，`StringField.Store.YES`表示存储字段值。
  - `TextField`：会被分词器处理，适用于需要进行全文搜索的文本内容，如文章内容、描述等字段，`TextField.Store.YES`表示存储字段值并进行分词处理。

    ``` java
    // 使用StringField存储ID字段
    doc.add(new StringField("id", "1", StringField.Store.YES));

    // 使用TextField存储描述字段
    doc.add(new TextField("desc", "Lucene is a full-featured text search engine library written entirely in Java.", TextField.Store.YES));
    ```

- `IndexWriter`的`commit`和`close`方法
  - `commit`：提交索引，将内存中的索引写入磁盘，保证索引的一致性。
  - `close`：关闭索引写入器，释放资源。

- `TermQuery`中的文本可以是大写也可以是小写，但必须与索引中的文本完全匹配。因为`TermQuery`不会对查询文本进行分析或分词处理，所以如果索引中的文本是小写，那么查询文本也必须是小写。如果使用了`StandardAnalyzer`或其他分词器在索引时将文本转换为小写，那么在查询时也需要将文本转换为小写以确保匹配。可以使用`LowerCaseFilter`或`LowerCaseTokenizer`来确保查询文本是小写的。

    ``` java
    // 创建TermQuery并将文本转换为小写
    TermQuery termQuery = new TermQuery(new Term("position", "DevOps".toLowerCase()));
    ```

- `TermQuery`不会进行分词处理，只会进行精确匹配查询，所以查询文本必须与索引中的文本完全匹配。如果需要查询包含多个词的字段，可以使用`PhraseQuery`或`BooleanQuery`。

    ``` java
    // 使用PhraseQuery查询包含多个词的字段
    PhraseQuery phraseQuery = new PhraseQuery.Builder()
        .add(new Term("position", "DevOps"))
        .add(new Term("position", "Engineer"))
        .build();

    // 使用BooleanQuery查询包含多个词的字段
    BooleanQuery booleanQuery = new BooleanQuery.Builder()
        .add(new TermQuery(new Term("position", "DevOps")), BooleanClause.Occur.MUST)
        .add(new TermQuery(new Term("position", "Engineer")), BooleanClause.Occur.MUST)
        .build();
    ```

- $lunece.version>7.0$已经不再支持`setBoost`方法
  - [LUCENE-6819](https://issues.apache.org/jira/browse/LUCENE-6819): Index-time boosts are not supported anymore. As a replacement, index-time scoring factors should be indexed into a doc value field and combined at query time using eg. FunctionScoreQuery
- 同样不建议使用`Query.setBoost()`或者`Query.createWeight()`
  - [LUCENE-6590](https://issues.apache.org/jira/browse/LUCENE-6590): Query.setBoost(), Query.getBoost() and Query.clone() are gone. In order to apply boosts, you now need to wrap queries in a BoostQuery.
- 建议用`BoostQuery`封装来写权重查询，可以通过设置权重值来调整查询的相关性，值越大表示相关性越高。可以将`TermQuery`或`BooleanQuery`包装在`BoostQuery`中，然后将`BoostQuery`添加到`BooleanQuery`中。

    ``` java
    // 创建TermQuery
    TermQuery termQuery = new TermQuery(new Term("position", "DevOps".toLowerCase()));
    // 提升TermQuery的权重
    BoostQuery boostQuery = new BoostQuery(termQuery, 1.5f);
    // 创建BooleanQuery
    BooleanQuery.Builder booleanQuery = new BooleanQuery.Builder();
    booleanQuery.add(query, BooleanClause.Occur.MUST);
    booleanQuery.add(boostQuery, BooleanClause.Occur.SHOULD);
    ```
