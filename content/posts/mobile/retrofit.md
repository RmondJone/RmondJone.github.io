---
title: "Retrofit网络框架介绍"
date: 2023-04-10T15:49:48+08:00
draft: false
categories: ["移动端"]
tags: ["Retrofit","Android"]
---

## 一、Retrofit是什么？
官网介绍：A type-safe HTTP client for Android and Java。Retrofit是一个类型安全的Android和Java网络Http请求框架。
## 二、Retrofit的使用方式
### 接口定义
```java
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```
### 创建Retrofit实例
```java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();
```
### 调用接口

```java
GitHubService service = retrofit.create(GitHubService.class);

Call<List<Repo>> repos = service.listRepos("octocat");
```
## 三、API文档

### 每个方法都必须有一个HTTP注释，提供请求方法和相对URL路径。内置的请求注解有5种方式：@GET、@POST、@PUT、@DELETE、@HADE

```java
@GET("users/list")
```
你也可以把请求参数放置在注解里

```java
@GET("users/list?sort=desc")
```
### URL的相关操作

你可以使用@Path注解替换URL相对路径

```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);
```
如果你需要添加请求参数，你可以使用@Query
```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);
//如果传参是这样 groupList(2,"8080");
//等价于group/2/users?sort=8080
```
如果你需要一次性传递多个参数，你可以使用@QueryMap注解
```java
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);
```
### 如果是POST请求，可以指定Http请求的请求体，使用@Body注解

```java
@POST("users/new")
Call<User> createUser(@Body User user);
```
### 表单

@FormUrlEncoded 表示请求体是一个表单数据，表示发送form-encoded的数据。每个键值对需要使用@Filed来标注
```java
@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);
```
### 文件上传
@Multipart表示文件上传，表示发送form-encoded的数据。每个键值对需要使用@Part来标注

```java
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
```
使用案例
```java
public interface GetRequest_Interface {
		/**
		 *表明是一个表单格式的请求（Content-Type:application/x-www-form-urlencoded）
		 * <code>Field("username")</code> 表示将后面的 <code>String name</code> 中name的取值作为 username 的值
		 */
		@POST("/form")
		@FormUrlEncoded
		Call<ResponseBody> testFormUrlEncoded1(@Field("username") String name, @Field("age") int age);

		/**
		 * {@link Part} 后面支持三种类型，{@link RequestBody}、{@link okhttp3.MultipartBody.Part} 、任意类型
		 * 除 {@link okhttp3.MultipartBody.Part} 以外，其它类型都必须带上表单字段({@link okhttp3.MultipartBody.Part} 中已经包含了表单字段的信息)，
		 */
		@POST("/form")
		@Multipart
		Call<ResponseBody> testFileUpload1(@Part("name") RequestBody name, @Part("age") RequestBody age, @Part MultipartBody.Part file);

}

GetRequest_Interface service = retrofit.create(GetRequest_Interface.class);

// @FormUrlEncoded 
Call<ResponseBody> call1 = service.testFormUrlEncoded1("Carson", 24);

//  @Multipart
RequestBody name = RequestBody.create(textType, "Carson");
RequestBody age = RequestBody.create(textType, "24");
MultipartBody.Part filePart = MultipartBody.Part.createFormData("file", "test.txt", file);

Call<ResponseBody> call3 = service.testFileUpload1(name, age, filePart);
```
### 你可以使用@Headers构造请求头

```java
@Headers("Cache-Control: max-age=640000")
@GET("widget/list")
Call<List<Widget>> widgetList();
```

```java
@Headers({
    "Accept: application/vnd.github.v3.full+json",
    "User-Agent: Retrofit-Sample-App"
})
@GET("users/{username}")
Call<User> getUser(@Path("username") String username);
```
### 你可以使用@Header构造不固定的请求头

```java
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)

//这种方式等同于
@Headers("Authorization:authorization")
@GET("user")
Call<User> getUser()

// 区别在于使用场景和使用方式
// 1. 使用场景：@Header用于添加不固定的请求头，@Headers用于添加固定的请求头
// 2. 使用方式：@Header作用于方法的参数；@Headers作用于方法
```

## 四、Retrofit的配置

### 数据解析器
Retrofit默认返回的是ResponseBody数据，如果不做特殊处理则需要自己处理转换。如果返回数据为JSON格式，通过添加Gson解析器可直接把JSON格式数据转换为实体类。
|数据解析器|Gradle依赖
| :- | :- |
|Gson	|com.squareup.retrofit2:converter-gson:2.0.2
|Jackson|com.squareup.retrofit2:converter-jackson:2.0.2
|Simple XML|com.squareup.retrofit2:converter-simplexml:2.0.2
|Protobuf|com.squareup.retrofit2:converter-protobuf:2.0.2
|Moshi|com.squareup.retrofit2:converter-moshi:2.0.2
|Wire|com.squareup.retrofit2:converter-wire:2.0.2
|Scalars|com.squareup.retrofit2:converter-scalars:2.0.2

使用方式
```java
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://fanyi.youdao.com/") // 设置网络请求的Url地址
                .addConverterFactory(GsonConverterFactory.create()) // 设置数据解析器
                .build();
```

### 网络请求设配器
使用时如使用的是 Android 默认的 CallAdapter，则不需要添加网络请求适配器的依赖，否则则需要按照需求进行添加 Retrofit 提供的 CallAdapter
|网络请求设配器|Gradle依赖
| :- | :- |
|guava|com.squareup.retrofit2:adapter-guava:2.0.2
|Java8|com.squareup.retrofit2:adapter-java8:2.0.2
|rxjava|com.squareup.retrofit2:adapter-rxjava:2.0.2

使用方式
```java
Retrofit retrofit = new Retrofit.Builder()
                .baseUrl("http://fanyi.youdao.com/") // 设置网络请求的Url地址
                .addCallAdapterFactory(RxJavaCallAdapterFactory.create()) // 支持RxJava平台
                .build();
```

## Retrofit拓展
Retrofit原理解析：[https://blog.csdn.net/guiman/article/details/51480497](https://blog.csdn.net/guiman/article/details/51480497)
Github: [https://github.com/square/retrofit](https://github.com/square/retrofit)
