title: dx.jar is missing
date: 2015-07-03 00:00:00
tags: [Android,Bug调试]
categories: Android
description: "Android Studio Build 报错：dx.jar is missing"
---
# Bug代码：
Error:Execution failed for task ':app:transformClassesWithDexForDebug'. > com.android.build.api.transform.TransformException: java.lang.RuntimeException: java.lang.RuntimeException: com.android.ide.common.process.ProcessException: java.util.concurrent.ExecutionException: java.lang.IllegalStateException: dx.jar is missing
 报错的原因：
 ```该项目的buildTools太低了，把它改大一点就Ok了。Like this!```

 ![build gradle配置文件](http://img.blog.csdn.net/20170220195157052?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzMyOTkyODc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)