# app&小程序渗透

## APP渗透

使用反编译软件对apk进行反编译，反编译软件JEB、APKTool、AndroidKiller等

### 备份数据泄露风险

**描述：**
被测应用的AndroidManifest.xml文件中allowBackup属性值被设置true，可通过adb backup对应用数据进行备份，在无root的情况下可以导出应

**测试方法：**

``` 
7:<application android:allowBackup="true"
存储的所有数据，造成用户数据泄露在AndroidManifest.xml文件中查看allowBackup属性，检查是否未设置或者被设置为true
```

导出软件包的方法

1.使用adb shell查看安装的三方包名
`adb.exe shell pm list packages -3`

2.使用adb backup进行数据备份

``` 
adb.exe backup -nosystem -f D:\sjmnq\page\inj.ab packageName

adb.exe backup -nosystem -f D:\sjmnq\page\inj.ab b3nac.injuredandroid
```

3.进行解压

``` 
java15 -jar abe.jar unpack D:\sjmnq\page\inj.ab D:\sjmnq\page\inj.tar
```

### 信息篡改泄露风险

**描述：**AndroidManifest.xml文件中Debuggable属性值被设置为true，可以设置断点来控制程序的执行流程，在应用程序运行时修改其行为

**测试方法：**
1.使用反编译软件对apk进行反编译，反编译软件JEB、APKTool、AndroidKiller等
2、在AndroidManifest.xml中搜索Debuggable属性，检查是否被设置为true

### 数据泄漏

**描述：**Content Provider是安卓应用组件，以表格的形式把数据展现给外部的应用。每个Content Provider都对应一个以“content://”开头的特定URI，任何应用都可以通过这个URI操作Content Provider 应用的数据库。如果应用对权限控制不当就会造成信息泄露

**测试方法：**
1.使用drozer获取所有可以访问的URI

```
run scanner.provider.finduris -a packageName
```

2.在drozer控制台获取各个URI的数据

#### drozer简介

drozer是一款针对android的安全测试框架，最主要的功能是测试安卓四大组件Content Provider、Activity、Service、Broadcast Receiver

### 登录页面等被绕过

**描述：**Activity是安卓应用组件，提供与用户进行交互的界面，如果应用对权限控制不当，可以绕过登录界面直接显示该界面
**检测方法：**
1.使用drozer检查APK中是否存在暴露的activity
2.使用drozer进行activity之间切换

### 非法权限提升

**描述：**Service是Android中四大组件进行后台作业的主要组件，如果被测应用对权限控制不当，导致其他应用可以启动被测应用的Service

**检测方法：**
1.run app.service.info -a b3nac.injuredandroid或者使用AndroidKiller进行反编译，查看AndroidManifest.xml中service配置的android:exported= "true"

### 拒绝服务、非法越权

**描述：**Broadcast Receiver是Android中四大组件用于处理广播事件的组件，若存在配置不当则其他应用可以伪装发送广播从而可造成信息泄露，拒绝服务攻击

**检测方法：**
1.run app.broadcast.info -a b3nac.injuredandroid
2.使用AndroidKiller反编译查看源码action

### 反编译代码

**描述：**代码保护不足，常会带来以下危害：被盗版 、敏感信息泄露

**检测方法：**

使用反编译软件AndroidKiller对apk进行反编译，在smali代码处右键->查看->查看源码，调用jd-gui工具反编译为java代码，查看源码是否被混淆，查看源码是否包括了显而易见的敏感信息等

### 重新编译打包

**描述：**破解者通过反编译后得到程序源代码，修改后重新编译、签名并安装。在重新打包的过程中，破解者可能注入恶意代码，或者修改软件逻辑绕过鉴权等

**检测方法：**使用AndroidKiller反编译，并修改代码

点击菜单栏Android->编译->Android Killer->直接点击编译

WebView漏洞

