# Spring Cloud Netflix Eureka

하나의 PC에서 여러 개의 서비스를 동시에 실행하기 위해선 포트를 분산시켜서 실행시켜야 한다.

| 서비스                | url                   |
|--------------------|-----------------------|
| SERVICE INSTANCE A | http://localhost:8080 |
| SERVICE INSTANCE B | http://localhost:8081 |
| SERVICE INSTANCE C | http://localhost:8082 |

만약 PC가 한 대 이상일 때는 IP가 다르기 때문에 포트는 동일하게 실행해도 된다.

| 서비스                | url                    |
|--------------------|------------------------|
| SERVICE INSTANCE A | http://my-server1:8080 |
| SERVICE INSTANCE B | http://my-server2:8080 |
| SERVICE INSTANCE C | http://my-server3:8080 |

- **Eureka**가 해주는 역할을 **Service Discovery**라고 한다.
- **Service** **Discovery**는 외부에서 다른 서비스들이 마이크로서비스들을 검색하기 위해서 사용되는 개념이다.

**동작 방식**

1. 각각의 마이크로서비스들의 위치 정보를 `Spring Cloud Netflix Eureka`에 등록을 해준다.
2. 등록된 마이크로서비스를 사용하고 싶은 Client는 자신이 필요한 요청정보를 `Load Balancer(API Gateway)`에 전달한다.
3. 요청 정보는 `Service Discovery`에 전달이 되서 필요한 정보에 맞는 마이크로서비스의 위치 정보를 반환한다.
4. 마이크로서비스의 위치 정보를 받은 `Load Balancer`는 해당 마이크로서비스에 요청 정보를 전달해 필요한 정보를 반환받는다.

## 프로젝트 생성

![](img/img.png)

![](img/img_1.png)

*pom.xml*

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.0.6</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.example</groupId>
    <artifactId>discoveryservice</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>discoveryservice</name>
    <description>discoveryservice</description>
    <properties>
        <java.version>17</java.version>
        <spring-cloud.version>2022.0.2</spring-cloud.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
    <repositories>
        <repository>
            <id>netflix-candidates</id>
            <name>Netflix Candidates</name>
            <url>https://artifactory-oss.prod.netflix.net/artifactory/maven-oss-candidates</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>

</project>
```

- pom.xml을 확인해보면 spring cloud의 dependency를 추가되어 있는 것을 확인할 수 있다.

*application.yml*

```yaml
#Eureka Server의 Port
server:
  port: 8761

#마이크로서비스 어플리케이션의 고유 이름
spring:
  application:
    name: discoveryservice

#Eureka 라이브러리를 포함한채 스프링 부트가 기동이 되면
#기본적으로 Eureka Client 입장으로 어딘가 등록하게 된다.
#필요없는 행위이기에 값을 false로 설정해줘야 한다.
eureka:
  client:
    register-with-eureka: false #default: true
    fetch-registry: false       #default: true
```

*DiscoveryserviceApplication.java*

```java
package com.example.discoveryservice;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class DiscoveryserviceApplication {

    public static void main(String[] args) {
        SpringApplication.run(DiscoveryserviceApplication.class, args);
    }

}
```

- `@EnableEurekaServer`어노테이션을 추가해준다.

*실행* [http://127.0.0.1:8761](http://127.0.0.1:8761/)

![](img/img_2.png)

유레카 서버를 실행중인게 확인이 된다.