Maven 是一个项目管理和整合工具，为开发者提供了一套完整的构建生命周期框架。

- Maven 使用了一个标准的目录结构和一个默认的构建生命周期
- 无缝衔接了编译、发布、文档生成、团队合作和其他任务

### 1.概述

#### 1.1 POM

> **工程对象模型**：Project Object Model

- 是使用 Maven 的基本组件，位于工程根目录下，是一个名为 pom的 xml 文件
- POM 包含了关于工程和各种配置细节的信息，Maven 使用这些信息构建工程
- POM 也包含了目标和插件

在创建 POM 之前，我们首先确定工程组（groupId），及其名称（artifactId）和版本，这是工程的唯一标识。

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
   http://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>

   <!-- 这是工程组的标识。它在一个组织或者项目中通常是唯一的 -->
   <groupId>com.companyname.project-group</groupId>
   <!-- 这是工程的标识。它通常是工程的名称 -->
   <artifactId>project</artifactId>
   <version>1.0</version>

</project>
```

**Super POM**

所有的 POM 都继承自一个父 POM（无论是否显式定义了这个父 POM）。它包含了一些可以被继承的默认设置。

查看 Super POM 默认配置的一个简单方法是执行以下命令：**mvn help:effective-pom**

#### 1.2 构建生命周期

> 构建生命周期是一组阶段的序列（sequence of phases），每个阶段定义了目标被执行的顺序。

一个典型的 Maven 构建生命周期是由以下几个阶段的序列组成的：

| 阶段              | 处理     | 描述                                           |
| ----------------- | -------- | ---------------------------------------------- |
| prepare-resources | 拷贝资源 | 可以自定义需要拷贝的资源                       |
| compile           | 编译     | 完成源代码编译                                 |
| package           | 打包     | 根据 pom.xml 中描述的打包配置创建 JAR / WAR 包 |
| install           | 安装     | 本地 / 远程仓库中安装工程包                    |

#### 1.3 构建配置文件

构建配置文件是一组配置的集合，用来设置或者覆盖 Maven 构建的默认配置。使用构建配置文件，可以为不同的环境定制构建过程，如 Producation 和 Development 环境。

**Profile 类型**

- Per Project：定义在工程 POM 文件 pom.xml 中
- Per User：定义在 Maven 设置 xml 文件中 （%USER_HOME%/.m2/settings.xml）
- Global：定义在 Maven 全局配置 xml 文件中 （%M2_HOME%/conf/settings.xml）

**Profile 激活**

- 显式使用命令控制台输入：**mvn test -Ptest**

```xml
<profiles>
    <profile>
        <id>test</id>
        <build>
        </build>
    </profile>
</profiles>
```

- 通过 maven 设置
- 基于环境变量（用户 / 系统变量）
- 操作系统配置（例如，Windows family）
- 现存 / 缺失 文件

#### 1.4 仓库

> 仓库是一个位置，可以存储所有的工程 jar 文件、library jar 文件、插件或任何其他的工程指定的文件。

**本地仓库（local）**

Maven 本地仓库是本地机器上的一个文件夹，它在你第一次运行任何 maven 命令的时候创建。

```xml
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0
   http://maven.apache.org/xsd/settings-1.0.0.xsd">
      <localRepository>C:/MyLocalRepository</localRepository>
