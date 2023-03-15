# Spring WebClient Experimental

# 项目位于[GitLab](https://gitlab.com/ximinghui/spring-webclient-experimental)

### 简介

Spring WebClient Experimental是一个参考Spring 5的WebClient设计的简陋、轻量实现，该WebClient尽量提供与Spring WebClient
一致的使用方式。

### 轻量级

该WebClient不依赖任何HTTP库（如JDK11 HttpClient、Apache HttpClient、OkHttp），它仅使用自JDK 1.1就存在的古老
HttpURLConnection类。WebClient实现类仅仅300行代码左右，其核心代码“retrieve()方法”只有三四十行。

### 依赖清单

- [required] [Apache Commons Lang3 - 3.12.0](https://central.sonatype.com/artifact/org.apache.commons/commons-lang3/3.12.0)
- [required] [Apache Commons IO - 2.11.0](https://central.sonatype.com/artifact/commons-io/commons-io/2.11.0)
- [required] [Apache Log4j API - 2.20.0](https://central.sonatype.com/artifact/org.apache.logging.log4j/log4j-api/2.20.0)
- [provided] [Jackson Databind - 2.14.2](https://central.sonatype.com/artifact/com.fasterxml.jackson.core/jackson-databind/2.14.2)
- [provided] [Project Lombok - 1.18.26](https://central.sonatype.com/artifact/org.projectlombok/lombok/1.18.26)
- [test] [JUnit Jupiter API - 5.9.2](https://central.sonatype.com/artifact/org.junit.jupiter/junit-jupiter-api/5.9.2)
- [test] [JUnit Jupiter Engine - 5.9.2](https://central.sonatype.com/artifact/org.junit.jupiter/junit-jupiter-engine/5.9.2)
- [test] [Apache Log4j Core - 2.20.0](https://central.sonatype.com/artifact/org.apache.logging.log4j/log4j-core/2.20.0)

### 适用场景

- 自己的项目或模块要作为公用工具库提供给其它项目使用，考虑到不同项目可能未采用Spring框架、Spring版本过低或也许会导致Spring依赖版
  本冲突从而无法添加“spring-webflux”依赖，但又需要一个方便的HttpClient，这时可采用本库作为一个简单的替代和过渡。

### 不适用场景

- **不适用于低于JDK8的环境（JDK8可用）**；
- **该库不支持HTTPS请求**；
- 仅将该库作为过渡期间的桥接，一旦有机会请使用Spring WebClient取代；
- 对XML格式的支持不够完善。

### 使用方式

#### a) 引入依赖

该库位于Maven中央仓库，使用阿里云镜像仓库的小伙伴可能会出现找不到该依赖报错（转发阿里云云效Maven通知）：
> 自2022.12.12起，受 Maven 中央仓库网络限制，阿里云云效 Maven 中央代理仓库可能会出现部分新增依赖查找不到的情况，但不影响已有依赖，请知悉

```xml
<dependency>
    <groupId>org.ximinghui</groupId>
    <artifactId>spring-webclient-experimental</artifactId>
    <version>1.0</version>
</dependency>
```

### b) 配置日志（optional）

在resources目录下建log4j2.xml文件，示范如下：

```xml
<?xml version="1.0"?>
<Configuration name="ConfigTest" status="WARN">
    <Appenders>
        <Console name="console" target="SYSTEM_OUT">
            <PatternLayout>
                <Pattern>%date{yyyy-MM-dd HH:mm:ss.sss} %5level: %msg%n</Pattern>
            </PatternLayout>
        </Console>
    </Appenders>
    <Loggers>
        <Root level="info">
            <AppenderRef ref="console" level="info"/>
        </Root>
    </Loggers>
</Configuration>
```

### c) 享受便利

以下代码演示了不同的示例，根据情况参考：

```java
// 声明WebClient.ResponseSpec
WebClient.ResponseSpec response;

// URI可以和BaseUrl分开设置
response = WebClient.create(baseUrl).get().uri("/schools/123").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// URI可以不以“/”开头，程序会自动判断并作出处理
response = WebClient.create(baseUrl).get().uri("schools/123").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// URI支持URI变量形式
response = WebClient.create(baseUrl).get().uri("/schools/{schoolId}", "123").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// URI支持多个URI变量形式
response = WebClient.create(baseUrl).get().uri("/schools/{schoolId}/classes/{classId}", "123", "16").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// URI支持URI变量形式（传递Map）
Map<String, String> params = new LinkedHashMap<>();
params.put("schoolId", "123");
params.put("classId", "16");
response = WebClient.create(baseUrl).get().uri("/schools/{schoolId}/classes/{classId}", params).retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// BaseUrl可以直接给完整地址
response = WebClient.create(baseUrl + "/schools/123").get().retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// 设置Content-Type请求头
response = WebClient.create(baseUrl).post().contentType(MediaType.APPLICATION_JSON).body("{\"name\": \"测试\"}").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// 设置Content-Type请求头（方式二）
response = WebClient.create(baseUrl).post().defaultHeader("Content-Type", "application/json").body("{\"name\": \"测试\"}").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// Content-Type可省略，支持自动检测（目前仅支持JSON格式）
response = WebClient.create(baseUrl).post().body("{\"name\": \"测试\"}").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// 设置Content-Type请求头（XML格式）
response = WebClient.create(baseUrl).post().contentType(MediaType.APPLICATION_XML).body("<name>测试</name>").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// 添加自定义标头
response = WebClient.create(baseUrl).post().defaultHeader(
        "Content-Type", "application/json",
        "Token", "abc",
        "x-username", "tom",
        "x-age", "55"
).body("{\"name\": \"测试\"}").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// 通过method方法设置请求方式
response = WebClient.create(baseUrl).method(HttpMethod.GET).uri("schools/123").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// 创建的时候不设置baseUrl，后面再设置
response = WebClient.create().baseUrl(baseUrl).get().uri("schools/123").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// 设置为30秒超时（默认是5秒）
response = WebClient.create(baseUrl).responseTimeout(1000 * 30).get().uri("schools/123").retrieve();
log.info("状态码：{} 响应体：{}", response.getStatus().value(), response.getBody());

// 将接口结果序列化为Java对象（目前仅支持JSON格式序列化）
Dog dog = WebClient.create("http://example.org/dogs/76").get().retrieve().bodyTo(Dog.class);
log.info("查询结果：{}", dog);

// 将接口结果序列化为Java对象（方式二）
dog = WebClient.create("http://example.org/dogs/76").get().retrieve().bodyTo(body -> Json.safeConvertTo(body, Dog.class));
log.info("查询结果：{}", dog);

// 将接口结果序列化为Java对象（XML格式可以手动转换）
dog = WebClient.create("http://example.org/dogs/76").get().retrieve().bodyTo(body -> convertToXML(body, Dog.class));
log.info("查询结果：{}", dog);

// introduction是一个创意性的小东西，用于给接口起一个简单说明的名字，日志打印时会优先显示而不是接口地址
dog = WebClient.create("http://example.org/dogs/76").get().introduction("查询狗信息").retrieve().bodyTo(body -> convertToXML(body, Dog.class));
log.info("查询结果：{}", dog);

// 所有示例均可不采用链式调用
WebClient myWebClient = WebClient.create(baseUrl);
myWebClient.uri("/students/{name}", "tom");
myWebClient.method(HttpMethod.GET);
myWebClient.responseTimeout(1000 * 6);
WebClient.ResponseSpec retrieve = myWebClient.retrieve();
log.info("查询结果：{}", retrieve.getBody());
```
