---
title: '搜索引擎'
layout: post
tags:
  - elasticsearch
  - java
category: 数据库
---
elasticsearch.

<!--more-->

# 一、docker安装

### 准备前置文件
用于存放配置、数据、插件
1. 文件夹
```shell
#其中elasticsearch.yml是挂载的配置文件，data是挂载的数据，plugins是es的插件，如ik，而数据挂载需要权限，需要设置data文件的权限为可读可写,需要下边的指令。
chmod -R 777 要修改的路径
/home/es/config/
/home/es/data/
/home/es/plugins/
```
2. /home/es/config/elasticsearch.yml 配置
```
http.cors.enabled: true
http.cors.allow-origin: "*"
http.host: 0.0.0.0
indices.breaker.total.limit: 80%
indices.fielddata.cache.size: 10%
indices.breaker.fielddata.limit: 60%
indices.breaker.request.limit: 40%
indices.breaker.total.use_real_memory: false

```
3. /home/es/data/ 权限
```
chmod -R 777 /home/es/data/
```

## 创建容器
```shell
#设置max_map_count不能启动es会启动不起来
cat /proc/sys/vm/max_map_count
sysctl -w vm.max_map_count=262144
sysctl -p
#拉取镜像
docker pull elasticsearch:7.7.0

#启动镜像 访问ip:9200
docker run --name elasticsearch -p 9200:9200 -p 9300:9300  -e "discovery.type=single-node" -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -v /mydata/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml -v /mydata/elasticsearch/data:/usr/share/elasticsearch/data -v /mydata/elasticsearch/plugins:/usr/share/elasticsearch/plugins -d elasticsearch:7.7.0

docker run -d --name elasticsearch \
--privileged=true \
-p 9200:9200 -p 9300:9300 \
-e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
-e "discovery.type=single-node" \
 -v /home/es/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
 -v /home/es/data:/usr/share/elasticsearch/data \
 -v /home/es/plugins:/usr/share/elasticsearch/plugins \
elasticsearch:7.7.0

#-e "discovery.type=single-node" 设置为单节点
#-e ES_JAVA_OPTS="-Xms256m -Xmx256m" \ 测试环境下，设置ES的初始内存和最大内存，否则导致过大启动不了ES

docker run --name elasticsearch -d -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -e "discovery.type=single-node" -p 9200:9200 -p 9300:9300 elasticsearch:7.7.0


```

## 安装并配置elasticsearch-head

```shell
#进入容器修改elasticsearch.yml允许跨域
docker exec -it elasticsearch /bin/bash 

vi config/elasticsearch.yml
#最下边增加两行
http.cors.enabled: true 
http.cors.allow-origin: "*"

#重启生效
docker restart elasticsearch

#elasticsearch-head
#拉取镜像
docker pull mobz/elasticsearch-head:5

#创建容器
docker create --name elasticsearch-head -p 9100:9100 mobz/elasticsearch-head:5

#修改vendor.js
docker cp elasticsearch-head:/usr/src/app/_site/vendor.js ./
vim vendor.js
#vim显示行号命令：set number
#6886行改为：contentType:"application/json;charset=UTF-8"
#7573行改为：contentType:"application/json;charset=UTF-8"
docker cp ./vendor.js elasticsearch-head:/usr/src/app/_site/

#启动容器 访问ip:9100
docker start elasticsearch-head
```

# ik分词器

https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v7.7.0/elasticsearch-analysis-ik-7.7.0.zip

```shell
#将压缩包移动到容器中
docker cp /tmp/elasticsearch-analysis-ik-7.7.0.zip elasticsearch:/usr/share/elasticsearch/plugins

#进入容器
docker exec -it elasticsearch /bin/bash  

#创建目录
mkdir /usr/share/elasticsearch/plugins/ik

#将文件压缩包移动到ik中
mv /usr/share/elasticsearch/plugins/elasticsearch-analysis-ik-7.7.0.zip /usr/share/elasticsearch/plugins/ik

#进入目录
cd /usr/share/elasticsearch/plugins/ik

#解压
unzip elasticsearch-analysis-ik-7.7.0.zip

#删除压缩包
rm -rf elasticsearch-analysis-ik-7.7.0.zip

```