</settings>
```

**中央仓库（central）**

Maven 中央仓库是由 Maven 社区提供的仓库，其中包含了大量常用的库。

- 这个仓库由 Maven 社区管理。
- 不需要配置。
- 需要通过网络才能访问。

**远程仓库（remote）**

它是开发人员自己定制仓库，包含了所需要的代码库或者其他工程中用到的 jar 文件。

**Maven 依赖搜索顺序**

本地仓库——>中央仓库——>远程仓库——>停滞处理并抛出error

#### 1.5 插件

> Maven 实际上是一个依赖插件执行的框架，每个任务实际上是由插件完成
>
> 插件可以创建 jar 文件，创建 war 文件，编译代码文件，代码单元测试，创建工程文档，创建工程报告

**插件类型**

- **Build plugins**：在构建时执行，并在 pom.xml 的 元素中配置
- **Reporting plugins**：在网站生成过程中执行，并在 pom.xml 的 元素中配置

### 2. 工程

#### 2.1 创建工程

Maven 使用**原型（archetype）**插件创建工程。要创建一个简单的 Java 应用，我们将使用 maven-archetype-quickstart 插件。

1. 从控制台进入工程根目录，键入命令：

```
mvn archetype:generate
```

2. Maven 将开始处理，并将创建完成的 java 应用工程结构

#### 2.2 构建 & 测试工程

- `mvn clean package`
- `mvn clean compile`

#### 2.3 外部依赖

- 在 src 文件夹下添加 lib 文件夹

- 复制任何 jar 文件到 lib 文件夹下

- 同时将依赖库添加到POM中

  ```xml
  <dependency>
      <groupId>ldapjdk</groupId>
      <artifactId>ldapjdk</artifactId>
      <scope>system</scope>
      <version>1.0</version>
      <systemPath>${basedir}\src\lib\ldapjdk.jar</systemPath>
  </dependency>
  ```

**依赖阶段**

- *compile*
  默认scope为compile，表示为当前依赖参与项目的编译、测试和运行阶段，属于强依赖。打包之时，会达到包里去。
- *test*
  该依赖仅仅参与测试相关的内容，包括测试用例的编译和执行，比如定性的Junit。
- *runtime*
  依赖仅参与运行周期中的使用。一般这种类库都是接口与实现相分离的类库，比如JDBC类库，在编译之时仅依赖相关的接口，在具体的运行之时，才需要具体的mysql、oracle等等数据的驱动程序。
  此类的驱动都是为runtime的类库。
- *provided*
  该依赖在打包过程中，不需要打进去，这个由运行的环境来提供，比如tomcat或者基础类库等等，事实上，该依赖可以参与编译、测试和运行等周期，与compile等同。区别在于打包阶段进行了exclude操作。
- *system*
  使用上与provided相同，不同之处在于该依赖不从maven仓库中提取，而是从本地文件系统中提取，其会参照systemPath的属性进行提取依赖。
- *import*
  这个是maven2.0.9版本后出的属性，import只能在dependencyManagement的中使用，能解决maven单继承问题，import依赖关系实际上并不参与限制依赖关系的传递性。

#### 2.4 工程文档

> Maven 使用称作 Doxia 的文件处理引擎创建文档，它将多个源格式的文件转换为一个共通的文档模型。

- 根目录下命令：**mvn site**
- 生成文档在 target\site 文件夹下

#### 2.5工程模板

Maven 使用原型（Archetype）概念为用户提供了大量不同类型的工程模版（614 个），Maven 模板帮助用户快速创建 java 项目。

### 3. 管理

#### 3.1 快照

> **快照**是一个特殊的版本，它表示当前开发的一个副本。与常规版本不同，Maven 为每一次构建从远程仓库中检出一份新的快照版本

**快照 vs 版本**

- 对于版本，Maven 一旦下载了指定的版本（例如 data-service:1.0），它将不会尝试从仓库里再次下载一个新的 1.0 版本。想要下载新的代码，数据服务版本需要被升级到 1.1。
- 对于快照，每次用户接口团队构建他们的项目时，Maven 将自动获取最新的快照（data-service:1.0-SNAPSHOT）。

使用命令强制下载最新快照：**mvn clean package -U**

#### 3.2 管理依赖

> Maven 核心特点之一是依赖管理。一旦我们开始处理多模块工程（包含数百个子模块或者子工程）的时候，模块间的依赖关系就变得非常复杂，Maven 提供了一种高度控制的方法。

**传递依赖发现**

| 功能     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| 依赖调节 | 决定当多个手动创建的版本同时出现时，哪个依赖版本将会被使用。 如果两个依赖版本在依赖树里的深度是一样的时候，第一个被声明的依赖将会被使用 |
| 依赖管理 | 直接的指定手动创建的某个版本被使用。如工程 C 在自己的以来管理模块包含工程 B，即 B 依赖于 A， 那么 A 即可指定在 B 被引用时所使用的版本 |
| 依赖范围 | 包含在构建过程每个阶段的依赖                                 |
| 依赖排除 | 任何可传递的依赖都可以通过 "exclusion" 元素被排除在外。举例说明，A 依赖 B， B 依赖 C，因此 A 可以标记 C 为 “被排除的” |
| 依赖可选 | 任何可传递的依赖可以被标记为可选的，通过使用 "optional" 元素。例如：A 依赖 B， B 依赖 C。因此，B 可以标记 C 为可选的， 这样 A 就可以不再使用 C |

**依赖范围**

| 范围     | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| 编译阶段 | 该范围表明相关依赖是只在工程的类路径下有效。默认取值         |
| 供应阶段 | 该范围表明相关依赖是由运行时的 JDK 或者 网络服务器提供的     |
| 运行阶段 | 该范围表明相关依赖在编译阶段不是必须的，但是在执行阶段是必须的 |
| 测试阶段 | 该范围表明相关依赖只在测试编译阶段和执行阶段                 |
| 系统阶段 | 该范围表明你需要提供一个系统路径                             |
| 导入阶段 | 该范围只在依赖是一个 pom 里定义的依赖时使用。同时，当前工程的POM 文件的 部分定义的依赖关系可以取代某特定的 POM |

**依赖管理**

通常情况下，在一个共通的工程下，有一系列的工程。在这种情况下，我们可以创建一个公共依赖的 pom 文件，该 pom 包含所有的公共的依赖关系，我们称其为其他子工程 pom 的 pom 父。

#### 3.3 自动化部署

- Maven 构建和发布项目，
- SubVersion, 源代码库用以管理源代码，
- 远程仓库管理工具 (Jfrog/Nexus) 用以管理工程的二进制文件。

我们将会使用 Maven 发布的插件来创建一个自动化发布过程：

**Maven Release 插件**

- mvn release:clean：清理工作空间，保证最新的发布进程成功进行
- mvn release:rollback：回滚修改的工作空间代码和配置保证发布过程成功进行
- mvn release:prepare
- mvn release:perform：将代码切换到之前做标记的地方

#### 3.4 Web 应用

- maven-archetype-webapp 插件

### 4. 常用插件

#### 4.1 maven-compiler-plugin

设置maven编译的Jdk版本，用于编译源码。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.1</version>
    <configuration>
        <source>1.8</source> <!-- 源代码使用的JDK版本 -->
        <target>1.8</target> <!-- 需要生成的目标class文件的编译版本 -->
        <encoding>UTF-8</encoding><!-- 字符集编码 -->
        <skipTests>true</skipTests><!-- 跳过测试 -->
    </configuration>
</plugin>
```

