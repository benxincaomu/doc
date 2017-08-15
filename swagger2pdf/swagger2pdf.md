# swagger2pdf

## 转换步骤

### 编写api的swagger.yaml

### 将yaml转换为.doc格式

​	使用java进行转换,pom依赖如下

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.asiainfo</groupId>
    <artifactId>swagger2adoc</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>swagger2adoc</name>
    <url>http://maven.apache.org</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.github.swagger2markup</groupId>
            <artifactId>swagger2markup</artifactId>
            <version>1.3.1</version>
            <exclusions>
                <exclusion>
                    <groupId>org.pegdown</groupId>
                    <artifactId>pegdown</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.pegdown</groupId>
            <artifactId>pegdown</artifactId>
            <version>1.6.0</version>
        </dependency>
        <dependency>
            <groupId>org.asciidoctor</groupId>
            <artifactId>asciidoctorj</artifactId>
            <version>1.5.4.1</version>
        </dependency>
        <dependency>
            <groupId>org.asciidoctor</groupId>
            <artifactId>asciidoctorj-pdf</artifactId>
            <version>1.5.0-alpha.11</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <repositories>
        <repository>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
            <id>jcenter-releases</id>
            <name>jcenter</name>
            <url>http://jcenter.bintray.com</url>
        </repository>
    </repositories>
</project>

```

java转换代码

```java
public class Yaml2Adoc {
    public static void main(String[] args) {
        Path in= Paths.get("d:/swagger.yaml");
        Path outDir=Paths.get("d:/out/aa");
        Swagger2MarkupConverter.from(in).build().toFile(outDir);
    }
}
```

运行代码后，将会在`d:/out`目录下产生一个`aa.adoc`的文件

## 将adoc转换为pdf

转换过程需要使用`asciidoctor-pdf`，这是一个`ruby`库，所以需要先安装ruby，并安装gem包管理工具。可以根据自己的环境下载对应的安装包进行安装。

安装完成后，执行命令验证是否安装成功

```shell
ruby -version
gem -v
```

### 安装asciidoctor-pdf及其依赖库

安装依赖库

```shell
gem install prawn --version 1.3.0
gem install addressable --version 2.4.0
gem install prawn-svg --version 0.21.0
gem install prawn-templates --version 0.0.3
```

在中文版windows下，需要确认cmd的编码为`UTF-8`(`官方文档是这么写的，不过我的win10在未设置的情况下并没有出现乱码，根据情况自行设置`)

```shell
chcp 65001
```

安装asciidoctor-pdf及高亮库

```shell
gem install asciidoctor-pdf --pre
gem install rouge
gem install pygments.rb
gem install coderay
```

设置高亮，在文档开始加上`:source-highlighter: rouge`

检查安装结果

```shell
asciidoctor-pdf -v
```

环境搭建成功后就可以将adoc格式转换为pdf。

### 格式转换

简单格式转换

```shell
asciidoctor-pdf aa.adoc
```

但是`asciidoctor-pdf` 在默认情况下使用的是英文字体，有可能会存在中文乱码或显示不全的问题，可按一下步骤进行修复。

官方关于自定义字体的方案

```
https://github.com/asciidoctor/asciidoctor-pdf/blob/master/docs/theming-guide.adoc#custom-fonts
```

#### 修复中文显示不全

下载官方代码库到本地

```shell
git clone https://github.com/asciidoctor/asciidoctor-pdf.git
```

复制asciidoctor-pdf/data/fonts到adoc所在的目录，并复制一个你想使用的中文字体库到fonts目录下

复制`asciidoctor-pdf/data/themes/default-theme.yml`到adoc所在目录，并重命名为`theme.yml`

修改theme.yml,将其中的`mplus1p-regular-fallback.ttf`替换为你指定的中文字体文件名

然后执行下面的命令

```shell
asciidoctor-pdf -a pdf-style=theme.yml -a pdf-fontsdir=fonts aa.adoc
```



