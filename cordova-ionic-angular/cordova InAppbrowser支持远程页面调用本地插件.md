# 1. 需求简介

​	我们需要利用cordova这个平台打造一个物联网平台级的app，其关系类似于微信平台app和微信小程序的关系，我们需要开发一款集路由器管理功能和集成各种智能硬件管理的小应用的app，微信支付宝平台实现都是自己在android或ios的里自己改造从底层撸，我们使用第三方平台cordova，减少开发成本，提高开发效率。虽然性能可能会不如微信这些平台，但目前来说这是最好的可行性方案。

# 2.Android平台实现 

前提：

Html

Css

JavaScript

Cordova

Java

Ionic（可选，有Ionic技术基础更好，因为我最终是在Ionic项目里更改源码）

Andriod（可选，有Android技术基础更好）

## 2.1通过InAppBrowser打开远程页面，支持调用本地插件

默认已经创建了一个ionic4项目并且已经安装了InAppBrowser插件

1.添加平台

如果是cordova 项目

```shell
cordova platform add android
```

Ionic项目

```shell
Ionic cordova platform add android
```

2.构建项目

如果是cordova 项目:

```shell
cordova build android
```

Ionic项目：

```
Ionic cordova build android
```

3修改InAppBrowser Android部分的源码

![](https://github.com/liukai90/liukai90.github.io/blob/master/cordova%2Bionic/img/chromeClient.png)



打开platforms/android/app/src/main/java/org/apache/cordova/inappbrowser/InAppChromeClient.java文件

修改部分：

```java
@Override
    public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
        // See if the prompt string uses the 'gap-iab' protocol. If so, the remainder should be the id of a callback to execute.
        if (defaultValue != null && defaultValue.startsWith("gap")) {
            if(defaultValue.startsWith("gap-iab://")) {
                PluginResult scriptResult;
                String scriptCallbackId = defaultValue.substring(10);
                if (scriptCallbackId.startsWith("ThemeableBrowser")) {
                    if(message == null || message.length() == 0) {
                        scriptResult = new PluginResult(PluginResult.Status.OK, new JSONArray());
                    } else {
                        try {
                            scriptResult = new PluginResult(PluginResult.Status.OK, new JSONArray(message));
                        } catch(JSONException e) {
                            scriptResult = new PluginResult(PluginResult.Status.JSON_EXCEPTION, e.getMessage());
                        }
                    }
                    this.webView.sendPluginResult(scriptResult, scriptCallbackId);
                    result.confirm("");
                    return true;
                }
            }
            else
            {
                //修改部分
                //处理 cordova API回调
		// ----------------start---------------
                CordovaWebViewEngine engine = webView.getEngine();
                if (engine != null) {
                    CordovaBridge cordovaBridge = engine.getCordovaBridge();
                    if (cordovaBridge != null) {
                        // Unlike the @JavascriptInterface bridge, this method is always called on the UI thread.
                        String handledRet = cordovaBridge.promptOnJsPrompt(url, message, defaultValue);
                        if (handledRet != null) {
                            result.confirm(handledRet);
                        } else {
                            final JsPromptResult final_result = result;
                            CordovaDialogsHelper dialogsHelper = new CordovaDialogsHelper(webView.getContext());
                            dialogsHelper.showPrompt(message, defaultValue, new CordovaDialogsHelper.Result() {
                                @Override
                                public void gotResult(boolean success, String value) {
                                    if (success) {
                                        result.confirm(value);
                                    } else {
                                        result.cancel();
                                    }
                                }
                            });
                        }
                    } else {
                        // Anything else with a gap: prefix should get this message
                        LOG.w(LOG_TAG, "InAppBrowser does not support Cordova API calls: " + url + " " + defaultValue);
                        result.cancel();
                    }
                    // Anything else with a gap: prefix should get this message
//                    LOG.w(LOG_TAG, "InAppBrowser does not support Cordova API calls: " + url + " " + defaultValue);
//                    result.cancel();
//                    return true;
                }
                return true;
            }
//------------------------------end----------------------------
        }
        return false;
    }

```

由于引入了一些新的类所以需要引包，我这里为了方面引入所有cordova的包

```java
import org.apache.cordova.*;
```



4.修改cordova android部分的源码 

![](https://github.com/liukai90/liukai90.github.io/blob/master/cordova%2Bionic/img/engine.png)



打开 platforms/android/CordovaLib/src/org/apache/cordova/CordovaWebViewEngine.java 文件

新增抽象方法

```java
CordovaBridge getCordovaBridge();
```

打开CordovaWebViewEngin.java接口的实现类SystemWebViewEngine.java 

![](https://github.com/liukai90/liukai90.github.io/blob/master/cordova%2Bionic/img/systemengine.png)

路径如下：**platforms/android/CordovaLib/src/org/apache/cordova/engine/SystemWebViewEngine.java**

实现接口方法

```java
@Override
public CordovaBridge getCordovaBridge(){
    return bridge;
}

```



同理发现没有引入包的需要引入。

这样就基本实现远程页面可以调用本地插件的，调用本地插件的前提是需要提前先安装所能用到的插件。

## 2.2 解决浏览器只能访问一次的bug

​	有上述代码后发现确实可以访问本地插件了，但会发现app卡到那个页面不能动，经测试有一个小bug,若通过inappbrowser跳转新的webview后可以正常调用cordova API,但是点击返回如果跳转前的网页没用刷新(即:不重新加载),那么此时的页面将不可以调用cordova API.  

​	这是因为cordova初始时在原生会随机生成一个整数传给网页,来作为原生与H5之间交互的secret.此值由UI线程编写，由JS线程读取。

解决方案：

CordovaBridge.java新增代码，路径platforms/android/CordovaLib/src/org/apache/cordova/CordovaBridge.java 

![](https://github.com/liukai90/liukai90.github.io/blob/master/cordova%2Bionic/img/bridge.png)

```java
@SuppressLint("TrulyRandom")
int generateBridgeSecret() {
    //新增
	//-----------start-----------
    if (expectedBridgeSecret >= 0)
        return expectedBridgeSecret;
    //----------end-----------
    SecureRandom randGen = new SecureRandom();
    expectedBridgeSecret = randGen.nextInt(Integer.MAX_VALUE);
    return expectedBridgeSecret;
}

```

## 2.3 解决插件无回调

这种方法远程页面的确可以访问本地插件，但是无法将插件执行的回调结果返回给远程页面。

解决方案：

找到InAppBrowser插件的js接口如下图：

![](https://github.com/liukai90/liukai90.github.io/blob/master/cordova%2Bionic/img/inapp.png)



打开platforms/android/platform_www/plugins/cordova-plugin-inappbrowser/www/inappbrowser.js文件

新增：

```javascript
module.exports = function (strUrl, strWindowName, strWindowFeatures, callbacks) {
        // Don't catch calls that write to existing frames (e.g. named iframes).
        if (window.frames && window.frames[strWindowName]) {
            var origOpenFunc = modulemapper.getOriginalSymbol(window, 'open');
            return origOpenFunc.apply(window, arguments);
        }

        strUrl = urlutil.makeAbsolute(strUrl);
        var iab = new InAppBrowser();

        callbacks = callbacks || {};
        for (var callbackName in callbacks) {
            iab.addEventListener(callbackName, callbacks[callbackName]);
        }

        var cb = function (eventname) {
            iab._eventHandler(eventname);
        };

        strWindowFeatures = strWindowFeatures || '';

        exec(cb, cb, 'InAppBrowser', 'open', [strUrl, strWindowName, strWindowFeatures]);
		//新增
        // 声明全局变量__globalBrowser，表示当前界面开启了InAppBrowser
	//-----------------start--------------
        window.__globalBrowser = iab;
        //-----------------end--------------
        return iab;
    };
})();

```

打开cordova.Js修改如图

![](https://github.com/liukai90/liukai90.github.io/blob/master/cordova%2Bionic/img/cordova.png)

打开platforms/android/platform_www/cordova.js文件 新增：

```javascript
callbackFromNative: function (callbackId, isSuccess, status, args, keepCallback) {
    try {
        var callback = cordova.callbacks[callbackId];
        if (callback) {
            if (isSuccess && status === cordova.callbackStatus.OK) {
                callback.success && callback.success.apply(null, args);
            } else if (!isSuccess) {
                callback.fail && callback.fail.apply(null, args);
            }
            /*
            else
                Note, this case is intentionally not caught.
                this can happen if isSuccess is true, but callbackStatus is NO_RESULT
                which is used to remove a callback from the list without calling the callbacks
                typically keepCallback is false in this case
            */
            // Clear callback if not expecting any more results
            if (!keepCallback) {
                delete cordova.callbacks[callbackId];
            }
        }
        //新增
//-------------------start------------------
        else {
            // __globalBrowser为表示当前界面开启了InAppBrowser
            if(window.__globalBrowser) {
                var message = 'cordova.callbackFromNative("'+callbackId+'",'+isSuccess+',' + status +',' +JSON.stringify(args) + ',' + keepCallback + ')';
                // 调用InAppBrowser插件里的js回传方法
                window.__globalBrowser.executeScript({code: message});
            }
        }
//-------------------end------------------

    } catch (err) {
        var msg = 'Error in ' + (isSuccess ? 'Success' : 'Error') + ' callbackId: ' + callbackId + ' : ' + err;
        console && console.log && console.log(msg);
        console && console.log && err.stack && console.log(err.stack);
        cordova.fireWindowEvent('cordovacallbackerror', { 'message': msg });
        throw err;
    }
},

```

这样就解决了插件回调结果的问题。 

## 2.4远程页面文件部署问题

从本质上来讲已经实现，但是还有一个问题就是将远程页面调用本地插件的接口放哪？

①将插件js接口和远程页面一起部署到服务器，远程页面本地调用js接口。

这种方式的问题：

1．每个开发者开发的小应用都需要维护这些插件接口。

2.部署到服务器占用存储空间。

3.app打开每个小应用都需要加载远程js文件，性能影响较大且耗费网络资源

4.远程插件接口文件和App本地插件版本可能会冲突。

 

②搭建一个cordova 插件接口CDN服务器，每个用户通过远程连接引入插件js接口。

1.       每个用户共用一套js插件接口无需开发者维护。

2.       App本地只需加载一次。

3.       远程插件接口文件和App本地插件版本可能会冲突。

 

③app本地向插件浏览器注入插件js接口

1.       无需浏览器远程加载js接口文件。

2.       无需开发者维护插件接口。

3.       无版本冲突问题，因为和app共用一套插件接口。

4.       需要我们修改InAppBrowser源码

5.       开发者开发程序调用插件接口无提示

 

通过对以上三种方案分析，1和2其实是一种只是2对静态资源进行了优化，3比较符合我们的需求。

所以决定对InAppBrowser插件进行改造。

打开文件platforms/android/app/src/main/java/org/apache/cordova/inappbrowser/InAppBrowser.java



![](https://github.com/liukai90/liukai90.github.io/blob/master/cordova%2Bionic/img/inject.png)

修改

1.定义我们的私有协议头

```typescript
//改造
private static final String NATIVE_JS_PREFIX = "https://native-js/";

```

如下图

![](https://github.com/liukai90/liukai90.github.io/blob/master/cordova%2Bionic/img/inject2.png)



1.通过私有协议注入cordova.Js文件

在页面加载完成后注入

```java
public void onPageFinished(WebView view, String url) {
    super.onPageFinished(view, url);

    // CB-10395 InAppBrowser's WebView not storing cookies reliable to local device storage
    if (android.os.Build.VERSION.SDK_INT >= android.os.Build.VERSION_CODES.LOLLIPOP) {
        CookieManager.getInstance().flush();
    } else {
        CookieSyncManager.getInstance().sync();
    }

    // https://issues.apache.org/jira/browse/CB-11248
    view.clearFocus();
    view.requestFocus();

    try {
        JSONObject obj = new JSONObject();
        obj.put("type", LOAD_STOP_EVENT);
        obj.put("url", url);

        sendUpdate(obj, true);
    } catch (JSONException ex) {
        LOG.d(LOG_TAG, "Should never happen");
    }
    //新增
//-------------------------satrt------------------------------------
    String jsWrapper = "(function(d) { var c = d.createElement('script'); c.src = %s; d.body.appendChild(c); })(document)";
    //在InAppBrowser WebView中注入一个对象(脚本或样式)。
    injectDeferredObject(NATIVE_JS_PREFIX + "cordova.js", jsWrapper);
//---------------------------end---------------------------------
}

```

虽然注入，但是我们的文件实际上在本地，那么我们还是需要从本地加载，那么就需要拦截私有协议

如下重写shouldInterceptRequest方法以下全部是新增代码

```java
@SuppressLint("NewApi")
@Override
public WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request) {
    Uri uri = request.getUrl();
    String url = uri.toString();
    return processInterceptRequest(view, url);
}

@Override
public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
    // SDK API < 21 时走这个方法
    return processInterceptRequest(view, url);
}
public WebResourceResponse processInterceptRequest(WebView view, String url) {
    // 对于注入的 nativeJs，从本地读取
    if (url.startsWith(NATIVE_JS_PREFIX) && url.endsWith(".js")) {
        String path = url.substring(NATIVE_JS_PREFIX.length());
        String assetPath = "www/" + path;
        try {
            //打开并返回本地js文件资源
            InputStream inputStream = webView.getContext().getAssets().open(assetPath);
            return new WebResourceResponse("application/javascript", "UTF-8", inputStream);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
    return null;
}

```

这样就构建了一个简单小程序容器。 



# 3.服务端如何调用

服务端页面就和平时的h5页面即可，调用则按照cordova官方的插件文档教程使用即可。但页面无需再引入cordova.js文件，也无需在项目中安装，直接使用即可。

Cordova官网：https://cordova.apache.org/docs/en/latest/

# 4.参考资料

[Cordova android平台开发四(支持在新webview中打开网页,并支持cordova其他插件)](https://www.jianshu.com/p/31b28b8661c5)

[iOS InAppBrowser 调用原生插件](https://www.jianshu.com/p/8bff29e15f47)

