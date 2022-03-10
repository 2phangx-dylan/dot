# 前言

> - 本篇介绍如何在Java中以传统的方式使用ElasticSearch，以及SpringData中如何使用更简单的方式使用ElasticSearch。
> - 总结：操作索引库在完全是可以类比操作数据库的，只是在ElasticSearch 7.x中Type的概念已经被标注为已过时，并将在ElasticSearch 8.x版本中完全被移除。

# ElasticSearch编程操作

- 本小节将阐述如何在Java中使用ElasticSearch 7.9.1，大多数的知识点来源于官方提供的API，可以参考以下网址：`https://www.elastic.co/guide/en/elasticsearch/client/java-api/7.9/index.html`
- 提示：大多数即使是官方所提供的方法，都被标注为`Deprecated`(已过时)，推荐不再使用这些方法。

## 1. 创建工程并导入坐标

- 使用Maven工程，需要在`pom.xml`中导入相应的依赖坐标：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.dylanphang</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>1.0-SNAPSHOT</version>

    <name>elasticsearch</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>9</maven.compiler.source>
        <maven.compiler.target>9</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- ElasticSearch依赖 -->
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>7.9.1</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>transport</artifactId>
            <version>7.9.1</version>
        </dependency>

        <!-- 从ElasticSearch官方API获取的相关日志配置 -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-to-slf4j</artifactId>
            <version>2.12.1</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.30</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.30</version>
        </dependency>

        <!-- 测试单元 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
</project>
```

## 2. 配置log4j2文件

- 从官方获取了依赖坐标配置，需要添加`log4j2.properties`的配置文件到`resources`目录中：

```properties
appender.console.type = Console
appender.console.name = console
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = [%d{ISO8601}][%-5p][%-25c] %marker%m%n

rootLogger.level = info
rootLogger.appenderRef.console.ref = console
```

## 3. 编写ElasticSearch测试类

- ElasticSearch的操作主要包括两项：
  1. 索引库操作：
     1. 创建索引库不带Mappings、创建索引库带Mappings、添加Mappings；
     2. 删除索引库、删除索引库中的Mappings。
  2. 文档的操作：
     1. 创建文档；
     2. 修改文档；
     3. 删除文档；
     4. 查询文档。

### a. 索引库的增删操作

- 所有的方法都是比较定式的，详情参考以下的Java代码：

```java
package cn.dylanphang.traditional;

import org.elasticsearch.client.transport.TransportClient;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.transport.TransportAddress;
import org.elasticsearch.common.xcontent.XContentBuilder;
import org.elasticsearch.common.xcontent.XContentFactory;
import org.elasticsearch.transport.client.PreBuiltTransportClient;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.net.InetAddress;
import java.net.UnknownHostException;

/**
 * 本篇介绍传统的在ElasticSearch中添加索引库、在索引库中添加Mappings以及在Mappings中添加文档的方法。
 * 严格来说，很多方法都已经过时了，学旧的是为了看旧代码，还是需要贴近时代。
 * 更多方法参考：https://www.elastic.co/guide/en/elasticsearch/client/java-api/7.9/index.html
 *
 * @author dylan
 */
public class ElasticSearchClientTest {

    private TransportClient client = null;

    /**
     * 创建索引库，设置了分片数量、复制数量，但没有添加Mappings
     *
     * @throws IOException IO流异常
     */
    @Test
    public void testCreateIndex() throws IOException {
        // 1.创建索引库设置
        XContentBuilder builder = XContentFactory.jsonBuilder()
                .startObject()
                .startObject("settings")
                .field("number_of_shards", 3)
                .field("number_of_replicas", 1)
                .endObject()
                .endObject();
        // 2.添加一个索引库
        this.client.admin().indices()
                .prepareCreate("index-hello")
                .setSource(builder)
                .get();
    }

    /**
     * 此方法调用删除索引库的方法。
     */
    @Test
    public void testDeleteIndex() {
        this.deleteIndices("test-indices");
    }

