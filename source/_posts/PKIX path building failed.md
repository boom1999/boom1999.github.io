---
title: PKIX path building failed
date: 2024-07-11
tags: 
    - SSL
    - HTTPS
categories: Summary
copyright: true
---

> :pushpin:开发中遇到 `javax.net.ssl.SSLHandshakeException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target`，猜测应该是SSL证书的问题，找了一些解决方式，在这里做总结。

<!--more-->

## :books: 1. 前言

调用HTTPS接口时遇到的SSL证书相关问题，这里是`PKIX path building failed`。

## :satellite: 2. 出现的背景

> 捕获抛出的`IOException`，检查发现是`HttpURLConnection.getResponseCode();`出现了问题，当然在其他连接处也可能产生，具体表现为`sun.security.validator.ValidatorException: PKIX path building failed`，也就是说无法找到请求目标的有效认证路径导致的。

- JDK缺少某个目标Server的安全证书

## :mag_right: 3. 解决方案

> JDK在`lib/security/`目录下存放着`cacerts`文件，这个文件是JDK的证书库，产生上述错误的原因就是未能找到目标服务器的SSL证书，需要做的就是找到证书或者禁用证书验证，下面提供几种解决方案。

### 3.1 将目标服务器的证书导入到Java的信任库

- 获取目标服务器的SSL证书，可以通过浏览器地址栏左侧的图标进行下载（crt或者pem文件）（*不会的话可以自行Google*），也可以通过命令行导出。

    `openssl s_client -connect <your-server>:443 -showcerts`

- 导入证书到Java的信任库(记得先备份原始的信任库`cacerts`)

    `keytool -import -alias test_https_certificate -file D:\***.crt -keystore D:\java**\lib\security\cacerts`

  - 密钥库口令默认为`changeit`

- 也可以把`-keystore`后的路径改为自己新建的信任库路径，然后在SpringBoot的启动类中修改`javax.net.ssl.trustStore`和`javax.net.ssl.trustStorePassword`的值。

    ``` java
    System.setProperty("javax.net.ssl.trustStore", "D:\\yourCacerts");
    System.setProperty("javax.net.ssl.trustStorePassword", "changeit");
    ```

### 3.2 使用自定义的信任管理器

``` java
import javax.net.ssl.*;
import java.security.cert.X509Certificate;

public class CustomTrustManager {
    public static void main(String[] args) throws Exception {
        TrustManager[] trustAllCerts = new TrustManager[]{
            new X509TrustManager() {
                public X509Certificate[] getAcceptedIssuers() { return null; }
                public void checkClientTrusted(X509Certificate[] certs, String authType) { }
                public void checkServerTrusted(X509Certificate[] certs, String authType) { }
            }
        };

        SSLContext sc = SSLContext.getInstance("SSL");
        sc.init(null, trustAllCerts, new java.security.SecureRandom());
        HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());

        // Your code to open connection and interact with the server
    }
}
```

### 3.3 禁用证书验证

- 建议仅在测试环境下使用

    `HttpsURLConnection.setDefaultHostnameVerifier((hostname, session) -> true);`

## 4. 常见问题

- 可以使用 `keytool -list -keystore D:\java**\lib\security\cacerts` 命令来查看已导入的证书
