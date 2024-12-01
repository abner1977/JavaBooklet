

# ElasticSearch(搜索服务器)

## 一、介绍

1. 高度可扩展的开源**全文搜索**和分析引擎。
2. 快速、**近实时**的对大数据进行存储、搜索、分析。
3. 用来支撑有复杂的数据搜索需求的企业级应用。

### **特点**

*分布式**、高可用、多数据类型、多API（**Restfu**、java原生）、面向文档、异步写入、近实时、基于lucene、apache协议

### **核心概念：**

近实时、集群、节点、索引、类型、文档、分片、副本

一个 ElasticSearch 集群可以 包含多个 索引 ，相应的每个索引可以包含多 个 类型 。 这些不同的类型存储着多个 文档 ，每个文档又有 多个 属性 。

 • 类似关系： **– 索引-数据库 – 类型-表 – 文档-表中的记录 – 属性-列**

**倒排索引** 提供索引效率



Elasticsearch是一个分布式搜索服务，提供Restful API，<u>底层基于Lucene，采用 多shard（分片）的方式保证数据安全，并且提供自动resharding的功能</u>，

先说Elasticsearch的文件存储，Elasticsearch是面向文档型数据库，一条数据在这里就是一个文档，用JSON作为文档序列化的格式，比如下面这条用户数据：

```
{
    "name" :     "John",
    "sex" :      "Male",
    "age" :      25,
    "birthDate": "1990/05/01",
    "about" :    "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

用Mysql这样的数据库存储就会容易想到建立一张User表，有balabala的字段等，在Elasticsearch里这就是一个*文档*，当然这个文档会属于一个User的*类型*，各种各样的类型存在于一个*索引*当中。这里有一份简易的将Elasticsearch和关系型数据术语对照表:

```
关系数据库     ⇒ 数据库 ⇒ 表    ⇒ 行    ⇒ 列(Columns)  => schema 

Elasticsearch  ⇒ 索引(Index)   ⇒ 类型(type)  ⇒ 文档(Docments)  ⇒ 字段(Fields)   =>Mapping
```

<img src="C:\Users\10674\AppData\Roaming\Typora\typora-user-images\image-20201223165549718.png" alt="image-20201223165549718" style="zoom:50%;" />

**一个 Elasticsearch 集群可以包含多个索引(数据库)，也就是说其中包含了很多类型(表)。这些类型中包含了很多的文档(行)，然后每个文档中又包含了很多的字段(列)**。Elasticsearch的交互，可以使用Java API，也可以直接使用HTTP的Restful API方式，比如我们打算插入一条记录，可以简单发送一个HTTP的请求：

```
PUT /megacorp/employee/1  
{
    "name" :     "John",
    "sex" :      "Male",
    "age" :      25,
    "about" :    "I love to go rock climbing",
    "interests": [ "sports", "music" ]
}
```

## 二、原理

### 倒排索引

简单来说，倒排索引是将查询内容分词之后，与文档的词汇列表进行匹配，从而定位到具体文档。

**倒排索引分为**

**单词词典 Term Dictionary** ： 记录所有文档的单词以及单词到倒排列表的关联关系。 通过B+树 或者哈希拉链法实现。

**倒排列表**：Posting list 记录单词对应的文档，由倒排索引项构成。

倒排索引索引项包括：文档ID / 词频 TF 该单词在文档中出现的次数，用于评分 / 位置 Position 单词在文档中分词的位置 用于语句搜索 / 偏移 Offset 记录单词的开始结束位置，实现高亮显示。

<img src="D:\dailySoftWare\typora\md\img\e4599b618e270df9b64a75eb77bfb326.jpg" alt="e4599b618e270df9b64a75eb77bfb326" style="zoom:50%;" />



**term index 在内存中是以 FST（finite state transducers）的形式保存的**，其特点是非常节省内存。

它包含的是 term 的一些前缀。通过 term index 可以快速地定位到 term dictionary 的某个 offset，然后从这个位置再往后顺序查找。

从 term index 查到对应的 term dictionary 的 block 位置之后，再去磁盘上找 term，然后根据term和posting list对应关系 找到文档ID ，然后根据文档ID找到目标数据。



**Term dictionary 在磁盘上是以分 block 的方式保存的，一个 block 内部利用公共前缀压缩**，比如都是 Ab 开头的单词就可以把 Ab 省去。这样 term dictionary 可以比 b-tree 更节约磁盘空间。



## 三、使用

**注意springboot 和elastic search 版本适配**

https://github.com/spring-projects/spring-data-elasticsearch

```java
/**
 * SpringBoot默认支持两种技术来和ES交互；
 * 1、Jest（默认不生效）
 * 	需要导入jest的工具包（io.searchbox.client.JestClient）
 * 2、SpringData ElasticSearch【ES版本有可能不合适】
 * 		版本适配说明：https://github.com/spring-projects/spring-data-elasticsearch
 *		如果版本不适配：2.4.6
 *			1）、升级SpringBoot版本
 *			2）、安装对应版本的ES
 *
 * 		1）、Client 节点信息clusterNodes；clusterName
 * 		2）、ElasticsearchTemplate 操作es
 *		3）、编写一个 ElasticsearchRepository 的子接口来操作ES；
 *	两种用法：https://github.com/spring-projects/spring-data-elasticsearch
 *	1）、编写一个 ElasticsearchRepository
 */
