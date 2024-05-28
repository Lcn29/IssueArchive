
## 1 快速搭建一个可以通过 java -jar 启动的项目

不做任何配置的 Maven 项目, 打包成 jar 文件后, 无法通过 java -jar 启动, 会提示 **没有主清单属性**

### 解决方法一

在 Maven 项目中添加 spring-boot-maven-plugin 插件, 并配置 mainClass。

在 pom 文件中添加如下配置
```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.2.2.RELEASE</version>
            <configuration>
                <mainClass>main 方法所在类的包路径</mainClass>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

### 解决方法二

在 Maven 项目中添加 spring-boot-maven-plugin 插件, 并配置 mainClass。

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>2.4.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>main 方法所在类的包路径</mainClass>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

如果项目是 Spring 项目, 会报异常, 可以通过增加下面的配置解决

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>2.4.1</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>shade</goal>
                    </goals>
                    <configuration>
                        <transformers>
                            <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                <mainClass>main 方法所在类的包路径</mainClass>
                            </transformer>
                            <!-- 兼容 Spring 项目 -->
                            <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>META-INF/spring.handlers</resource>
                            </transformer>
                            <transformer
                                    implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                                <resource>META-INF/spring.schemas</resource>
                            </transformer>
                        </transformers>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```


## 2 项目中快速引入 log4j2 日志框架

步骤 1 : 在 pom 文件中添加 log4j2 依赖

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j2-impl</artifactId>
    <version>2.23.1</version>
</dependency>
```

步骤 2 : 在 resources 目录下添加 log4j2.xml 配置文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<Configuration>

    <properties>
        <Property name="LOG_DIRECTORY">logs</Property>
        <property name="PATTERN">[%d{yyyy-MM-dd HH:mm:ss.SSS}][%style{%t}{magenta}][%highlight{%5level}][%style{%40.40logger{39}}{cyan}][%style{%X{traceId}}{BLUE}] %m%n
        </property>
    </properties>

    <appenders>
        <Console name="CONSOLE" target="system_out">
            <PatternLayout pattern="${PATTERN}"/>
        </Console>
    </appenders>

    <loggers>

        <root level="info">
            <appenderref ref="CONSOLE"/>
        </root>
    </loggers>

</Configuration>
```