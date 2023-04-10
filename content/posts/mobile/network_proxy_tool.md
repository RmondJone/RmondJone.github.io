---
title: "Android应用内网络抓包快捷工具开发"
date: 2023-04-07T09:55:49+08:00
tags: ["Android"]
categories: [ "移动端"]
draft: false
---
# 一、背景
在日常开发中，如果需要抓包，过程很繁琐，于是我就想着这一步能不能简化。主要遇到如下几个痛点：

- 每次设置WIFI代理**都得去**手机设置中心—无线网—设置WIFI代理
- 每次设置完WIFI代理不想用了，还得去手机设置中心**去清理WIFI代理**
- 设置完WIFI代理之后，**手机无法正常访问网站**，比如测试经常跑过来问我为什么蒲公英包下载不了，结果排查半天是无线网设置了代理

# 二、方案预研
一开始网上找了一些方案，主要找到了以下几种方案

- **ADB设置系统WIFI代理**
  这个方案**弊端很明显**，需要手机连着电脑，而且每次设置完代理之后，想要清除代理还得重启手机。

- **代码反射设置系统WIFI代理**
  这个方案也不行，因为貌似**Android5.0之后就没有用**，然后还得不停的跟着Android系统版本进行适配

# 三、方案定稿
综上所述，貌似没有一个完美的方案可以解决这个问题。难道这个问题真的无解了？我们只能默默的手动去老老实实的手动设置？
![](/images/network_proxy_1.webp)

功夫不负有心人，只要你想，没有不可能。观察我们项目的代码，我发现所有的网络接口请求都是使用OkHttp去请求，思路一下打开。是不是OkHttp有提供方法出来给我们设置代理呢，网上一搜，果然一堆。

# 四、实现效果
我在Dokit工具中加了一个小工具，我们就可以快乐的使用这个小工具快速的进行抓包了。而且不会设置手机自身的WIFI代理，不影响手机正常的访问网站。
![](/images/network_proxy_2.webp)

![](/images/network_proxy_3.webp)

# 五、核心代码

