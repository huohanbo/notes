# 基本使用
```
@Component
public class MongoReadWrapper {
    private static final String COLLECTION_NAME = "demo";

    @Autowired
    private MongoTemplate mongoTemplate;
}
```

## 1. 根据字段进行查询
```
    /**
     * 指定field查询
     */
    public void specialFieldQuery() {
        Query query = new Query(Criteria.where("user").is("一灰灰blog"));
        // 查询一条满足条件的数据
        Map result = mongoTemplate.findOne(query, Map.class, COLLECTION_NAME);
        System.out.println("query: " + query + " | specialFieldQueryOne: " + result);

        // 满足所有条件的数据
        List<Map> ans = mongoTemplate.find(query, Map.class, COLLECTION_NAME);
        System.out.println("query: " + query + " | specialFieldQueryAll: " + ans);
    }
```
上面是一个实际的case，从中可以知道一般的查询方式为:
- Criteria.where(xxx).is(xxx)来指定具体的查询条件
- 封装Query对象 new Query(criteria)
- 借助mongoTemplate执行查询 mongoTemplate.findOne(query, resultType, collectionName)

findOne表示只获取一条满足条件的数据；
find则会将所有满足条件的返回。

## 2. and多条件查询
```
/**
 * 多个查询条件同时满足
 */
public void andQuery() {
    Query query = new Query(Criteria.where("user").is("一灰灰blog").and("age").is(18));
    Map result = mongoTemplate.findOne(query, Map.class, COLLECTION_NAME);
    System.out.println("query: " + query + " | andQuery: " + result);
}
```