**描述：**在webView下有一个非常特殊的接口函数addJavascriptInterface，能实现本地java和js的交互。被测应用中存在WebView漏洞，没有对注册JAVA类的方法调用进行限制，导致攻击者利用addJavascriptInterface这个接口函数穿透webkit控制android本机

**检测方法：**

使用AndroidKiller反编译，在代码中搜索addJavascriptInterface函数

**webview介绍：**Android WebView在Android平台上是一个特殊的view，它能用来显示网页，这个WebView类可以被用来在app中仅仅显示一张在线的网页，还可以用来开发浏览器

webview的addJavascriptInterface函数映射流程：

- 在安卓代码中定义一个AndroidObj类，其中包含一个test()方法

- 使用webview.addJavascriptInterface(new AndroidObj(), “jsTest”)进行映射

- 在前端js中定义一个函数callAndroid()，在该函数中直接调用jsTest.test()即可成功调用安卓后端的对象和其所有的方法

**历史漏洞：**WebView任意代码执行漏洞

漏洞原理：JS和Android直接的通信可以通过addJavascriptInterface接口进行对象映射当JS拿到Android这个对象后，就可以调用这个Android对象中所有的方法，包括系统类（java.lang.Runtime 类），从而进行任意代码执行具体

### 不安全的本地存储

**描述：**安卓开发者使用多种方法将数据存储在安卓应用中，而存储在本地的数据文件如果未加密，易造成敏感信息泄漏如用户名，密码，手势密码，敏感个人信息等

**检测方法：**

1、Shared Preferences是用key-value来存储私有的原始数据的xml文件。数据类型有布尔型，浮点型，整形，长整型和字符串。通常情况下存储的路径为：`/data/data/<packagename>/shared_prefs/<filename.xml>`，直接打开查看是否有敏感数据

2、SQLite数据库是轻量级基于文件的数据库。这些文件通常以db或者sqlite结尾。安卓默认提供了大量SQLite支持。应用的数据库一般存储在下面的地方：`/data/data/<packagename>/databases/<databasename.db>`，可以使用SQLite数据库直接打开查看是否有敏感数据

3、检查SD卡目录中是否存在敏感数据

### 边信道信息泄漏

**描述：**当APP处理用户或其它数据源输入的数据时，可能会把数据放在不安全的位置，容易导致边信道被攻击者利用造成信息泄露

## 抓包工具

### Fiddler

### 存在的风险

后台服务器地址泄露

**描述：**在使用BurpSuite等工具对应用进行监听的过程中，发现后台服务器地址。对后台服务器进行测试，若后台服务器存在漏洞，则可控制后台服务器

未使用有效的token机制，导致可以绕过鉴权

**描述：**如果被测应用没有使用有效的token机制，对登陆响应中的服务器返回的鉴权信息进行修改，即可绕过服务器鉴权，直接访问系统内部信息

传输数据可修改，造成越权访问

**描述：**利用已有的用户名密码登录应用，当应用访问某一模块时，使用

Fiddler等工具进行监听，对访问该模块时的关键信息进行替换，则可越

权访问他人的应用模块

登录设计缺陷，存在被暴力破解风险

**描述：**用户登录过程中，未对同一用户的登录失败次数做限制，导致存

在被暴力破解的风险

利用业务逻辑缺陷制作短信炸弹

**描述：**如果在用户注册过程中存在逻辑设计缺陷，可对指定手机号码随

意发送短信，造成短信炸弹攻击，可能造成用户投诉或恶意软件传播等

## 小程序渗透

1、抓取数据包后，即可对小程序进行渗透测试，测试项参考app渗透

后台服务器地址泄露

未使用有效的token机制，导致可以绕过鉴权

传输数据可修改，造成越权访问

登录设计缺陷，存在被暴力破解风险

利用业务逻辑缺陷制作短信炸弹

其他web漏洞

2、如果出现设置代理后小程序打不开情况，可以使用Xposed框架和JustTrustMe解决

3、对于小程序客户端反编译，进行信息收集和审计，可以参考信息收集篇