```kotlin
class NetWorkProxyKit : AbstractKit() {
    override val icon: Int
        get() = R.drawable.ic_acting
    override val name: Int
        get() = R.string.tools_proxy

    override fun onAppInit(context: Context?) {
    }

    @RequiresApi(Build.VERSION_CODES.LOLLIPOP)
    @SuppressLint("SetTextI18n")
    override fun onClickWithReturn(activity: Activity): Boolean {
        val view = LayoutInflater.from(activity).inflate(R.layout.layout_proxy_setting, null)
        val hostEdit = view.proxy_host
        val portEdit = view.proxy_port
        val proxyHistoryGroup = view.proxy_history_group
        val emptyView = view.proxy_history_empty
        //设置历史代理设置监听
        proxyHistoryGroup.setOnCheckedChangeListener { group, checkedId ->
            val radioButton = group.findViewById<RadioButton>(checkedId)
            val ip = radioButton.text.toString()
            hostEdit.setText(ip.split(":")[0])
            portEdit.setText(ip.split(":")[1])
        }
        //回显上次设置的代理
        val host = ACache.get(activity).getAsString("proxy_host")
        val port = ACache.get(activity).getAsString("proxy_port")
        if (!TextUtils.isEmpty(host) && !TextUtils.isEmpty(port)) {
            hostEdit.setText(host)
            portEdit.setText(port)
        }
        //回显历史设置
        val proxyList: JSONArray = ACache.get(activity).getAsJSONArray("proxy_list") ?: JSONArray()
        if (proxyList.length() > 0) {
            emptyView.visibility = View.GONE
            proxyHistoryGroup.visibility = View.VISIBLE
            var index = 0
            val keys = arrayListOf<String>()
            var selectId = -1
            //动态构建选中项
            for (i in (proxyList.length() - 1) downTo 0) {
                //只展示最近5条
                if (index == 5) break
                val proxy: JSONObject = proxyList[i] as JSONObject
                val proxyHost = proxy["proxy_host"]
                val proxyPort = proxy["proxy_port"]
                val ip = "${proxyHost}:${proxyPort}"
                if (!keys.contains(ip)) {
                    val radioButton = RadioButton(activity)
                    radioButton.text = ip
                    radioButton.id = ThreadLocalRandom.current().nextInt()
                    if (proxyHost.equals(host)) {
                        selectId = radioButton.id
                    }
                    proxyHistoryGroup.addView(radioButton)
                    keys.add(ip)
                    index++
                }
            }
            //设置选中项
            if (selectId != -1) {
                proxyHistoryGroup.check(selectId)
            }
        } else {
            emptyView.visibility = View.VISIBLE
            proxyHistoryGroup.visibility = View.GONE
        }
        val dialog = AlertDialog.Builder(activity)
                .setTitle("设置网络代理")
                .setView(view)
                .setNegativeButton("清空代理") { _, _ ->
                    ACache.get(activity).put("proxy_host", "")
                    ACache.get(activity).put("proxy_port", "")
                    ToastUtil.shortToast(activity, "1s后将退出应用，再次重启生效！")
                    Handler(Looper.getMainLooper()).postDelayed({
                        exitProcess(0)
                    }, 1500)
                }
                .setPositiveButton("设置代理") { _, _ ->
                    ACache.get(activity).put("proxy_host", hostEdit.text.toString())
                    ACache.get(activity).put("proxy_port", portEdit.text.toString())
                    ToastUtil.shortToast(activity, "1s后将退出应用，再次重启生效！")
                    saveProxyList(activity, hostEdit.text.toString(), portEdit.text.toString())
                    Handler(Looper.getMainLooper()).postDelayed({
                        exitProcess(0)
                    }, 1500)
                }.create()
        dialog.show()
        return super.onClickWithReturn(activity)
    }

    /**
     * 注释：保存历史代理设置
     * 时间：2021/12/21 0021 17:34
     */
    private fun saveProxyList(context: Context?, host: String, port: String) {
        val proxyList = ACache.get(context).getAsJSONArray("proxy_list") ?: JSONArray()
        val params = mapOf(Pair("proxy_host", host), Pair("proxy_port", port))
        proxyList.put(JSONObject(params))
        ACache.get(context).put("proxy_list", proxyList)
    }
}
```

```java
OkHttpClient.Builder mOkHttpClient = new OkHttpClient.Builder()
        //设置请求读写的超时时间
        .connectTimeout(15, TimeUnit.SECONDS)
        .writeTimeout(15, TimeUnit.SECONDS)
        .readTimeout(15, TimeUnit.SECONDS)
        .callTimeout(15, TimeUnit.SECONDS)
        .cache(cache)
        .addInterceptor(new HeaderInterceptor())
        .addInterceptor(interceptor);

//设置网络抓包代理
if (BuildConfig.DEBUG && !TextUtils.isEmpty(ACache.get(context).getAsString("proxy_host"))) {
  val host = ACache.get(context).getAsString("proxy_host")
  val port = ACache.get(context).getAsString("proxy_port")
  config.setProxy(Proxy(Proxy.Type.HTTP, InetSocketAddress(host, port.toInt())))
}

//网络代理
if (config != null && config.getProxy() != null) {
    mOkHttpClient.proxy(config.getProxy());
}
```
**H5代理设置**

WebView需要导入最新的WebKit内核完成这个事情，支持Android9以后的系统

```groovy
implementation 'androidx.webkit:webkit:1.4.0'
```
```java
String host = ACache.get(getActivity()).getAsString("proxy_host");
String port = ACache.get(getActivity()).getAsString("proxy_port");
if (WebViewFeature.isFeatureSupported(WebViewFeature.PROXY_OVERRIDE) && !TextUtils.isEmpty(host)) {
    ProxyConfig proxyConfig = new ProxyConfig.Builder()
        .addProxyRule(host + ":" + port)
        .addDirect().build();
    ProxyController.getInstance().setProxyOverride(proxyConfig, command -> {
        //do nothing
    }, () -> Log.e(TAG, "WebView代理改变"));
}
```

