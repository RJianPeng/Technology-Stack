# caffeine
一个高性能的Java缓存库。caffeine使用了Window TinyLfu回收策略，提供了一个近乎绝佳的命中率。

## caffeine的三种同步策略
### 手动加载
```
Cache<String, Object> cache = Caffeine.newBuilder()
        //写入后的过期时间
        .expireAfterWrite(10, TimeUnit.MINUTES)
        //缓存中最大存储数量
        .maximumSize(10_000)
        .build();
 
String key = "key";
// 根据Key查询一个缓存，如果没有调用createExpensiveGraph方法，并采用手动的方式将返回值保存到缓存。
graph = cache.get(key, k -> createExpensiveGraph(k));
// 手动将一个值放入缓存，如果以前有值就覆盖以前的值
cache.put(key, graph);
// 删除缓存
cache.invalidate(key);
```

### 同步加载
```
//方式一
LoadingCache<String, Object> loadingCache = Caffeine.newBuilder()
        .maximumSize(10_000)
        .expireAfterWrite(10, TimeUnit.MINUTES)
        //用于获取缓存中没有的值的方法
        .build(key -> createExpensiveGraph(key));
    
String key = "key";
// 查询并在缺失的情况下使用同步的方式即createExpensiveGraph方法来查询到值
//caffeine会将该方法返回值同步到缓存中，不需要使用方手动执行
Object graph = loadingCache.get(key);





//方式二
LoadingCache<String,Object> loadingCache =  Caffeine.newBuilder()
                .recordStats()
                .maximumSize(1000)
                .expireAfterWrite(10, TimeUnit.MINUTES)
                .refreshAfterWrite(5, TimeUnit.MINUTES)
                .build(localCacheLoader);
```

### 异步加载
```
AsyncLoadingCache<String, Object> asyncLoadingCache = Caffeine.newBuilder()
            .maximumSize(10_000)
            .expireAfterWrite(10, TimeUnit.MINUTES)
            //异步同步时获取缓存中没有的值的方法
            .buildAsync(key -> createExpensiveGraph(key));
 
String key = "name1";
 
// 查询并在缺失的情况下异步的调用createExpensiveGraph方式来构建缓存
CompletableFuture<Object> graph = asyncLoadingCache.get(key);
// 查询一组缓存并在缺失的情况下使用异步的方式来构建缓存
List<String> keys = new ArrayList<>();
keys.add(key);
CompletableFuture<Map<String, Object>> graphs = asyncLoadingCache.getAll(keys);
// 异步cache转同步cache
loadingCache = asyncLoadingCache.synchronous();
```


## caffeine使用中的一些重要参数
* expireAfterWrite：自最后一次写入后的过期时间
* expireAfterAccess：自最后一次访问（读/写）的过期时间
（afterWrite和afterAccess同时配置的时候不报错，但是access已经包含了write TODO 测试具体生效的配置）
* refreshAfterWrite(重要配置):数据刷新策略，在写入时间阈值后更新数据



## caffeine使用踩坑
* 1.重复设置配置并不会覆盖老的配置，而是直接抛异常（TODO 这个地方后面看caffeine的源码时可以看看）
