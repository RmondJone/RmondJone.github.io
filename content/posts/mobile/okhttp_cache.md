---
title: "OKHttp3缓存策略"
date: 2023-04-10T16:22:38+08:00
draft: false
categories: ["移动端"]
tags: ["Okhttp","Android"]
---

从上篇文章[《OkHttp源码深入》](/posts/mobile/okhttp)中，我们知道okhttp最后所有的请求都会经过getResponseWithInterceptorChain方法，进行拦截处理。面试中常问的拦截策略必属缓存拦截器CacheInterceptor的实现。

## OKHttp中缓存策略
* 强制缓存，即缓存在有效期内就直接返回缓存，不进行网络请求。
* 对比缓存，即缓存超过有效期，进行网络请求。若数据未修改，服务端返回不带body的304响应，表示客户端缓存仍有效可用；否则返回完整最新数据，客户端取网络请求的最新数据

这里先做一个总结：
OkHttp中缓存机制实现大致流程是先判断是否执行强制缓存策略，不执行则请求服务端获取数据，然后判断执行对比缓存策略，接着更新缓存或新增缓存，最后返回response到上层拦截器

## CacheInterceptor源码解析
```java
@Override public Response intercept(Chain chain) throws IOException {
    // cache即构建OkHttpClient时传入的Cache对象的内部成员，用request的url作为key，查找缓存的response作为候选缓存
    Response cacheCandidate = cache != null
        ? cache.get(chain.request())
        : null;

    long now = System.currentTimeMillis();

    //todo === 1.生成缓存策略 ===
    CacheStrategy strategy = new CacheStrategy.Factory(now, chain.request(), cacheCandidate).get();
    // 获取缓存策略生成的networkRequest和cacheResponse(合法缓存)
    // 下面流程会根据生成的networkRequest和cacheResponse来决定执行什么缓存策略
    Request networkRequest = strategy.networkRequest;
    Response cacheResponse = strategy.cacheResponse;

    if (cache != null) {
      // cache内部的计数器加1
      cache.trackResponse(strategy);
    }

    // 若候选缓存存在但是缓存策略生成的cache不存在，关闭cacheCandidate中的BufferedSource的输入输出流
    if (cacheCandidate != null && cacheResponse == null) {
      closeQuietly(cacheCandidate.body()); // The cache candidate wasn't applicable. Close it.
    }

    // If we're forbidden from using the network and the cache is insufficient, fail.
    // 即无网络请求又无合法缓存，返回状态码504的response
    if (networkRequest == null && cacheResponse == null) {
      return new Response.Builder()
          .request(chain.request())
          .protocol(Protocol.HTTP_1_1)
          .code(504)
          .message("Unsatisfiable Request (only-if-cached)")
          .body(Util.EMPTY_RESPONSE)
          .sentRequestAtMillis(-1L)
          .receivedResponseAtMillis(System.currentTimeMillis())
          .build();
    }

    // If we don't need the network, we're done.
    // 强制缓存策略
    if (networkRequest == null) {
      return cacheResponse.newBuilder()
          .cacheResponse(stripBody(cacheResponse))
          .build();
    }

    // networkResponse表示当前网络请求的最新response，cacheResponse表示由缓存策略获取的合法缓存response
    Response networkResponse = null;
    try {
      // 调用下层拦截器进行网络请求
      networkResponse = chain.proceed(networkRequest);
    } finally {
      // If we're crashing on I/O or otherwise, don't leak the cache body.
      if (networkResponse == null && cacheCandidate != null) {
        closeQuietly(cacheCandidate.body());
      }
    }

    // If we have a cache response too, then we're doing a conditional get.
    if (cacheResponse != null) {
      // 对比缓存策略
      if (networkResponse.code() == HTTP_NOT_MODIFIED) {
        // 服务端返回304状态码，表示本地缓存仍有效
        Response response = cacheResponse.newBuilder()
            .headers(combine(cacheResponse.headers(), networkResponse.headers()))
            .sentRequestAtMillis(networkResponse.sentRequestAtMillis())
            .receivedResponseAtMillis(networkResponse.receivedResponseAtMillis())
            .cacheResponse(stripBody(cacheResponse))
            .networkResponse(stripBody(networkResponse))
            .build();
        networkResponse.body().close();

        // Update the cache after combining headers but before stripping the
        // Content-Encoding header (as performed by initContentStream()).
        // cache内计数器加1
        cache.trackConditionalCacheHit();
        // 更新本地缓存信息
        cache.update(cacheResponse, response);
        return response;
      } else {
        closeQuietly(cacheResponse.body());
      }
    }

    Response response = networkResponse.newBuilder()
        .cacheResponse(stripBody(cacheResponse))
        .networkResponse(stripBody(networkResponse))
        .build();

  //todo === 2.判断能否缓存networkResponse ===
  if (cache != null) {
  // OkHttpClient设置了Cache
  if (HttpHeaders.hasBody(response) && CacheStrategy.isCacheable(response, networkRequest)) {
    // Offer this request to the cache.
    // 保存拥有body且符合缓存条件的响应
    // put方法中只缓存GET请求的response
    CacheRequest cacheRequest = cache.put(response);
    // 通过该方法返回新建的repsonse，保存缓存response的body能够正确的写入和关闭流
    return cacheWritingResponse(cacheRequest, response);
  }

  // 判断是否时无效缓存，若请求方式是POST、PATCH、PUT、DELETE、MOVE，则移除缓存
  if (HttpMethod.invalidatesCache(networkRequest.method())) {
    try {
      cache.remove(networkRequest);
    } catch (IOException ignored) {
      // The cache cannot be written.
    }
  }
}

    return response;
}
```


