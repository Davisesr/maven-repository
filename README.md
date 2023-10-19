# GitHub作为maven仓库

## 一、准备工作：

1. 在github上创建仓库，用于存储jar。例如：`maven-repository` （[仓库地址](https://github.com/Davisesr/maven-repository.git)）
2. 在github上创建一个token （[地址链接](https://github.com/settings/tokens)），因为我们要将jar上传到仓库，避免不了权限问题，但是直接使用账号密码不安全
3. 配置maven的setting.xml

```xml
<servers>
  <server>
    <id>your-github</id><!-- 自定义的id。记住此id, 后续发布jar包时要使用 -->
    <password>your-token</password><!-- 上面提到的token -->
  </server>
</servers>
```

## 二、生成JAR包

1、配置项目中的pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <!--项目信息-->
    <groupId>cn.dujy.mybatis</groupId><!-- groupId，推荐：前缀(公司).组名(你自己).项目相关（必填） -->
    <artifactId>spring-boot-mybatis-plus-starter</artifactId><!-- 一般是项目名（必填） -->
    <version>1.0.2</version><!-- 版本（必填） -->
    <name>spring-boot-mybatis-plus-starter</name><!-- 一般是项目名（非必填）  -->
    <description>基于 mybatis-plus 的二次开发</description><!-- 详情（非必填） -->

    <!--springboot的父工程：管理相当多jar的不冲突的版本-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.5</version>
    </parent>

    <!--统一版本-->
    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <!--具体依赖-->
    <dependencies>
          <!--项目中需要的依赖-->
    </dependencies>

    <!--打包配置-->
    <build>
        <!-- 用于打标准的maven包，并上传至GitHub -->
        <plugins>

            <!--用于编译Java源代码。该插件允许你在Maven项目中定义和配置Java编译的行为，包括源代码版本、目标字节码版本等-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>3.0.2</version>
            </plugin>

            <!--用于编译Java源代码。该插件允许你在Maven项目中定义和配置Java编译的行为，包括源代码版本、目标字节码版本等-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.5.1</version>
                <configuration>
                    <target>1.8</target>
                    <source>1.8</source>
                    <excludes>
                        <exclude>
                            cn/dujy/mybatis/SpringbootDomeApplication.java
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>

            <!--用于生成并管理项目的源代码（通常是Java源代码）和附带的源代码JAR文件-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-source-plugin</artifactId>
                <version>3.0.1</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>jar</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

            <!--用于将构建产物（通常是JAR文件、WAR文件等）部署到指定Maven仓库-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-deploy-plugin</artifactId>
                <version>2.8.2</version>
                <configuration>
                    <!-- 配置本地打包后的本地仓库存储地址，后续上传jar包会从此仓库中去取 -->
                    <altDeploymentRepository>
                        internal.repo::default::file://${project.build.directory}/maven-repository
                    </altDeploymentRepository>
                </configuration>
            </plugin>

            <!--用于发布到指定仓库-->
            <plugin>
                <groupId>com.github.github</groupId>
                <artifactId>site-maven-plugin</artifactId>
                <version>0.12</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>site</goal>
                        </goals>
                        <phase>deploy</phase>
                        <configuration>
                            <!--服务名（settings.xml中配置内容，上述配置的server的id）-->
                            <server>your-github</server>
                            <!-- 提交消息（commit message）-->
                            <message>Maven artifacts for 【${project.artifactId}-${project.version}】</message>
                            <!-- 上传网站的位置（远程仓库名） -->
                            <repositoryName>maven-repository</repositoryName>
                            <!--组织或用户名-->
                            <repositoryOwner>Davisesr</repositoryOwner>
                            <!-- 使用合并或覆盖内容 -->
                            <merge>true</merge>
                            <!--分支（固定写法：refs/heads/ + 你的分组）-->
                            <branch>refs/heads/main</branch>
                            <!--源文件地址-->
                            <outputDirectory>${project.build.directory}/maven-repository</outputDirectory>
                            <!-- 包含outputDirectory标签内填的文件夹中的所有内容 -->
                            <includes>
                                <include>**/*</include>
                            </includes>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```



2、终端执行：

```shell
mvn clean deploy
```

> 打包结束后，会根据配置的内容，自动上传到github仓库中

## 三、使用JAR包

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