    /**
     * 创建一个索引库，在此之前设置分片数量、复制数量，同时在创建之后，添加Mappings。
     *
     * @throws IOException IO流异常
     */
    @Test
    public void testAddMappings() throws IOException {
        // 1.创建索引库设置
        XContentBuilder settingsBuilder = XContentFactory.jsonBuilder()
                .startObject()
                .startObject("settings")
                .field("number_of_shards", 5)
                .field("number_of_replicas", 1)
                .endObject()
                .endObject();

        // 2.添加一个测试索引库
        this.client.admin().indices()
                .prepareCreate("test-indices")
                .setSource(settingsBuilder)
                .get();

        // 3.创建一个Mappings
        XContentBuilder mappingsBuilder = XContentFactory.jsonBuilder()
                .startObject()
                .startObject("properties")
                .startObject("id")
                .field("type", "text")
                .field("store", true)
                .endObject()
                .startObject("title")
                .field("type", "text")
                .field("store", true)
                .field("index", true)
                .field("analyzer", "standard")
                .endObject()
                .startObject("content")
                .field("type", "text")
                .field("store", true)
                .field("index", true)
                .field("analyzer", "standard")
                .endObject()
                .endObject()
                .endObject();

        // 4.向索引库添加Mappings
        this.client.admin().indices()
                .preparePutMapping("test-indices")
                .setType("_doc")
                .setSource(mappingsBuilder)
                .get();
    }

    /**
     * 向指定索引库中添加文档，文档必须要已经具有Mappings。
     *
     * @throws IOException IO流异常
     */
    @Test
    public void testAddDocument() throws IOException {
        // 1.创建一个索引库并具有相应的Mappings
        this.testAddMappings();

        // 2.创建文档
        XContentBuilder documentBuilder = XContentFactory.jsonBuilder()
                .startObject()
                .field("id", 1L)
                .field("title", "1我是标题啊")
                .field("content", "1我是内容啊")
                .endObject();

        // 3.向索引库添加文档（document）
        this.client
                .prepareIndex()
                .setIndex("test-indices")
                .setType("_doc")
                .setId("1")
                .setSource(documentBuilder)
                .get();
    }

    /**
     * 删除指定名称的索引库。
     *
     * @param indices 索引库名称
     */
    private void deleteIndices(String indices) {
        // *.删除一个索引库
        this.client.admin().indices()
                .prepareDelete(indices)
                .get();
    }

    /**
     * 初始化集群/节点的Client对象，如果默认集群名称不是“elasticsearch”，需要使用Settings指定集群名字。
     *
     * @throws UnknownHostException 未知主机错误
     */
    @Before
    public void init() throws UnknownHostException {
        // 1.创建设置，如果集群名称不为“elasticsearch”的话，需要指定Settings对象中的cluster.name
        Settings settings = Settings.builder()
                .put("cluster.name", "my-esCluster").build();

        // 2.创建客户端对象TransportClient
        this.client = new PreBuiltTransportClient(settings)
                .addTransportAddress(new TransportAddress(InetAddress.getByName("127.0.0.1"), 9301))
                .addTransportAddress(new TransportAddress(InetAddress.getByName("127.0.0.1"), 9302))
                .addTransportAddress(new TransportAddress(InetAddress.getByName("127.0.0.1"), 9303));
    }

    /**
     * 测试结束后，关闭Client对象。
     */
    @After
    public void close() {
        // *.释放资源
        if (this.client != null) {
            this.client.close();
        }
    }
}
```

- 使用基本的ElasticSearch所提供的TransportClient，过程是比较繁琐的，并且在ElasticSearch 7.x中，TransportClient类已经被标注为`Deprecate`，官方已经不推荐使用了。

### b. 文档的增删改查操作

- ElasticSearch API中提供的也是一些增删改查操作的基本方式，参考以下Java代码：

```java
package cn.dylanphang.traditional;

import org.elasticsearch.action.search.SearchResponse;
import org.elasticsearch.client.transport.TransportClient;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.text.Text;
import org.elasticsearch.common.transport.TransportAddress;
import org.elasticsearch.common.xcontent.XContentBuilder;
import org.elasticsearch.common.xcontent.XContentFactory;
import org.elasticsearch.index.query.IdsQueryBuilder;
import org.elasticsearch.index.query.QueryBuilders;
import org.elasticsearch.index.query.QueryStringQueryBuilder;
import org.elasticsearch.index.query.TermQueryBuilder;
import org.elasticsearch.search.SearchHit;
import org.elasticsearch.search.SearchHits;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightBuilder;
import org.elasticsearch.search.fetch.subphase.highlight.HighlightField;
import org.elasticsearch.transport.client.PreBuiltTransportClient;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.net.InetAddress;
import java.net.UnknownHostException;
import java.util.Map;
import java.util.Random;

