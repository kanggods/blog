---
title: springboot集成dubbo以nacos为注册中心
date: 2022-09-14 18:00:05
categories: RPC
tags: [Dubbo,Nacos]
---
### 1、依赖导入

注意：3.0以后的dubbo版本。nacos的客户端版本要2.0以上，不然会报错。

```xml
<dependency>
    <groupId>org.apache.dubbo</groupId>
    <artifactId>dubbo-spring-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba.nacos</groupId>
    <artifactId>nacos-client</artifactId>
    <version>2.0.4</version>
</dependency>

<dependency>
    <groupId>com.pua</groupId>
    <artifactId>pua-common-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.alibaba.nacos</groupId>
            <artifactId>nacos-client</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### 2、配置文件

没啥好说的。nacos作为配置注册中心，dubbo，设置nacos为注册中心，设置接入协议。

```yaml
spring:
  cloud:
    nacos:
      host: 47.98.236.213:8848
      namespace: pua
      discovery:
        server-addr: ${spring.cloud.nacos.host}
        namespace: ${spring.cloud.nacos.namespace}
        group: ${spring.profiles.active}
      config:
        server-addr: ${spring.cloud.nacos.host}
        namespace: ${spring.cloud.nacos.namespace}
        group: ${spring.profiles.active}
        file-extension: yaml
        refresh-enabled: true
      username: nacos
      password: ENC(F0qwKRQUWxXgXfNDN05yIg==)
dubbo:
  protocol:
    name: dubbo
    port: -1
  application:
    name: pua-producer
  registry:
    register-mode: interface
    address: nacos://47.98.236.213:8848
    parameters:
      namespace: ${spring.cloud.nacos.namespace}
    username: nacos
    password: ENC(F0qwKRQUWxXgXfNDN05yIg==)
```

### 3、服务提供者使用

在启动类设置Dubbo启动扫描的。

```java
@EnableEncryptableProperties
@SpringBootApplication
@EnableDubbo(scanBasePackages = "com.pua.producer.service")
public class ProducerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ProducerApplication.class,args);
    }
}

@Service
@DubboService
public class ProducerServiceImpl implements ProducerService {
    @Override
    public String getId() {
        return "110";
    }
}

```

### 4、消费者使用

直接引入和生产者一样的依赖还需要引入消费者模块。

```
<dependency>
    <groupId>com.pua</groupId>
    <artifactId>producer</artifactId>
    <version>1.0</version>
    <scope>compile</scope>
</dependency>
```

```java
@DubboReference(interfaceClass = ProducerService.class)
private ProducerService producerService;

@ApiOperation("测试")
@GetMapping("/index")
public BaseResult<String> test(){
    String id = producerService.getId();
    return BaseResult.success(id);
}
```

### 5、结果

![image-20220929134726957](http://114.55.238.234:9000/images/image-20220929134726957.png)
