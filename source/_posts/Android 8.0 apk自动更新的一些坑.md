title: Android 8.0 apk自动更新的一些坑
date: 2017-09-06 00:00:00
tags: [Android]
categories: bug
description: 自动更新下载对应apk后，跳转系统安装界面闪退，以及安装时提示软件包出错
---

## 跳转系统安装界面闪退

  这个是8.0独有的坑，在AndroidManifast中加上
  
```java
<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
```

即可解决跳转安装界面闪退bug。

## 安装时提示软件包出错

  在用Intent install apk，时加上install.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);  

```java
 		 Intent install = new Intent(Intent.ACTION_VIEW);
         install.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
         install.setDataAndType(uri, "application/vnd.android.package-archive");
         install.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
         context.startActivity(install);
```  