/**
 * 本篇将演示如何对文档进行查询。
 * 更多方法参考：https://www.elastic.co/guide/en/elasticsearch/client/java-api/7.9/index.html
 *
 * @author dylan
 */
public class ElasticSearchDocumentTest {

    private TransportClient client = null;

    /**
     * 可以将输出的结果中的检索条件进行高亮设置，即添加指定的html标签，达到高亮的效果
     */
    @Test
    public void highlightKeyword() {
        // 1.设置高亮格式，即需要添加的html标签是什么，对哪个域（field）的检索条件生效
        final HighlightBuilder highlightBuilder = new HighlightBuilder();
        highlightBuilder.preTags("<font style='color: red'>");
        highlightBuilder.postTags("</font>");
        highlightBuilder.field("content");

        // 2.创建检索条件，检索域为content，检索条件为“juice string”，会自动进行分词
        final QueryStringQueryBuilder queryStringQueryBuilder
                = QueryBuilders.queryStringQuery("juice string").defaultField("content");

        // 3.设置目标检索索引库，设置检索条件，设置检索输出页、每页显示条数，及设置检索高亮
        final SearchResponse searchResponse = this.client
                .prepareSearch("test-indices")
                .setQuery(queryStringQueryBuilder)
                .setFrom(0)
                .setSize(12)
                .highlighter(highlightBuilder)
                .get();

        // 4.获取SearchHits对象，可以从对象中获取到具体的命中结果数
        final SearchHits hits = searchResponse.getHits();

        System.out.println("Total hits: " + hits.getTotalHits());
        System.out.println("=====================================");

        // 5.遍历SearchHits，SearchHits继承了Iterable<SearchHit>
        for (SearchHit searchHit : hits) {
            // 6.从SearchHit对象中获取到检索结果的内容
            System.out.println("Document Property:");
            final Map<String, Object> sourceMap = searchHit.getSourceAsMap();

            System.out.println("id: " + sourceMap.get("id"));
            System.out.println("title: " + sourceMap.get("title"));
            System.out.println("content: " + sourceMap.get("content"));
            System.out.println("=====================================");

            // 7.获取高亮部分，检索结果中可能包含一个部分以上的、包含检索条件的部分，因此getFragments()获取到的是Text[]
            System.out.println("Highlight Part");
            final Map<String, HighlightField> highlightFields = searchHit.getHighlightFields();
            final HighlightField highlightField = highlightFields.get("content");
            final Text[] fragments = highlightField.getFragments();
            if (fragments != null) {
                System.out.println(fragments[0].toString());
            }
            System.out.println("=====================================");
        }
    }

    /**
     * 通过StringQuery查询，会对检索条件进行分词。
     */
    @Test
    public void queryByStringQuery() {
        // 1.创建检索条件，QueryStringQueryBuilder对象
        final QueryStringQueryBuilder queryStringQueryBuilder
                = QueryBuilders.queryStringQuery("juice string").defaultField("content");

        // 2.给Client对象传入检索目标索引库、检索条件对象以及分页信息等，获取SearchResponse对象
        final SearchResponse searchResponse = this.client
                .prepareSearch("test-indices")
                .setQuery(queryStringQueryBuilder)
                .setFrom(0)
                .setSize(8)
                .get();

        // 3.SearchHits对象就类似于数据库JDBC中的ResultSet一样
        final SearchHits hits = searchResponse.getHits();

        System.out.println("Total hits: " + hits.getTotalHits());
        System.out.println("=====================================");

        // 4.遍历SearchHits并打印结果
        for (SearchHit searchHit : hits) {
            System.out.println("Document Property:");
            final Map<String, Object> sourceMap = searchHit.getSourceAsMap();

            System.out.println("id: " + sourceMap.get("id"));
            System.out.println("title: " + sourceMap.get("title"));
            System.out.println("content: " + sourceMap.get("content"));
            System.out.println("=====================================");
        }
    }

