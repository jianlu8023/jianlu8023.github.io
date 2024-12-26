+++
title = 'java语言发送https请求'
date = '2024-07-08T13:41:37+08:00'
author = 'jianlu'
draft = true
description = "java语言发送https请求"
aliases = ["https"]
+++

# HTTPS 不开启证书验证发送请求


## okhttp3 操作步骤

* 通用SSLUtils

```java
public class SSLUtils {
    public static SSLContext createSSLContext() {
        try {
            SSLContext sslContext = SSLContext.getInstance("SSL");
            sslContext.init(null, new TrustManager[]{new X509TrustManager() {
                @Override
                public void checkClientTrusted(X509Certificate[] x509Certificates, String s) {
                }

                @Override
                public void checkServerTrusted(X509Certificate[] x509Certificates, String s) {
                }

                @Override
                public X509Certificate[] getAcceptedIssuers() {
                    return new X509Certificate[0];
                }
            }}, new SecureRandom());
            return sslContext;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static TrustManager[] getTrustAllCerts() {
        return new TrustManager[]{new X509TrustManager() {
            @Override
            public void checkClientTrusted(X509Certificate[] x509Certificates, String s) {
            }

            @Override
            public void checkServerTrusted(X509Certificate[] x509Certificates, String s) {
            }

            @Override
            public X509Certificate[] getAcceptedIssuers() {
                return new X509Certificate[]{};
            }
        }};
    }
}
```

```java

public class OkHttpUtil {
    public static OkHttpClient getUnsafeOkHttpClient() {
        try {
            final TrustManager[] trustAllCerts = new TrustManager[]{
                    new X509TrustManager() {
                        @Override
                        public void checkClientTrusted(
                                java.security.cert.X509Certificate[] chain, String authType) {
                        }

                        @Override
                        public void checkServerTrusted(
                                java.security.cert.X509Certificate[] chain, String authType) {
                        }

                        @Override
                        public java.security.cert.X509Certificate[] getAcceptedIssuers() {
                            return new java.security.cert.X509Certificate[]{};
                        }
                    }
            };
            final SSLContext sslContext = SSLContext.getInstance("SSL");
            sslContext.init(null, trustAllCerts, new java.security.SecureRandom());
            final javax.net.ssl.SSLSocketFactory sslSocketFactory = sslContext.getSocketFactory();
            OkHttpClient.Builder builder = new OkHttpClient.Builder();
            // 下一行 这一行JDK9以上适用，JDK8把第二个参数去掉
            builder.sslSocketFactory(sslSocketFactory, (X509TrustManager) trustAllCerts[0]);
            //            builder.sslSocketFactory(sslSocketFactory);
            builder.hostnameVerifier((hostname, session) -> true);
            return builder.connectTimeout(6, TimeUnit.SECONDS).build();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        OkHttpClient client = getUnsafeOkHttpClient();
        // 等效于
        // OkHttpClient build = new OkHttpClient.Builder().sslSocketFactory(SSLUtils.createSSLContext().getSocketFactory(), (X509TrustManager) SSLUtils.getTrustAllCerts()[0]).hostnameVerifier((hostname, session) -> true).build();
        
        String url = "https://example.com:8090/api/login";
        String param2 = "this is a param";
        String path = "/opt/files/demo.txt";
        File param1 = new File(path);
        RequestBody requestBody = new MultipartBody.Builder().setType(MultipartBody.FORM)
                .addFormDataPart("param1", param1.getName(),
                        RequestBody.create(param1, MediaType.parse("application/octet-stream")))
                .addFormDataPart("param2", param2)
                .build();
        Request request = new Request.Builder().url(url).post(requestBody).build();
        try (Response response = client.newCall(request).execute()) {
            if (response.isSuccessful()) {
                return response.headers().get("Set-Cookie");
            } else {
                return null;
            }
        }
    }
}
```

## hutool 操作步骤

```java
public class Example {
    public static void main(String[] args) {
        HttpResponse execute = HttpRequest.post("https://example.com:8090/api/login")
                .header("Content-Type", "application/json;charset=utf-8")
                .setSSLSocketFactory(SSLUtils.createSSLContext().getSocketFactory())
                .execute();
        System.out.println(execute.body());
    }
}
```