```

### jest使用样例

```java
package com.atguigu.elastic;

import com.atguigu.elastic.bean.Article;
import com.atguigu.elastic.bean.Book;
import com.atguigu.elastic.repository.BookRepository;
import io.searchbox.client.JestClient;
import io.searchbox.core.Index;
import io.searchbox.core.Search;
import io.searchbox.core.SearchResult;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;

import java.io.IOException;
import java.util.List;

@RunWith(SpringRunner.class)
@SpringBootTest
public class Springboot03ElasticApplicationTests {

	@Autowired
	JestClient jestClient;

	@Autowired
	BookRepository bookRepository;

	@Test
	public void test02(){
//		Book book = new Book();
//		book.setId(1);
//		book.setBookName("西游记");
//		book.setAuthor("吴承恩");
//		bookRepository.index(book);


		for (Book book : bookRepository.findByBookNameLike("游")) {
			System.out.println(book);
		}
		;

	}




	@Test
	public void contextLoads() {
		//1、给Es中索引（保存）一个文档；
		Article article = new Article();
		article.setId(1);
		article.setTitle("好消息");
		article.setAuthor("zhangsan");
		article.setContent("Hello World");

		//构建一个索引功能
		Index index = new Index.Builder(article).index("atguigu").type("news").build();

		try {
			//执行
			jestClient.execute(index);
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	//测试搜索
	@Test
	public void search(){

		//查询表达式
		String json ="{\n" +
				"    \"query\" : {\n" +
				"        \"match\" : {\n" +
				"            \"content\" : \"hello\"\n" +
				"        }\n" +
				"    }\n" +
				"}";

		//更多操作：https://github.com/searchbox-io/Jest/tree/master/jest
		//构建搜索功能
		Search search = new Search.Builder(json).addIndex("atguigu").addType("news").build();

		//执行
		try {
			SearchResult result = jestClient.execute(search);
			System.out.println(result.getJsonString());
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

}


```

### springdata 使用样例

```java

//参照
    // https://docs.spring.io/spring-data/elasticsearch/docs/3.0.6.RELEASE/reference/html/
```





### 安装

安装es6.3.2要求jdk最低版本1.8

安装步骤：

1. https://www.elastic.co/cn/

2. 产品->Elasticsearch->Elasticsearch->下载->[past releases](https://www.elastic.co/downloads/past-releases#elasticsearch)

3. 下载压缩包->解压->双击运行bin/elaticsearch.bat

4. 浏览器输入localhost:9200能看到json说明安装成功

   ```json
   {
     "name": "CRkkVZ7",
     "cluster_name": "elasticsearch",
     "cluster_uuid": "cUI6hcDiSDmZ6w3d_P0-jQ",
     "version": {
       "number": "6.3.2",
       "build_flavor": "default",
       "build_type": "zip",
       "build_hash": "053779d",
       "build_date": "2018-07-20T05:20:23.451332Z",
       "build_snapshot": false,
       "lucene_version": "7.3.1",
       "minimum_wire_compatibility_version": "5.6.0",
       "minimum_index_compatibility_version": "5.0.0"
     },
     "tagline": "You Know, for Search"
   }
   ```

<u>kibana工具 安装步骤一致</u>

### docker结合使用

![docker_es](D:\dailySoftWare\typora\md\img\docker_es.png)

### postman交互

```json
Get 查看所有索引

localhost:9200/_all

PUT 创建索引-test

localhost:9200/test  


DEL 删除索引-test

localhost:9200/test  


PUT 创建索引-person-1

localhost:9200/person


PUT 新增数据-person-1

localhost:9200/person/_doc/1

{
	"first_name" : "John",
  "last_name" : "Smith",
  "age" : 25,
  "about" : "I love to go rock climbing",
  "interests" : [ "sports", "music" ]
}

PUT 新增数据-person-2

localhost:9200/person/_doc/2

{
	"first_name" : "Eric",
  "last_name" : "Smith",
  "age" : 23,
  "about" : "I love basketball",
  "interests" : [ "sports", "reading" ]
}

GET 搜索数据-person-id

localhost:9200/person/_doc/1

GET 搜索数据-person-name

localhost:9200/person/_doc/_search?q=first_name:john

{
  "took": 56,
  "timed_out": false,
  "_shards": {
    "total": 1,
    "successful": 1,
    "skipped": 0,
    "failed": 0
  },
  "hits": {
    "total": {
      "value": 1,
      "relation": "eq"
    },
    "max_score": 0.6931472,
    "hits": [
      {
        "_index": "person",
        "_type": "_doc",
        "_id": "1",
        "_score": 0.6931472,
        "_source": {
          "first_name": "John",
          "last_name": "Smith",
          "age": 25,
          "about": "I love to go rock climbing",
          "interests": [
            "sports",
            "music"
          ]
        }
      }
    ]
  }
}

```

### kibana交互

Kibana 是一个免费且开放的用户界面，对 Elasticsearch 数据进行可视化的平台。

http://localhost:5601/app/kibana

```json
GET _search
{
  "query": {
    "match_all": {}
  }
}


GET _all

GET /person/_doc/1

POST /person/_search
{
	"query": {
  	"bool": {
			"should": [
      	{"match": {
        		"first_name": "Eric"
        	}
   			}
      ]
  	}
}

POST /person/_search
{
	"query": {
  	"bool": {
			"should": [
      	{"match": {
        		"last_name": "Smith"
        	}
   			},
        {
        	"match": {
          	"about": "basketball"
          }
				}
      ]
  	}
  }
}



POST /person/_search
{
	"query": {
  	"bool": {
			"must": [
      	{"match": {
        		"last_name": "Smith"
        	}
   			},
        {
        	"match": {
          	"about": "basketball"
          }
				}
      ]
  	}
  }
}
```



### springboot集成

1、引入依赖

2、添加配置 服务器IP信息

3、接口继承 ElasticSearchRepository 实现增删改查



### mysql、es数据同步

全量、增量方式

#### go-mysql-elasticsearch

https://github.com/siddontang/go-mysql-elasticsearch

不足：不支持 ES6.X 以上、Mysql 8.X 以上



#### logstash

**mysql.conf**

```shell
bin/logstash -f config/mysql.conf
```



```xml
input{
    jdbc{
        # jdbc驱动包位置
        jdbc_driver_library => "/Users/gaohanghang/software/0ELK7.5/logstash-7.5.0/mysql-connector-java-5.1.31.jar"
        # 要使用的驱动包类
        jdbc_driver_class => "com.mysql.jdbc.Driver"
        # mysql数据库的连接信息
        jdbc_connection_string => "jdbc:mysql://127.0.0.1:3306/blog"
        # mysql用户
        jdbc_user => "root"
        # mysql密码
        jdbc_password => "root"
        # 定时任务，多久执行一次查询，默认一分钟，如果想要没有延迟，可以使用 schedule => "* * * * * *"
        schedule => "* * * * *"
        # 清空上传的sql_last_value记录
        clean_run => true
        # 你要执行的语句
        statement => "select * FROM t_blog WHERE update_time > :sql_last_value AND update_time < NOW() ORDER BY update_time desc"
    }
}

output {
    elasticsearch{
        # es host : port
        hosts => ["127.0.0.1:9200"]
        # 索引
        index => "blog"
        # _id
        document_id => "%{id}"
    }
}
```

### 分词器

<img src="C:\Users\10674\AppData\Roaming\Typora\typora-user-images\image-20201224151352271.png" alt="image-20201224151352271" style="zoom:50%;" />

#### IK分词器的安装和使用

```json
POST _analyze
{
  "analyzer": "standard",
  "text" : "hello imooc"
}
```

**ik分词器下载地址**

适合中国语言的分词器

下载之后，放于plugins目录下

https://github.com/medcl/elasticsearch-analysis-ik/releases

## 四、参考

[慕课网参考](https://www.yuque.com/gaohanghang/vx5cb2/aa576g)

[参考](https://www.cnblogs.com/dreamroute/p/8484457.html)

[参考2](https://www.jianshu.com/p/b50d7fdbe544)

