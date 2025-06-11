# Elasticsearch 基本使用

> Elasticsearch(简称ES)是一个基于Apache Lucene构建的开源、分布式、RESTful接口的全文搜索引擎，Elasticsearch通过对Lunece的封装，隐藏了复杂性，提供了使用简单的RESTful Api。Elasticsearch还是一个分布式文档数据库，其中每个字段均可被索引，而且每个字段的数据均可被搜索，因为对文档进行了分词处理。ES能够横向扩展至数以百计的服务器存储以及处理PB级的数据，可以在极短的时间内存储、搜索和分析大量的数据。Elasticsearch 是用 Java 开发的，并作为 Apache 许可条款下的开放源码发布，是当前流行的企业级搜索引擎。设计用于云计算中，能够达到实时搜索，稳定，可靠，快速，安装使用方便

[Elasticsearch官网](https://www.elastic.co/cn/)

### 在项目中引入ElasticSearch JAVA SDK

**Maven**

```
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>7.6.2</version>
</dependency>
```

**Gradle**

	implementation 'org.elasticsearch.client:elasticsearch-rest-high-level-client:7.6.2'

**yaml**

```
elasticsearch:
  cluster-name: test-otc-cluster
  cluster-nodes: 127.0.0.1
  port: 9301
  pool: 5
```

###配置ElasticSearch  Java High Level REST Client 操作对象

初始化RestHighLevelClient 并交给容器管理


```
@Bean(name = "restHighLevelClient")
    public RestHighLevelClient restHighLevelClient() {
        //可以传httpHost数组
        String[] nodes = esNodes.split(",");
        HttpHost[] httpHosts = new HttpHost[nodes.length];
        for (int i = 0; i < httpHosts.length; i++) {
            httpHosts[i] = new HttpHost(nodes[i], ES_PORT, "http");
        }

        RestClientBuilder builder = RestClient.builder(httpHosts);
        // 异步httpclient的连接延时配置
        builder.setRequestConfigCallback(requestConfigBuilder -> {
            //设置超时
            requestConfigBuilder.setConnectTimeout(CONNECT_TIME_OUT);
            requestConfigBuilder.setSocketTimeout(SOCKET_TIME_OUT);
            return requestConfigBuilder.setConnectionRequestTimeout(CONNECTION_REQUEST_TIME_OUT);
        });
        // 异步httpclient的连接数配置
        builder.setHttpClientConfigCallback(requestConfigBuilder -> {
            requestConfigBuilder.setMaxConnTotal(MAX_CONNECT_NUM);
            return requestConfigBuilder.setMaxConnPerRoute(MAX_CONNECT_PER_ROUTE);
        });

        return new RestHighLevelClient(builder);
    }
```

###RestHighLevelClient 相关操作

基于7.6.2封装到工具类

```
public class EsClientUtils {
    /**
     * The official recommendation is 1000-5000
     */
    private static int batchSize = 1000;

    private static String EsPreTags = "<span style='color:red' >";

    private static String EsPostTags = "</span>";

    private static String baseScript = "ctx._source['%s']=params.%s;";


    public enum EsRangeEnums {
        /**
         * 开始值
         */
        START,
        /**
         * 结束值
         */
        END,

        ;

    }

    @AllArgsConstructor
    @Getter
    public enum AggregationEnums {
        /**
         * 聚合
         */
        MAX("max", new MaxAggregationBuilder("max")),

        MIN("min", new MinAggregationBuilder("min")),

        AVG("avg", new AvgAggregationBuilder("avg")),

        SUM("sum", new SumAggregationBuilder("sum")),

        COUNT("count", new ValueCountAggregationBuilder("count", ValueType.LONG)),
        ;

        private  String code;


        private  ValuesSourceAggregationBuilder aggregationBuilder;
    }


    private static RestHighLevelClient restHighLevelClient;


    @Autowired
    public void init(RestHighLevelClient restHighLevelClient) {
        EsClientUtils.restHighLevelClient = restHighLevelClient;
    }


    /**
     * 判断索引是否存在
     *
     * @param index
     * @return
     */
    public static boolean isIndexExist(String index) throws IOException {
        return restHighLevelClient.indices().exists(new GetIndexRequest(index), RequestOptions.DEFAULT);
    }


    /**
     * 创建索引
     *
     * @param index
     * @return
     */
    public static boolean createIndex(String index) throws IOException {
        if (isIndexExist(index)) {
            CreateIndexRequest request = new CreateIndexRequest(index);
            CreateIndexResponse createIndexResponse = restHighLevelClient.indices().create(request, RequestOptions.DEFAULT);
            return createIndexResponse.isAcknowledged();
        }
        log.info("The index already exists");
        return false;
    }


    /**
     * 删除索引
     *
     * @param index
     * @return
     */
    public static boolean deleteIndex(String index) throws IOException {
        if (!isIndexExist(index)) {
            log.info("Index is not exits!");
        }
        return restHighLevelClient.indices().delete(new DeleteIndexRequest(index), RequestOptions.DEFAULT).isAcknowledged();
    }

    /**
     * @param dataMap <文档ID,数据源>
     * @param index   索引
     * @throws IOException
     */
    public static void batchAddToElasticSearch(Map<String, Object> dataMap, String index) throws IOException {
        //批量添加
        BulkRequest bulkRequest = new BulkRequest();
        int inc = 0;
        for (Map.Entry<String, Object> data : dataMap.entrySet()) {
            inc++;
            Object value = data.getValue();
            bulkRequest.add(new IndexRequest(index).id(data.getKey()).source(JSON.toJSONString(value), XContentType.JSON).opType(DocWriteRequest.OpType.INDEX));
            if (inc >= batchSize) {
                inc = 0;
                restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
            }
        }
        if (inc >= dataMap.size() % batchSize) {
            restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        }
    }

    /**
     * 数据添加，正定ID
     *
     * @param index 索引，类似数据库
     * @param id    数据ID
     * @return
     */
    public static String addData(String json, String index, String id) throws IOException {
        log.info("Add data to cache|input:[{}]", json);
        IndexResponse response = restHighLevelClient.index(new IndexRequest(index).id(id).source(json, XContentType.JSON), RequestOptions.DEFAULT);
        log.info("Add data to cache|output:status:[{}],id:[{}]", response.status().getStatus(), response.getId());
        return response.getId();
    }


    /**
     * 通过ID删除数据
     *
     * @param index 索引，类似数据库
     * @param id    数据ID
     */
    public static void deleteDataById(String index, String id) throws IOException {
        DeleteResponse response = restHighLevelClient.delete(new DeleteRequest(index, id), RequestOptions.DEFAULT);
        log.info("deleteDataById response status:{},id:{}", response.status().getStatus(), response.getId());
    }

    /**
     * 根据条件删除索引
     *
     * @param params
     * @param index
     * @throws IOException
     */
    public static void deleteIndexByQuery(Map<String, Object> params, String... index) throws IOException {
        DeleteByQueryRequest request = new DeleteByQueryRequest(index);
        request.setConflicts("proceed");
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        exactFieldMatch(params, boolQueryBuilder);
        request.setQuery(boolQueryBuilder);
        restHighLevelClient.deleteByQuery(request, RequestOptions.DEFAULT);
    }


    /**
     * 通过IDs批量删除数据
     *
     * @param index 索引，类似数据库
     * @param ids   数据IDs
     */
    public static void batchDeleteData(String index, List<String> ids) throws IOException {
        //批量删除
        BulkRequest bulkRequest = new BulkRequest();
        int inc = 0;
        for (String id : ids) {
            inc++;
            bulkRequest.add(new DeleteRequest(index, id));
            if (inc >= batchSize) {
                inc = 0;
                restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
            }
        }
        if (inc >= ids.size() % batchSize) {
            restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        }
    }


    /**
     * 通过ID 更新数据
     *
     * @param jsonObject 要增加的数据
     * @param index      索引，类似数据库
     * @param id         数据ID
     * @return
     */
    public static void updateDataById(String jsonObject, String index, String id) throws IOException {
        UpdateRequest request = new UpdateRequest(index, id).doc(jsonObject, XContentType.JSON).docAsUpsert(true);
        restHighLevelClient.update(request, RequestOptions.DEFAULT);
    }

    /**
     * 批量更新数据
     *
     * @param map   数据map
     * @param index 索引，类似数据库
     */
    public static void batchUpdateData(Map<String, Object> map, String index) throws IOException {
        //批量添加
        BulkRequest bulkRequest = new BulkRequest();
        int inc = 0;
        for (Map.Entry<String, Object> data : map.entrySet()) {
            inc++;
            bulkRequest.add(new UpdateRequest(index, data.getKey()).doc(JSON.toJSONString(data.getValue()), XContentType.JSON).docAsUpsert(true));
            if (inc >= batchSize) {
                inc = 0;
                restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
            }
        }
        if (inc >= map.size() % batchSize) {
            restHighLevelClient.bulk(bulkRequest, RequestOptions.DEFAULT);
        }
    }


    /**
     * 根据条件更新索引
     *
     * @param index    索引
     * @param painless painless脚本
     * @param params   条件
     * @param data     数据
     * @throws IOException
     */
    public static void bulkUpdateByScript(String index, String painless, Map<String, Object> params, Map<String, Object> data) throws IOException {
        UpdateByQueryRequest request = new UpdateByQueryRequest(index);

        //设置版本忽略
        request.setConflicts("proceed");

        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();

        // 完全匹配过滤
        exactFieldMatch(params, boolQueryBuilder);

        request.setQuery(boolQueryBuilder);

        request.setScript(new Script(ScriptType.INLINE, "painless", painless, data));

        BulkByScrollResponse bulkByScrollResponse = restHighLevelClient.updateByQuery(request, RequestOptions.DEFAULT);

        //打印的内容 可以在 Elasticsearch head 和 Kibana  上执行查询
        log.info("\n{}", request);
        long total = bulkByScrollResponse.getTotal();
        log.info("更新条数：[{}]|入参：[{}]", total, params);
    }


    /**
     * 复制索引
     *
     * @param target 目标
     * @param source 源
     * @throws IOException
     */
    public static void copyIndex(String target, String... source) throws IOException {
        ReindexRequest request = new ReindexRequest();
        request.setSourceIndices(source);
        request.setDestDocType("create");
        request.setDestIndex(target);
        restHighLevelClient.reindex(request, RequestOptions.DEFAULT);
    }


    /**
     * 根据条件更新索引
     *
     * @param index  索引
     * @param params 条件
     * @param data   数据
     * @throws IOException
     */
    public static void bulkUpdateByScript(String index, Map<String, Object> params, Map<String, Object> data) throws IOException {
        EsClientUtils.bulkUpdateByScript(index, EsClientUtils.scriptBuilder(data), params, data);
    }

    public static String scriptBuilder(Map<String, Object> data) {
        StringBuilder builder = new StringBuilder();
        for (Map.Entry<String, Object> entry : data.entrySet()) {
            builder.append(String.format(EsClientUtils.baseScript, entry.getKey(), entry.getKey()));
        }
        String script = builder.toString();
        log.info("\n script:[{}]", script);
        return script;
    }

    /**
     * 聚合查询（小数精度丢失）
     *
     * @param field            域
     * @param aggregationEnums 类型
     * @param params           条件
     * @param index            索引
     * @return
     * @throws IOException
     */
    public static BigDecimal aggregationByQuery(String field, AggregationEnums aggregationEnums, Map<String, Object> params, String... index) throws IOException {
        SearchRequest request = new SearchRequest(index);
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder()
                .size(0)
                .aggregation(aggregationEnums.getAggregationBuilder().field(field));
        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();
        exactFieldMatch(params, boolQueryBuilder);
        searchSourceBuilder.query(boolQueryBuilder);
        request.source(searchSourceBuilder);
        //打印的内容 可以在 Kibana  上执行查询
        log.info("\n{}", searchSourceBuilder);
        SearchResponse search = restHighLevelClient.search(request, RequestOptions.DEFAULT);
        NumericMetricsAggregation.SingleValue maxTemperature = search.getAggregations().get(aggregationEnums.getCode());
        return new BigDecimal(maxTemperature.value());
    }

    /**
     * @param index          索引名称
     * @param page           当前页（深度分页请考虑scroll）
     * @param size           每页条数
     * @param highlightField 高亮
     * @param params         条件
     * @param sorts          排序
     * @param rangeParams    范围
     * @param matchParams    全文检索
     * @param isFuzzy        是否分词
     * @return
     * @throws IOException
     */
    public static EsPageResult searchDataPage(String index,
                                              int page,
                                              int size,
                                              String highlightField,
                                              Map<String, Object> params,
                                              Map<String, SortOrder> sorts,
                                              Map<String, Map<EsRangeEnums, Object>> rangeParams,
                                              Map<String, Object> matchParams,
                                              boolean isFuzzy) throws IOException {

        SearchRequest searchRequest = new SearchRequest(index);
        SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();

        BoolQueryBuilder boolQueryBuilder = QueryBuilders.boolQuery();

        // 完全匹配过滤
        exactFieldMatch(params, boolQueryBuilder);

        //全文检索
        fuzzyFieldSearch(matchParams, boolQueryBuilder, isFuzzy);

        //排序
        sortByField(sorts, searchSourceBuilder);

        //高亮
        setHighlightField(highlightField, searchSourceBuilder);

        //范围查询
        rangeByField(boolQueryBuilder, rangeParams);

        page = Math.max(page, 0);
        size = Math.max(size, 1);

        searchSourceBuilder.query(boolQueryBuilder).from(page).size(size);
        searchRequest.source(searchSourceBuilder);
        SearchResponse searchResponse = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);

        //打印的内容 可以在 Kibana  上执行查询
        log.info("\n{}", searchSourceBuilder);
        return handleResponseOfData(searchResponse, page, size, highlightField);
    }

    private static void rangeByField(BoolQueryBuilder boolQueryBuilder, Map<String, Map<EsRangeEnums, Object>> rangeParams) {
        for (Map.Entry<String, Map<EsRangeEnums, Object>> entry : rangeParams.entrySet()) {
            boolQueryBuilder.must(QueryBuilders.rangeQuery(entry.getKey())
                    .from(entry.getValue().get(EsRangeEnums.START))
                    .to(entry.getValue().get(EsRangeEnums.END)));
        }
    }

    private static void setHighlightField(String highlightField, SearchSourceBuilder searchSourceBuilder) {
        if (StringUtils.isNotBlank(highlightField)) {
            HighlightBuilder highlightBuilder = new HighlightBuilder();
            highlightBuilder.preTags(EsClientUtils.EsPreTags);
            highlightBuilder.postTags(EsClientUtils.EsPostTags);
            highlightBuilder.field(highlightField);
            searchSourceBuilder.highlighter(highlightBuilder);
        }
    }

    private static void sortByField(Map<String, SortOrder> sorts, SearchSourceBuilder searchSourceBuilder) {
        for (Map.Entry<String, SortOrder> entry : sorts.entrySet()) {
            searchSourceBuilder.sort(entry.getKey(), entry.getValue());
        }
    }

    private static void fuzzyFieldSearch(Map<String, Object> matchParams, BoolQueryBuilder boolQueryBuilder, boolean isFuzzy) {
        for (Map.Entry<String, Object> entry : matchParams.entrySet()) {
            if (isFuzzy) {
                boolQueryBuilder.should(QueryBuilders.matchQuery(entry.getKey(), entry.getValue()));
            } else {
                boolQueryBuilder.should(QueryBuilders.matchPhraseQuery(entry.getKey(), entry.getValue()));
            }
        }
    }

    private static void exactFieldMatch(Map<String, Object> params, BoolQueryBuilder boolQueryBuilder) {
        for (Map.Entry<String, Object> entry : params.entrySet()) {
            boolQueryBuilder.filter(QueryBuilders.termQuery(entry.getKey(), entry.getValue()));
        }
    }

    private static EsPageResult handleResponseOfData(SearchResponse searchResponse, int page, int size, String highlightField) {
        if (Objects.equals(RestStatus.OK, searchResponse.status())) {

            JSONArray jsonArray = new JSONArray();

            SearchHits hits = searchResponse.getHits();

            long rowTotal = hits.getTotalHits().value;

            long pageTotal = (rowTotal % size == 0) ? rowTotal / size : rowTotal / size + 1;

            for (SearchHit hit : hits) {

                if (StringUtils.isNotBlank(highlightField)) {

                    Text[] text = hit.getHighlightFields().get(highlightField).getFragments();

                    if (text != null) {
                        for (Text str : text) {
                            //遍历 高亮结果集，覆盖 正常结果集
                            hit.getSourceAsMap().put(highlightField, str.string());
                        }
                    }
                }
                jsonArray.add(hit.getSourceAsMap());
            }

            return EsPageResult.builder().pages(pageTotal).current(page).total(rowTotal).pageSize(size).dataJson(jsonArray.toJSONString()).build();
        }
        return null;
    }


    /**
     * @param index  索引名称
     * @param params 条件
     * @param page   当前页
     * @param size   每页显示条数
     * @return
     * @throws IOException
     */
    public static EsPageResult searchDataPage(String index, Map<String, Object> params, int page, int size) throws IOException {
        return searchDataPage(index, page, size, StringUtils.EMPTY, params, Collections.emptyMap(), Collections.emptyMap(), Collections.emptyMap(), false);
    }


    /**
     * @param params 条件
     * @param index  索引名称
     * @param page   当前页
     * @param size   每页显示条数
     * @param sorts  排序
     * @return
     * @throws IOException
     */
    public static EsPageResult searchDataPage(String index, Map<String, Object> params, int page, int size, Map<String, SortOrder> sorts) throws IOException {
        return searchDataPage(index, page, size, StringUtils.EMPTY, Collections.emptyMap(), sorts, Collections.emptyMap(), Collections.emptyMap(), false);
    }

    /**
     * @param params 条件
     * @param index  索引名称
     * @param page   当前页
     * @param size   每页显示条数
     * @param sorts  排序
     * @return
     * @throws IOException
     */
    public static EsPageResult searchDataPage(String index,
                                              int page,
                                              int size, Map<String, Object> params,
                                              Map<String, Map<EsRangeEnums, Object>> rangeParams,
                                              Map<String, SortOrder> sorts) throws IOException {
        return searchDataPage(index, page, size, StringUtils.EMPTY, params, sorts, rangeParams, Collections.emptyMap(), false);
    }


    /**
     * 范围查询,并分页
     *
     * @param index       索引名称
     * @param rangeParams 范围
     * @param page        当前页
     * @param size        每页显示条数
     * @return
     * @throws IOException
     */
    public static EsPageResult searchDataPageByRange(String index, Map<String, Map<EsRangeEnums, Object>> rangeParams, int page, int size) throws IOException {
        return searchDataPage(index, page, size, StringUtils.EMPTY, Collections.emptyMap(), Collections.emptyMap(), rangeParams, Collections.emptyMap(), false);
    }

    /**
     * @param index       索引名称
     * @param rangeParams 范围
     * @param page        当前页
     * @param size        每页显示条数
     * @param sorts       排序
     * @return
     * @throws IOException
     */
    public static EsPageResult searchDataPageByRange(String index, Map<String, Map<EsRangeEnums, Object>> rangeParams, int page, int size, Map<String, SortOrder> sorts) throws IOException {
        return searchDataPage(index, page, size, StringUtils.EMPTY, Collections.emptyMap(), sorts, rangeParams, Collections.emptyMap(), false);
    }

    /**
     * 条件范围查询,并分页
     *
     * @param index       索引名称
     * @param params      条件
     * @param rangeParams 范围
     * @param page        当前页
     * @param size        每页显示条数
     * @return
     * @throws IOException
     */
    public static EsPageResult searchDataPageByRange(String index, Map<String, Object> params, Map<String, Map<EsRangeEnums, Object>> rangeParams, int page, int size) throws IOException {
        return searchDataPage(index, page, size, StringUtils.EMPTY, params, Collections.emptyMap(), rangeParams, Collections.emptyMap(), false);
    }


    /**
     * @param index       索引名称
     * @param matchParams 全文检索条件
     * @param page        当前页
     * @param size        每页显示条数
     * @param isFuzzy     是否分词
     * @param sorts       排序
     * @return
     * @throws IOException
     */
    public static EsPageResult searchDataPageByMatch(String index, Map<String, Object> matchParams, int page, int size, boolean isFuzzy, Map<String, SortOrder> sorts) throws IOException {
        return searchDataPage(index, page, size, StringUtils.EMPTY, Collections.emptyMap(), sorts, Collections.emptyMap(), matchParams, isFuzzy);
    }

    /**
     * @param index       索引名称
     * @param matchParams 全文检索条件
     * @param page        当前页
     * @param size        每页显示条数
     * @param isFuzzy     是否分词
     * @return
     * @throws IOException
     */
    public static EsPageResult searchDataPageByMatch(String index, Map<String, Object> matchParams, int page, int size, boolean isFuzzy) throws IOException {
        return searchDataPage(index, page, size, StringUtils.EMPTY, Collections.emptyMap(), Collections.emptyMap(), Collections.emptyMap(), matchParams, isFuzzy);
    }


    /**
     * @param index          索引名称
     * @param matchParams    全文检索条件
     * @param highlightField 高亮字段
     * @param page           当前页
     * @param size           每页显示条数
     * @param isFuzzy        是否分词
     * @return
     * @throws IOException
     */
    public static EsPageResult searchDataPageByMatch(String index, Map<String, Object> matchParams, String highlightField, int page, int size, boolean isFuzzy) throws IOException {
        return searchDataPage(index, page, size, highlightField, Collections.emptyMap(), Collections.emptyMap(), Collections.emptyMap(), matchParams, isFuzzy);
    }

    /**
     * 范围构建器
     *
     * @param rangeStart
     * @param rangeEnd
     * @return
     */
    public static Map<EsRangeEnums, Object> range(Object rangeStart, Object rangeEnd) {
        HashMap<EsRangeEnums, Object> rangeMap = new HashMap<>(0x10);
        rangeMap.put(EsRangeEnums.START, rangeStart);
        rangeMap.put(EsRangeEnums.END, rangeEnd);
        return rangeMap;
    }

    public static Map<String, Map<EsRangeEnums, Object>> rangeBuilder() {
        return new HashMap<>(0x10);
    }
```