    /**
     * 通过Term查询，不会对检索条件进行分词。
     */
    @Test
    public void queryByTerm() {
        // 1.创建检索条件，TermQueryBuilder对象
        final TermQueryBuilder termQueryBuilder
                = QueryBuilders.termQuery("content", "juice");

        // 2.给Client对象传入检索目标索引库、检索条件对象等，获取SearchResponse对象
        final SearchResponse searchResponse = this.client
                .prepareSearch("test-indices")
                .setQuery(termQueryBuilder)
                .get();

        // 3.SearchHits对象就类似于数据库JDBC中的ResultSet一样
        final SearchHits hits = searchResponse.getHits();

        System.out.println("Total hits: " + hits.getTotalHits());
        System.out.println("=====================================");

        // 4.遍历SearchHits并打印结果
        for (SearchHit searchHit : hits) {
            System.out.println("Document Property:");
            final Map<String, Object> sourceMap = searchHit.getSourceAsMap();

            System.out.println("id: " + sourceMap.get("id"));
            System.out.println("title: " + sourceMap.get("title"));
            System.out.println("content: " + sourceMap.get("content"));
            System.out.println("=====================================");
        }
    }

    /**
     * 通过id查询，这个id指代的是_id。
     */
    @Test
    public void queryById() {
        // 1.创建检索条件，IdsQueryBuilder对象
        final IdsQueryBuilder idsQueryBuilder = QueryBuilders.idsQuery().addIds("91", "28");

        // 2.给Client对象传入检索目标索引库、检索条件对象等，获取SearchResponse对象
        final SearchResponse searchResponse = this.client
                .prepareSearch("test-indices")
                .setQuery(idsQueryBuilder)
                .get();

        // 3.SearchHits对象就类似于数据库JDBC中的ResultSet一样
        final SearchHits hits = searchResponse.getHits();

        System.out.println("Total hits: " + hits.getTotalHits());
        System.out.println("=====================================");

        // 4.遍历SearchHits并打印结果
        for (SearchHit searchHit : hits) {
            System.out.println("Document Property:");
            final Map<String, Object> sourceMap = searchHit.getSourceAsMap();

            System.out.println("id: " + sourceMap.get("id"));
            System.out.println("title: " + sourceMap.get("title"));
            System.out.println("content: " + sourceMap.get("content"));
            System.out.println("=====================================");
        }
    }

    /**
     * 用于循环创建实验的搜索内容。
     *
     * @throws IOException IO流异常
     */
    @Test
    public void addDataCyclically() throws IOException {
        String[] searchContent = {"string", "lamp", "juice"};

        final Random random = new Random();
        XContentBuilder documentBuilder = null;
        final StringBuilder stringBuilder = new StringBuilder();

        for (int i = 1; i <= 100; i++) {
            // 1.获取必要的填充资料
            final String title = stringBuilder
                    .append(searchContent[random.nextInt(3)])
                    .append("标题")
                    .append(i)
                    .toString();
            stringBuilder.delete(0, title.length());

            final String content = stringBuilder
                    .append(searchContent[random.nextInt(3)])
                    .append("内容")
                    .append(i)
                    .toString();
            stringBuilder.delete(0, content.length());

            final String id = stringBuilder.append(i).toString();
            stringBuilder.delete(0, id.length());

            // 2.创建文档
            documentBuilder = XContentFactory.jsonBuilder()
                    .startObject()
                    .field("id", id)
                    .field("title", title)
                    .field("content", content)
                    .endObject();

            // 3.向索引库添加文档（document）
            this.client
                    .prepareIndex()
                    .setIndex("test-indices")
                    .setType("_doc")
                    .setId(id)
                    .setSource(documentBuilder)
                    .get();
        }
    }

    /**
     * 删除文档。
     */
    @Test
    public void testDeleteDocument() {
        // *.准备实验环境之前，需要删除其中的文档，索引库中自由一个_id为1的文档，将其删除
        this.client
                .prepareDelete("test-indices", "_doc", "1")
                .get();

    }

    /**
     * 创建索引库和Mappings。
     *
     * @throws IOException IO流异常
     */
    @Test
    public void testAddMappings() throws IOException {
        // 1.创建索引库设置
        XContentBuilder settingsBuilder = XContentFactory.jsonBuilder()
                .startObject()
                .startObject("settings")
                .field("number_of_shards", 5)
                .field("number_of_replicas", 1)
                .endObject()
                .endObject();

        // 2.添加一个测试索引库
        this.client.admin().indices()
                .prepareCreate("test-indices")
                .setSource(settingsBuilder)
                .get();

        // 3.创建一个Mappings
        XContentBuilder mappingsBuilder = XContentFactory.jsonBuilder()
                .startObject()
                .startObject("properties")
                .startObject("id")
                .field("type", "text")
                .field("store", true)
                .endObject()
                .startObject("title")
                .field("type", "text")
                .field("store", true)
                .field("index", true)
                .field("analyzer", "standard")
                .endObject()
                .startObject("content")
                .field("type", "text")
                .field("store", true)
                .field("index", true)
                .field("analyzer", "standard")
                .endObject()
                .endObject()
                .endObject();

        // 4.向索引库添加Mappings
        this.client.admin().indices()
                .preparePutMapping("test-indices")
                .setType("_doc")
                .setSource(mappingsBuilder)
                .get();
    }