#### 4.2 maven-jar-plugin

打包jar文件时，配置manifest文件，加入lib包的jar依赖。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <configuration>
        <classesDirectory>target/classes/</classesDirectory>
        <archive>
            <manifest>
                <mainClass>com.alibaba.dubbo.container.Main</mainClass>
                <!-- 打包时 MANIFEST.MF文件不记录的时间戳版本 -->
                <!--自动加载META-INF/spring目录下的所有Spring配置-->
                <useUniqueVersions>false</useUniqueVersions>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
            </manifest> <manifestEntries>
            <Class-Path>.</Class-Path>
            </manifestEntries>
        </archive>
    </configuration>
</plugin>
```

#### 4.3 maven-war-plugin

排除不想打进war包的jar 的配置。

```xml
<plugin> 
    <groupId>org.apache.maven.plugins</groupId> 
    <artifactId>maven-war-plugin</artifactId> 
    <version>2.1-alpha-1</version> 
    <configuration> 
        <!-- 
            打包之前过滤掉不想要被打进 .war包的jar,注意：这个地方，本来路径应该是 
            WEB-INF/lib/anaalyzer-2.0.4.jar,但是经过多次试验,不能这样，
            至于咋回事儿，搞不清楚。。经多方查证均无结果 
            暂且这样吧，虽然显得很丑陋，但是总能解决问题吧 
        --> 
        <warSourceExcludes>*/lib/analyzer-2.0.4.jar</warSourceExcludes> 
        <webResources> 
            <resource> 
                <!-- 元配置文件的目录，相对于pom.xml文件的路径 --> 
                <directory>src/main/webapp/WEB-INF</directory> 
                <!-- 是否过滤文件，也就是是否启动auto-config的功能 --> 
                <filtering>true</filtering> 
                <!-- 目标路径 --> 
                <targetPath>WEB-INF</targetPath> 
            </resource> 
        </webResources> 
    </configuration> 
</plugin> 
```

#### 4.4 maven-surefire-plugin

打包时候跳过测试。

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <skip>true</skip>
        <includes>
            <include>**/*Test*.java</include>
        </includes>
    </configuration>
</plugin>

```

#### 4.5 maven-source-plugin

```xml
<!-- 生成sources源码包的插件 -->
<plugin>
    <artifactId>maven-source-plugin</artifactId>
    <version>2.4</version>
    <configuration>
        <attach>true</attach>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>jar-no-fork</goal>
            </goals>
        </execution>
    </executions>
</plugin>

```

#### 4.6 maven-javadoc-plugin

```xml
<!-- 生成javadoc文档包的插件 -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-javadoc-plugin</artifactId>
    <version>2.10.2</version>
    <configuration>
        <aggregate>true</aggregate>
    </configuration>
    <executions>
        <execution>
            <id>attach-javadocs</id>
            <goals>
                <goal>jar</goal>
            </goals>
        </execution>
    </executions>
</plugin>

```

#### 4.7 maven-dependency-plugin

将Maven以来的Jar包复制到一个lib文件夹下，供jar包启动时候实用。

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <type>jar</type>
                <includeTypes>jar</includeTypes>
                <outputDirectory>
                    ${project.build.directory}/lib
                </outputDirectory>
            </configuration>
        </execution>
    </executions>
</plugin>

```

#### 4.8 多个环境

```xml
<properties>
    <env>dev</env>
</properties>
<profiles>
    <profile>
        <id>dev</id>
        <properties>
            <env>dev</env>
        </properties>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>
    <profile>
        <id>test</id>
        <properties>
            <env>test</env>
        </properties>
    </profile>
    <profile>
        <id>production</id>
        <properties>
            <env>production</env>
        </properties>
    </profile>
</profiles>

<build>
    <finalName>war_plugin</finalName>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
        </resource>
    </resources>
    <filters>
        <filter>src/main/filter/${env}/db.properties</filter>
    </filters>
</build>

```

filter根据不同的profile加载相应的properties文件，因此可以在不同的profile下加载不同的配置文件。

默认用dev的配置打包，如果想用production的配置`package -P production`即可加载正式环境的配置。
