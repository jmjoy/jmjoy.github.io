---
title: "UOS下使用Spring RestTemplate遇到的TLS问题"
date: 2021-11-26T15:43:47+08:00
categories: ["编程语言"]
tags: []
---

UOS下使用Spring RestTemplate，添加如下依赖：

```java
    @Bean
    public RestTemplate restTemplate(RestTemplateBuilder restTemplateBuilder) {
        return restTemplateBuilder
                .setConnectTimeout(Duration.ofMillis(5000L))
                .setReadTimeout(Duration.ofMillis(30000L))
                .build();
    }
```

在启动的时候会遇到如下报错：

> Bean instantiation via factory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.web.client.RestTemplate]: Factory method 'restTemplate' threw exception; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [org.springframework.http.client.OkHttp3ClientHttpRequestFactory]: Constructor threw exception; nested exception is java.lang.AssertionError: No System TLS

## 原因

究其原因，是因为UOS自带了国密证书，国密证书使用了自有的椭圆曲线，所以无法使用JDK自带的java.security解析证书。

比如运行如下命令：

```bash
sudo update-ca-certificates
```

会有如下报错：

> Updating certificates in /etc/ssl/certs...
> 0 added, 0 removed; done.
> Running hooks in /etc/ca-certificates/update.d...
> 
> Exception in thread "main" java.security.cert.CertificateParsingException: java.io.IOException: Unknown named curve: 1.2.156.10197.1.301
>         at sun.security.x509.X509CertInfo.<init>(X509CertInfo.java:169)
>         at sun.security.x509.X509CertImpl.parse(X509CertImpl.java:1804)
>         at sun.security.x509.X509CertImpl.<init>(X509CertImpl.java:195)
>         at sun.security.provider.X509Factory.engineGenerateCertificate(X509Factory.java:102)
>         at java.security.cert.CertificateFactory.generateCertificate(CertificateFactory.java:339)
>         at sun.security.provider.JavaKeyStore.engineLoad(JavaKeyStore.java:760)
>         at sun.security.provider.JavaKeyStore$JKS.engineLoad(JavaKeyStore.java:56)
>         at sun.security.provider.KeyStoreDelegator.engineLoad(KeyStoreDelegator.java:224)
>         at sun.security.provider.JavaKeyStore$DualFormatJKS.engineLoad(JavaKeyStore.java:70)
>         at java.security.KeyStore.load(KeyStore.java:1445)
>         at org.debian.security.KeyStoreHandler.load(KeyStoreHandler.java:66)
>         at org.debian.security.KeyStoreHandler.<init>(KeyStoreHandler.java:52)
>         at org.debian.security.UpdateCertificates.<init>(UpdateCertificates.java:65)
>         at org.debian.security.UpdateCertificates.main(UpdateCertificates.java:51)
> Caused by: java.io.IOException: Unknown named curve: 1.2.156.10197.1.301
>         at sun.security.ec.ECParameters.engineInit(ECParameters.java:143)
>         at java.security.AlgorithmParameters.init(AlgorithmParameters.java:293)
>         at sun.security.x509.AlgorithmId.decodeParams(AlgorithmId.java:132)
>         at sun.security.x509.AlgorithmId.<init>(AlgorithmId.java:114)
>         at sun.security.x509.AlgorithmId.parse(AlgorithmId.java:372)
>         at sun.security.x509.X509Key.parse(X509Key.java:168)
>         at sun.security.x509.CertificateX509Key.<init>(CertificateX509Key.java:75)
>         at sun.security.x509.X509CertInfo.parse(X509CertInfo.java:667)
>         at sun.security.x509.X509CertInfo.<init>(X509CertInfo.java:167)
>         ... 13 more
> E: /etc/ca-certificates/update.d/jks-keystore exited with code 1.
> done.

## 解决方案

### 方案一（推荐）

修改`$JAVA_HOME/jre/lib/security/java.security`，去掉SunEC，替换成BouncyCastleProvider：
```
#security.provider.3=sun.security.ec.SunEC
security.provider.3=org.bouncycastle.jce.provider.BouncyCastleProvider
```

### 方案二

参考 <https://blog.csdn.net/zhangji261/article/details/107723719>

```Java
    static {
       Security.removeProvider("SunEC");
   }
```

### 方案三

参考 <https://blog.csdn.net/SkyChaserYu/article/details/109157266>
参考 <http://people.eecs.berkeley.edu/~jonah/bc/org/bouncycastle/jce/provider/BouncyCastleProvider.html>


要在运行时添加提供程序，请使用：

```xml
        <dependency>
            <groupId>org.bouncycastle</groupId>
            <artifactId>bcprov-jdk15on</artifactId>
            <version>1.61</version>
        </dependency>
```

```Java
 import java.security.Security;
 import org.bouncycastle.jce.provider.BouncyCastleProvider;

 Security.addProvider(new BouncyCastleProvider());
```


## 2023年3月30日更新：

貌似最新版UOS不出现这个问题了。