    /**
     * 添加文档。
     *
     * @throws IOException IO流异常
     */
    @Test
    public void testAddDocument() throws IOException {
        // 1.创建一个索引库并具有相应的Mappings
        this.testAddMappings();

        // 2.创建文档
        XContentBuilder documentBuilder = XContentFactory.jsonBuilder()
                .startObject()
                .field("id", 1L)
                .field("title", "1我是标题啊")
                .field("content", "1我是内容啊")
                .endObject();

        // 3.向索引库添加文档（document）
        this.client
                .prepareIndex()
                .setIndex("test-indices")
                .setType("_doc")
                .setId("1")
                .setSource(documentBuilder)
                .get();
    }

    /**
     * 初始化集群/节点的Client对象，如果默认集群名称不是“elasticsearch”，需要使用Settings指定集群名字。
     *
     * @throws UnknownHostException 未知主机错误
     */
    @Before
    public void init() throws UnknownHostException {
        // 1.创建设置，如果集群名称不为“elasticsearch”的话，需要指定Settings对象中的cluster.name
        Settings settings = Settings.builder()
                .put("cluster.name", "my-esCluster").build();

        // 2.创建客户端对象TransportClient
        this.client = new PreBuiltTransportClient(settings)
                .addTransportAddress(new TransportAddress(InetAddress.getByName("127.0.0.1"), 9301))
                .addTransportAddress(new TransportAddress(InetAddress.getByName("127.0.0.1"), 9302))
                .addTransportAddress(new TransportAddress(InetAddress.getByName("127.0.0.1"), 9303));
    }

    /**
     * 测试结束后，关闭Client对象。
     */
    @After
    public void close() {
        // *.释放资源
        if (this.client != null) {
            this.client.close();
        }
    }
}
```

# SpirngData ES编程操作

## 1. 创建工程并导入坐标

- 使用Maven工程，需要在`pom.xml`中导入相应的依赖坐标：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>cn.dylanphang</groupId>
    <artifactId>springdata-es</artifactId>
    <version>1.0-SNAPSHOT</version>

    <name>springdata-es</name>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>9</maven.compiler.source>
        <maven.compiler.target>9</maven.compiler.target>
    </properties>

    <dependencies>
        <!-- ElasticSearch依赖 -->
        <dependency>
            <groupId>org.elasticsearch</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>7.9.1</version>
        </dependency>
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>transport</artifactId>
            <version>7.9.1</version>
        </dependency>

        <!-- SpringDate-ElasticSearch-->
        <dependency>
            <groupId>org.springframework.data</groupId>
            <artifactId>spring-data-elasticsearch</artifactId>
            <version>4.0.3.RELEASE</version>
            <exclusions>
                <exclusion>
                    <groupId>org.elasticsearch.plugin</groupId>
                    <artifactId>transport-netty4-client</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- SpringTest -->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-test</artifactId>
            <version>5.2.7.RELEASE</version>
        </dependency>

        <!-- 从ElasticSearch官方API获取的相关日志配置 -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-to-slf4j</artifactId>
            <version>2.12.1</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.30</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.7.30</version>
        </dependency>

        <!-- 测试单元 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>compile</scope>
        </dependency>
    </dependencies>
</project>
```

## 2. 创建Spring配置类

- 经过试验，Spring相关的xml约束已经不再支持相关的ElasticSearch包扫描标签，这些标签包括：
  - `<elasticsearch:transport-client>`：用于配置并获取TranspotClient对象；
  - `<elasticsearch:repositories>`：用于扫描dao包的标签

- 需要将该些设置放置于Spring的配置类中，Java代码如下：

