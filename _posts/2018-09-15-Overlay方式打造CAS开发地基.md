---
layout: post
title: Overlay 方式构建 CAS 
categories: []
tags: [CODE,CAS]
comments: true
---

>继 [上一篇](https://frank-cq.github.io/2018/06/09/%E8%AE%A4%E8%AF%81%E6%8E%88%E6%9D%83%E9%82%A3%E7%82%B9%E4%BA%8B%E5%84%BF/) 介绍了 OAuth2.0 协议，这次打算写一个系列，分享一些实践中的内容。



[Apereo CAS](https://apereo.github.io/cas/5.2.x/index.html) 是企业级的开源单点登录解决方案，单点登录洋文叫 [SSO](https://en.wikipedia.org/wiki/Single_sign-on)，也就是实现在有多个系统的情况下，只要登录其中一个系统，访问其他系统时就无需再次登录。


CAS 项目托管在全球最大的同性交友网站 [Github](https://github.com/apereo/cas) 上，源代码采用模块化的方式组织，通过配置文件来进行插拔。CAS 项目原生支持协议为  [CAS](https://apereo.github.io/cas/5.2.x/protocol/CAS-Protocol.html)，同时也支持多种其他业界协议，如 [SAML](https://apereo.github.io/cas/5.2.x/protocol/SAML-Protocol.html)、[WS-Federation](https://apereo.github.io/cas/5.2.x/protocol/WS-Federation-Protocol.html)、[OAuth2](https://apereo.github.io/cas/5.2.x/protocol/OAuth-Protocol.html)、OpenID、OpenID Connect、[REST](https://apereo.github.io/cas/5.2.x/protocol/REST-Protocol.html)，更多内容这里就不展开了。


>第一次接触这个项目，要如何开始工作呢 ？

官方推荐大家采用 Overlay 的方式来构建项目，可以说非常方便。

>什么是 Overlay ？

看 maven overlay 的 [官方文档](https://maven.apache.org/plugins/maven-war-plugin/overlays.html)（要夸一下，组织得相当赏心悦目），简单地说就是用来组合多个 WAR 项目的，就 CAS 项目来说，就是你会引入 CAS 项目组已经编译好的某一个可运行的二进制基础版本，然后在此基础上或者通过添加配置插入自己需要的模块，或者重写一些实现方法来定制个性化的功能，而基础版本中的代码会被覆盖，从而项目会以你希望的方式来执行。

没听明白？别着急，下面会教大家如何手撕 CAS。


首先我们到同性交友网站上把 overlay 模板拿下来，这里我用的 [cas-overlay-template-5.2.x](https://github.com/apereo/cas-overlay-template/tree/5.2) 版本。注意该版本要求 jdk 1.8+。

![5.2.6](https://upload-images.jianshu.io/upload_images/716099-5e330ccc1c22ca18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


选好版本后，clone 或者 download 都行。

打开拿到的 overlay-template 模板，结构如下：

![overlay 模板的结构](https://upload-images.jianshu.io/upload_images/716099-6d7cb3a45d8dc759.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最关键的就是倒数第二的 pom.xml 文件，大家搞 maven 应该都熟悉了。用 IntelliJ 或 eclipse，导入整个 maven 项目后会自动去拿依赖。因为官方提供的基础版本是一个已经编译好可直接执行的程序，所以用 IDE 构建完毕就得到了可部署的 war 包。

部署 war 包用的 tomcat，因为 CAS 项目要求用 https 连接（http 也行，只是会提示不安全），所以需要先生成 https 的加密密匙，用 jdk 自带的 keytool 就行。

```bash
# 生成证书，-dname 按需配置
# 这里把 keystore 和 加密密匙的访问密码都设为了 changeit，按需修改
keytool -genkey -alias cas -keyalg RSA -keysize 2048 -keypass changeit -storepass changeit -keystore D:/CODE/Keystore/<名字>.keystore -dname "CN=cas.example.org,OU=example.com,O=cas,L=Shenzhen,ST=Shenzhen,C=CN"
```

同时需要修改 tomcat 的配置文件 server.xml，把加密密匙的相关配置写进去。
```
    <!-- 可用 http 访问 8080 端口 -->
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               redirectPort="8443" />
    <!-- 用 https 访问 8443 端口 -->
    <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="200" scheme="https"
               secure="true" SSLEnabled="true"
               keystoreFile="D:/CODE/Keystore/<名字>.keystore"
               keystorePass="changeit"
               clientAuth="false" sslProtocol="TLS"/>
```

配置完毕把构建的 war 包扔到 tomcat  的 webapp 目录下，如果 tomcat 部署在本地，启起来后，访问 https//:localhost:8443/cas/login 就能看到登录页面，官方的基础版本中配置了一个用户 casuser/Mellon ，可以用此登录。

>如何进行个性化功能修改？

上面说过，CAS 提供两种方式：一种是引入可插拔的模块，另一种是用自己重写的 java 方法来覆盖基础版本中的 class 实现。

现在需要我们基于官方基础版本的目录结构来调整本地项目的目录结构了。先看一下官方 overlay 基础版本的目录结构：

![官方基础版本生成的 war 包结构](https://upload-images.jianshu.io/upload_images/716099-b8bc19a91546e7fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

application.properties 文件中存放了所有配置属性，而 org/apereo/cas 是官方基础版本的代码路径，自己基于官方重新实现的方法代码必须放到对应的路径下才会覆盖基础版本中自带的 class 文件。因此我们的本地项目做如下调整：

![添加存放自定义文件的目录](https://upload-images.jianshu.io/upload_images/716099-3cd1d938d1fb13da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在 IDE 构建的过程中会引入一个 overlay 文件夹，里面放置从 maven 仓库取下来的官方构建好的基础版本。上图中没有显示。

在调整后的目录结构中，我们引入了第一个配置文件 application.properties 。引入 application.properties 文件是为了调整配置参数，这种配置文件不会自动覆盖基础版本中的同名配置文件，需要在 pom.xml 中手动声明，该声明也就是告知在构建时排除掉官方基础中带的，用自己修改过的。注意排除的路径为 overlay 基础版本中的路径。
![覆盖一些配置文件](https://upload-images.jianshu.io/upload_images/716099-abb9efb27baf186f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


配置文件需要手动声明覆盖，重写代码则只需要路径一致就会自动覆盖。

pom.xml 文件中还有两个地方也简单说明一下，cas.version 设置使用的 cas-overlay 版本。在 profiles 中的 dependencies 里添加依赖，也是在此处引入新模块，当然也可以不在 profiles 中写依赖，直接在和 profiles 平级的 dependencies 里写依赖。

![image.png](https://upload-images.jianshu.io/upload_images/716099-b1bfdf0fc855bbf1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


本篇就讲到这里，欲知后事如何，且听下回分解。

