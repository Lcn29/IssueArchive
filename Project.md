
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

在 Maven 项目中添加 maven-shade-plugin 插件, 并配置 mainClass。

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

## 3 SpringBoot 项目打包后, 仍然可以读取到 resources 目录下的文件

```java
ClassLoader classLoader = this.getClass().getClassLoader();
// 读取到 resources 目录下的文件 picture/1.jpg
InputStream originFileInputStream = classLoader.getResourceAsStream("/picture/1.jpg");

// 需要临时保存下来, 可以转为 File

// 创建一个临时目录来存储文件
Path tempDir = Files.createTempDirectory("tempFiles");
// 确定文件的最终路径
Path filePath = tempDir.resolve("test.jpg");
FileCopyUtils.copy(originFileInputStream, Files.newOutputStream(filePath));
// 保存为文件
File file = filePath.toFile();
```

## 4 首字符排序

借助
```xml
<dependency>
    <groupId>com.belerweb</groupId>
    <artifactId>pinyin4j</artifactId>
    <version>2.5.1</version>
</dependency>
```

```java
public static class A {

    public A(String first, String secondName) {
        this.firstName = first;
        this.secondName = secondName;
    }
}

public static int compareByPinyin(Main.A compareStr1, Main.A compareStr2) {
    // 先按照 first name 排序
    String str1 = getPinyinInitials(compareStr1.getFirstName()).toLowerCase();
    String str2 = getPinyinInitials(compareStr2.getFirstName()).toLowerCase();
    int compareStrResult =  str1.compareTo(str2);
    if (compareStrResult != 0) {
        return compareStrResult;
    }

    // 如果 first name 相同, 再按照 second name 排序
    String str3 = getPinyinInitials(compareStr1.getSecondName()).toLowerCase();
    String str4 = getPinyinInitials(compareStr2.getSecondName()).toLowerCase();
    return str3.compareTo(str4);
}

public static String getPinyinInitials(String text) {
    StringBuilder sb = new StringBuilder();
    for (char c : text.toCharArray()) {
        String[] pinyinArray = PinyinHelper.toHanyuPinyinStringArray(c);
        if (pinyinArray != null && pinyinArray.length > 0) {
            sb.append(pinyinArray[0]);
        } else {
            sb.append(c);
        }
    }
    return sb.toString();
}
```

直接使用官方内嵌的

问题: Collator 在 Java 中, 中文使用的是 UNICODE 字符集, 大概收录了 6763 个汉字, 如果对应的汉字不在收录内, 会异常

```java
// 按照首字符排序
List<A> buildNameOrderList = upgradeDevicePageVoList.stream()
    .sorted(Comparator.comparing(A::getFirstName, Collator.getInstance(Locale.CHINA))
        .thenComparing(A::getSecondName, Collator.getInstance(Locale.CHINA)))
    .collect(Collectors.toList());
```