```java
package cn.dylanphang.transportclient.config;

import org.elasticsearch.client.Client;
import org.elasticsearch.client.transport.TransportClient;
import org.elasticsearch.common.settings.Settings;
import org.elasticsearch.common.transport.TransportAddress;
import org.elasticsearch.transport.client.PreBuiltTransportClient;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.elasticsearch.config.ElasticsearchConfigurationSupport;
import org.springframework.data.elasticsearch.core.ElasticsearchTemplate;
import org.springframework.data.elasticsearch.repository.config.EnableElasticsearchRepositories;

import java.net.InetAddress;
import java.net.UnknownHostException;

/**
 * @author dylan
 */
@Configuration // 表明是Spirng的配置类
@EnableElasticsearchRepositories("cn.dylanphang.transportclient.dao") // 此处配置相当于<elasticsearch:repositories>标签
public class TransportClientConfig extends ElasticsearchConfigurationSupport {

    @Bean
    public Client elasticsearchClient() throws UnknownHostException {
        Settings settings = Settings.builder().put("cluster.name", "my-esCluster").build();
        TransportClient client = new PreBuiltTransportClient(settings);
        client.addTransportAddress(new TransportAddress(InetAddress.getByName("127.0.0.1"), 9301));
        client.addTransportAddress(new TransportAddress(InetAddress.getByName("127.0.0.1"), 9302));
        client.addTransportAddress(new TransportAddress(InetAddress.getByName("127.0.0.1"), 9303));
        return client;
    }

    @Bean(name = {"elasticsearchOperations", "elasticsearchTemplate"})
    public ElasticsearchTemplate elasticsearchTemplate() throws UnknownHostException {
        return new ElasticsearchTemplate(elasticsearchClient());
    }
}
```

## 3. 创建Document类

- SpringDataES将文档实体化为一个普通的pojo类，通过配置pojo类的注解，可以达到配置索引库名称、域类型及名称等操作，Repository类如下：

- Article.java：

```java
package cn.dylanphang.transportclient.pojo;

import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

/**
 * @author dylan
 */
@Document(indexName = "spring-indices", shards = 3) // 表明需要创建或操作的索引库名称为“spring-indices”
public class Article {
    /**
     * 被@Id注解修饰的字段，其类型在ElasticSearch 7.x中会被标注为keyword，不管你设置什么type
     */
    @Id // 这个id是必要，是向ElasticsearchRepository<T, ID>传递的前提
    @Field(type = FieldType.Long, store = true) // 设置域Field的类型及是否存储
    private Long id;
    @Field(type = FieldType.Text, store = true, analyzer = "standard") // 可以设置该域Field默认分析器
    private String title;
    @Field(type = FieldType.Text, store = true, analyzer = "standard")
    private String content;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    @Override
    public String toString() {
        return "Article{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", content='" + content + '\'' +
                '}';
    }
}
```

- Paper.java：
```java
package cn.dylanphang.transportclient.pojo;

import org.springframework.data.annotation.Id;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Field;
import org.springframework.data.elasticsearch.annotations.FieldType;

/**
 * @author dylan
 */
@Document(indexName = "test-indices", createIndex = false)
public class Paper {
    @Id
    @Field(type = FieldType.Text, store = true)
    private String id;
    @Field(type = FieldType.Text, store = true, analyzer = "standard")
    private String title;
    @Field(type = FieldType.Text, store = true, analyzer = "standard")
    private String content;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    @Override
    public String toString() {
        return "Article{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", content='" + content + '\'' +
                '}';
    }
}
```

## 4. 创建Repository类

- Repository相当于Dao，用于对索引库进行操作的类，需要继承ElasticsearchRepository<T, ID>，源码如下：

- ArticleRepository.java：

```java
package cn.dylanphang.transportclient.dao;

import cn.dylanphang.transportclient.pojo.Article;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

import java.util.List;

/**
 * 务必要注意，ElasticsearchRepository<T, ID>中的ID需要传入被注解@Id修饰的字段的类型
 *
 * @author dylan
 */

public interface ArticleRepository extends ElasticsearchRepository<Article, Long> {
    // 你可以进行方法的自定义操作，但需要遵循SpringData的相关规范，否则方法无效
    List<Article> findAllByContent(String content, Pageable pageable); 
}
```

- PaperRepository.java：

```java
package cn.dylanphang.transportclient.dao;

import cn.dylanphang.transportclient.pojo.Paper;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;

import java.util.List;

/**
 * @author dylan
 */
public interface PaperRepository extends ElasticsearchRepository<Paper, String> {
    List<Paper> findAllByContent(String content, Pageable pageable);
}
```

