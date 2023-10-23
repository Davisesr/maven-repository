# GitHub作为maven仓库

## 使用JAR包

1、pom.xml

```xml
<!--1、使用-->    
<dependencies>
  <dependency>
    <groupId>cn.dujy.mybatis</groupId>
    <artifactId>spring-boot-mybatis-plus-starter</artifactId>
    <version>1.0.2</version>
  </dependency>
</dependencies>

<repositories>
  <repository>
    <id>your-repository-id</id>    <!--此id需要与setting中的配置-->
    <!--
      1. https://raw.github.com => 固定写法。若编译的时候一直连接不上，可将https改成http并反复重试
      2. davisesr => github登录的用户名
      3. maven-repository => 存储发布的jar仓库
      4. main => 从哪个分支中拿jar包
	-->
    <url>http://raw.github.com/davisesr/maven-repository/main</url>
    <snapshots>
      <enabled>true</enabled>
      <updatePolicy>always</updatePolicy>
    </snapshots>
  </repository>
</repositories>
```

2、配置 maven 的 `setting.xml`

```xml
<!-- 配置1：
  注意: <mirrorOf>*,!your-github</mirrorOf>
  原本: <mirrorOf>*</mirrorOf> 
-->
<mirror><!--此处使用阿里云镜像，增加!your-repository-id-->
  <id>nexus-aliyun</id>
  <mirrorOf>*,!your-repository-id</mirrorOf>
  <name>Nexus aliyun</name>
  <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```