## 二、配置
### 1、linux设置最大内存分片数
```
#默认食65530
cat /proc/sys/vm/max_map_count
sysctl -w vm.max_map_count=262144
```
### 2、宿主机文件&配置
```
mkdir /home/es/config
mkdir /home/es/data
mkdir /home/es/plugins

#权限
chmod -R 777 /home/es/data

#配置文件
vim /home/es/config/elasticsearch.yml
```
elasticsearch.yml
```
http.cors.enabled: true
http.cors.allow-origin: "*"
http.host: 0.0.0.0
```

### 4、访问9200端口验证

## 三、springboot 配置使用

### 1、依赖


```xml

<!-- Java High Level REST Client -->
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.7.0</version>
</dependency>

```

### 2、bean配置
```java
@Configuration
public class ElasticSearchConfig {


    @Bean
    public RestHighLevelClient restHighLevelClient() {
        RestHighLevelClient client = new RestHighLevelClient(
                RestClient.builder(
                        new HttpHost("172.17.181.4", 9200, "http")
                ));
        return client;
    }
}


```
### 3、DEMO - rest high level client

```

@SpringBootTest
class EsApiApplicationTests {

    @Autowired
    private RestHighLevelClient restHighLevelClient;


    /*1.创建索引*/
    @Test
    void createIndex() throws IOException {
        // 创建请求
//        CreateIndexRequest request = new CreateIndexRequest("article_index");
        CreateIndexRequest request = new CreateIndexRequest("article_index");
        // 执行请求,获取响应
        CreateIndexResponse response = restHighLevelClient.indices().create(request, RequestOptions.DEFAULT);
        System.out.println("creatIndex--" + response);
    }

    /*2.获取索引,判断是否存在*/
    @Test
    void getIndex() throws IOException {
        // 创建请求
        GetIndexRequest request = new GetIndexRequest("article_index");
        // 执行请求,获取响应
        boolean exists = restHighLevelClient.indices().exists(request, RequestOptions.DEFAULT);
        System.out.println("getIndex--exists--" + exists);
    }

    /*3.删除索引*/
    @Test
    void deleteIndex() throws IOException {
        // 创建请求
        DeleteIndexRequest request = new DeleteIndexRequest("article_index");
        // 执行请求
        AcknowledgedResponse response = restHighLevelClient.indices().delete(request, RequestOptions.DEFAULT);
        System.out.println("deleteIndex--" + response);
    }

    /*4.添加文档*/
    @Test
    void addDocument() throws IOException {
        // 创建对象
        ArticleSearchDto data = new ArticleSearchDto();
        data.setTitle("test demo api title");
        // 创建索引,即获取索引
        IndexRequest indexRequest = new IndexRequest("article_index");
        // 添加规则 /index/_doc/id
        indexRequest.id("4");
        indexRequest.timeout(TimeValue.timeValueSeconds(1));
        // 存入对象
        indexRequest.source(JSON.toJSONString(data), XContentType.JSON);
        // 发送请求
        IndexResponse response = restHighLevelClient.index(indexRequest, RequestOptions.DEFAULT);
        System.out.println("addDocument--" + response);
    }

    /*5.获取文档是否存在*/
    @Test
    void getDocument() throws IOException {
        // 创建get请求
        GetRequest getRequest = new GetRequest("article_index", "1");
        // 不获取返回的_source的上下文
        getRequest.fetchSourceContext(new FetchSourceContext(false));
        getRequest.storedFields("_none_");
        // 发送请求
        boolean exists = restHighLevelClient.exists(getRequest, RequestOptions.DEFAULT);
        System.out.println("addDocument--" + exists);

        /*6.获取文档信息*/
        if (exists) {
            // 创建请求
            GetRequest getRequest1 = new GetRequest("article_index", "1");
            // 发送请求
            GetResponse response = restHighLevelClient.get(getRequest1, RequestOptions.DEFAULT);
            System.out.println("source--" + response.getSource());
            System.out.println("getDocument--" + response.getSourceAsString());
        }

        /*7.更新文档信息*/
        if (exists) {
            // 创建请求
            UpdateRequest updateRequest = new UpdateRequest("article_index", "1");
            updateRequest.timeout("1s");
            // 修改内容
            ArticleSearchDto data = new ArticleSearchDto();
            data.setTitle("test2");
            updateRequest.doc(JSON.toJSONString(data), XContentType.JSON);
            UpdateResponse response = restHighLevelClient.update(updateRequest, RequestOptions.DEFAULT);
            System.out.println("updateDocument--" + response.status());
        }

        /*8.删除文档信息*/
        if (exists) {
            // 创建请求
            DeleteRequest deleteRequest = new DeleteRequest("article_index", "1");
            deleteRequest.timeout("1s");
            // 修改内容
            DeleteResponse response = restHighLevelClient.delete(deleteRequest, RequestOptions.DEFAULT);
            System.out.println("deleteDocument--" + response.status());
        }
    }

    /*9.批量添加文档*/
    @Test
    void batchAddDocument() throws IOException {
        // 批量请求
        BulkRequest bulkRequest = new BulkRequest();
        bulkRequest.timeout("10s");
        // 创建对象
        ArrayList<ArticleSearchDto> datas = new ArrayList<>();
        ArticleSearchDto d1 = new ArticleSearchDto();
        d1.setTitle("test hahha");
        ArticleSearchDto d2 = new ArticleSearchDto();
        d1.setTitle("test hahha111");
        ArticleSearchDto d3 = new ArticleSearchDto();
        d1.setTitle("test hahha5134");
        datas.add(d1);
        datas.add(d2);
        datas.add(d3);

        // 添加请求
        for (int i = 1; i < datas.size(); i++) {
            bulkRequest.add(new IndexRequest("article_index")
                    .id("" + (i + 2))
                    .source(JSON.toJSONString(datas.get(i)), XContentType.JSON));
        }
        // 执行请求
        BulkResponse response = restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        System.out.println("batchAddDocument--" + response.status());
    }

    /*10.查询*/
    @Test
    void search() throws IOException {
        // 创建请求
        SearchRequest searchRequest = new SearchRequest("article_index");
        // 构建搜索条件
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
//
//        MultiMatchQueryBuilder query = QueryBuilders.multiMatchQuery("框架技巧", "title", "queryList", "content");
//        query.lenient();
//
//        DisMaxQueryBuilder disMaxQueryBuilder = QueryBuilders.disMaxQuery();
//        disMaxQueryBuilder.add(query);

        MatchPhrasePrefixQueryBuilder query = QueryBuilders.matchPhrasePrefixQuery("title", "品专");

        searchSourceBuilder.query(query);
        searchSourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS));
        searchSourceBuilder.fetchSource(new String[]{"title"}, null);
        searchSourceBuilder.size(10);

        searchRequest.source(searchSourceBuilder);
        // 执行请求
        SearchResponse response = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
        // 解析查询结果
//        List<Long> result = new ArrayList<>();
        for (SearchHit documentFields : response.getHits()) {
//            result.add(Long.valueOf(documentFields.getSourceAsMap().get("id").toString()));
            System.out.println(documentFields.getSourceAsMap());
        }
//        System.out.println(result);
    }

}


```

## 四、语法说明
https://blog.csdn.net/qq_44850489/article/details/119182010

https://blog.csdn.net/qq_37158147/article/details/124431295

## 五、ik分词器

### 1、下载对应板块的包
https://github.com/medcl/elasticsearch-analysis-ik

### 2、安装
```
#在plugin下创建ik文件夹
mkdir /home/es/plugin/ik

#将解压包解压到ik下
tar -zxvf xxxx.tar.gz

#重启es服务
docker restart elasticsearch

```

### 3、检验
```
GET /_analyze
{
  "text": "中华人民共和国国歌",
  "analyzer": "ik_smart"
}
```
· ik_max_word 会将文本做最细粒度的拆分

比如会将「中华人民共和国国歌」拆分为：中华人民共和国、中华人民、中华、华人、人民共和国、人民、人、民、共和国、共和、和、国国、国歌，会穷尽各种可能的组合；

· ik_smart 最粗粒度的拆分

比如会将「中华人民共和国国歌」拆分为：中华人民共和国、国歌。