## 5. 编写ElasticSearch测试类

- 使用SpringDataElasticsearch能大大简化我们的检索操作，示例代码如下：

```java
package cn.dylanphang.transportclient.es;

import cn.dylanphang.transportclient.config.TransportClientConfig;
import cn.dylanphang.transportclient.dao.ArticleRepository;
import cn.dylanphang.transportclient.dao.PaperRepository;
import cn.dylanphang.transportclient.pojo.Article;
import cn.dylanphang.transportclient.pojo.Paper;
import org.elasticsearch.index.query.QueryBuilders;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.core.ElasticsearchTemplate;
import org.springframework.data.elasticsearch.core.mapping.IndexCoordinates;
import org.springframework.data.elasticsearch.core.query.NativeSearchQuery;
import org.springframework.data.elasticsearch.core.query.NativeSearchQueryBuilder;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

import java.util.List;

/**
 * @author dylan
 */
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes = {TransportClientConfig.class})
public class ElasticSearchTest {

    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;
    @Autowired
    private ArticleRepository articleRepository;
    @Autowired
    private PaperRepository paperRepository;

    /**
     * 使用SpringDataElasticsearch来创建索引库，同时添加Mappings。
     */
    @Test
    public void testCreateIndex() {
        // *.直接使用putMapping的方式创建索引库并添加Mappings
        this.elasticsearchTemplate.putMapping(Article.class);

        // *.不能使用createIndex，Article中包含了mappings信息，直接使用以下语句，会出现索引库已存在的错误
        // this.elasticsearchTemplate.createIndex(Article.class);
    }

    /**
     * 删除指定名称的索引库。
     */
    @Test
    public void testDeleteIndex() {
        // *.删除指定的索引库
        this.elasticsearchTemplate.deleteIndex(Article.class);
    }

    /**
     * 向已知索引库中添加文档。
     */
    @Test
    public void testAddDocument() {
        final Article article = new Article();
        article.setId(1L);
        article.setTitle("uSpring1");
        article.setContent("uContent1");

        this.elasticsearchTemplate.save(article);
    }

    /**
     * 从已知索引库中删除文档。
     */
    @Test
    public void testDeleteDocument() {
        // *.也可以使用this.elasticsearchTemplate.delete(Article article)删除索引库中指定id的文档
        this.articleRepository.deleteById(1L);
    }

    /**
     * ElasticsearchRepository中已经内置了一系列的方法，我们可以通过覆写的ArticleRepository直接使用它们。
     */
    @Test
    public void testSearchRepository() {
        // *.注意这里返回的是一个Iterable对象，我们可以调用forEach对象
        final Iterable<Article> articles = this.articleRepository.findAll();
        // *.System.out::println相当于(article) -> System.out.println(article)
        articles.forEach(System.out::println);
    }

    /**
     * 测试自定义的方法，注意此题中“uContent1”使用标准分析器的分词就只有一个token，那就是它本身。
     */
    @Test
    public void testSearchUserDefine() {
        final Pageable pageable = PageRequest.of(0, 12);
        // *.这个地方存入这个Content文本域的时候，standard analyzer分词直接就是"uContent1"，要十分注意
        final List<Article> articles = this.articleRepository.findAllByContent("uContent1", pageable);
        articles.forEach(System.out::println);
    }

    /**
     * 测试数据量更大的test-indices索引库中，自定义检索方法的执行效果。
     */
    @Test
    public void testSearchMore() {
        final Pageable pageable = PageRequest.of(0, 15);
        final List<Paper> juices = this.paperRepository.findAllByContent("juice", pageable);
        juices.forEach(System.out::println);
    }

    /**
     * SpringData也停供原始的搜索方式，使用NativeSearchQuery对象。
     */
    @Test
    public void testNativeSearchQuery() {
        NativeSearchQuery query = new NativeSearchQueryBuilder()
                .withQuery(QueryBuilders.queryStringQuery("juice string").defaultField("content"))
                .withPageable(PageRequest.of(0, 60))
                .build();

        // *.queryForList方法不会检查Document对象中的索引库名称，需要使用IndexCoordinates设置索引库名称
        final IndexCoordinates indexCoordinates = IndexCoordinates.of("test-indices");

        List<Paper> paperList = this.elasticsearchTemplate.queryForList(query, Paper.class, indexCoordinates);

        paperList.forEach(System.out::println);
    }
}
```

