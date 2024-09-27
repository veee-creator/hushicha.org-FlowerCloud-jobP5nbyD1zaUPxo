
# Maven


Maven 是一个以项目为中心的自动化构建工具，主要用于Java项目的管理和构建。它提供了一种统一的方式来描述项目的结构、依赖关系和构建过程，简化了项目的构建和管理。


### Maven 的主要特点：


1. **项目对象模型（POM）**：Maven 使用`pom.xml`文件来定义项目的依赖、插件和构建配置。POM 是 Maven 项目的核心，描述了项目的基本信息。
2. **依赖管理**：Maven 允许开发者轻松地管理项目所需的库和框架。通过声明依赖，Maven 会自动下载所需的库及其依赖项，解决版本冲突。
3. **插件体系**：Maven 提供了丰富的插件，可以在构建过程中执行各种任务，例如编译代码、打包、运行测试等。
4. **生命周期管理**：Maven 通过定义项目的生命周期来规范构建过程，主要包含清理、编译、测试、打包、部署等阶段。
5. **多模块项目支持**：Maven 支持多模块项目，允许将相关模块组织在同一个项目结构中，方便管理和构建。
6. **社区支持**：Maven 拥有庞大的社区支持，提供了大量的插件和资源，帮助开发者高效开展工作。


## 1\. 常见的依赖冲突报错


**1\.1、版本冲突报错**：



```
[WARNING] Found version conflict(s) in library dependencies; some are suspected to be binary incompatible:
[WARNING] 
[WARNING]   org.apache.commons:commons-lang3:jar:3.4 is referenced from more than one dependency.
[WARNING]     - org.apache.commons:commons-lang3:jar:3.4 (compile)
[WARNING]     - org.apache.commons:commons-lang3:jar:3.5 (compile)

```

**1\.2、类找不到或方法找不到**：


**NoClassDefFoundError** 或者 **ClassNotFoundException**或者 **NoSuchMethodException**



```
常见但不仅限于以下异常:
java.lang.NoClassDefFoundError: org/apache/commons/lang3/StringUtils
或者    
java.lang.NoSuchMethodError: org.apache.commons.lang3.StringUtils.isBlank(Ljava/lang/CharSequence;
 
NoSuchMethodException
依赖冲突可能会间接导致 NoSuchMethodException。例如：
    
不同版本的库：如果项目中引入了同一库的不同版本，Maven 可能会选择一个版本，而这个版本中可能缺少某些方法，从而在运行时导致 NoSuchMethodException。
依赖传递：某些依赖可能会引入其他库的特定版本，如果这些版本之间存在不兼容的方法，也可能导致 NoSuchMethodException。    

```

**1\.3、依赖树中的冲突**：



```
例如：
[INFO] +- org.springframework.boot:spring-boot-starter-web:jar:2.3.4.RELEASE:compile
[INFO] |  +- org.springframework.boot:spring-boot-starter:jar:2.3.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot:jar:2.3.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-autoconfigure:jar:2.3.4.RELEASE:compile
[INFO] |  |  +- org.springframework.boot:spring-boot-starter-logging:jar:2.3.4.RELEASE:compile
[INFO] |  |  |  +- ch.qos.logback:logback-classic:jar:1.2.3:compile
[INFO] |  |  |  |  +- ch.qos.logback:logback-core:jar:1.2.3:compile
[INFO] |  |  |  |  \- org.slf4j:slf4j-api:jar:1.7.30:compile
[INFO] |  |  |  +- org.apache.logging.log4j:log4j-to-slf4j:jar:2.13.3:compile
[INFO] |  |  |  |  \- org.apache.logging.log4j:log4j-api:jar:2.13.3:compile
[INFO] |  |  |  \- org.slf4j:jul-to-slf4j:jar:1.7.30:compile
[INFO] |  |  +- jakarta.annotation:jakarta.annotation-api:jar:1.3.5:compile
[INFO] |  |  +- org.springframework:spring-core:jar:5.2.9.RELEASE:compile
[INFO] |  |  |  \- org.springframework:spring-jcl:jar:5.2.9.RELEASE:compile

```

## 2、排查maven 依赖是否冲突


**2\.1、pom 依赖，这里展示部分依赖进行拆解**


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927083246075-872669293.png)


**2\.2、使用idea 自带工具进行排查分析：红色的线就表示冲突了。**


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927083400893-470136644.png)


**2\.3、使用工具 ：maven helper**


首先我使用的idea工具，可以安装插件maven helper，


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927083433634-1238566326.png)


**2\.4、重启idea**
这玩意装好，我们关闭窗口，有可能会叫你restart一下，你就乖乖听话。之后我们打开pom文件并且点击依赖分析。切换到：Dependency Analyzer


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927084000733-972844483.png)


## 3、分析冲突


**3\.1、点击右键\-\-\-》Jump to Source 就会跳回到自己的pom 文件**（我这跳转到 56 行）


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927083507021-879994601.png)


这个时候我们可以一直向下点击，去看 **mybatis 3\.5\.14 依赖路径和 mybatis 3\.5\.15 冲突依赖路径版本**


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927084920721-1836267454.png)


## 4、解决办法


总共有四种解决方式：


### **1，第一声明优先原则**


在pom.xml配置文件中，如果有两个名称相同版本不同的依赖声明，那么先写的会生效（同个pom.xml文件）。


所以，先声明自己要用的版本的jar包即可。


### **2，路径近者优先**


直接依赖优先于传递依赖，如果传递依赖的jar包版本冲突了，那么可以自己声明一个指定版本的依赖jar，即可解决冲突。


### **3，排除原则**


传递依赖冲突时，可以在不需要的jar的传递依赖中声明排除，从而解决冲突。


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927083615132-808813928.png)


在pom 文件就会自动生成 排除标识



```
<dependency>
            <groupId>com.baomidougroupId>
            <artifactId>mybatis-plus-spring-boot3-starterartifactId>
            <version>3.5.5version>
            
            <exclusions>
                <exclusion>
                    <artifactId>mybatisartifactId>
                    <groupId>org.mybatisgroupId>
                exclusion>
            exclusions>
        dependency>

```

结果：


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927083643529-1396184536.png)


刷新：先返回去pom 点击刷新，然后在 maven help 里面点击 Refresh UI


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927083719747-2133536791.png)


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927083735304-638827359.png)


最后依赖冲突只剩三个


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927083809863-110967782.png)


### **4，版本锁定原则**


在配置文件pom.xml中先声明要使用哪个版本的相应jar包，声明后其他版本的jar包一律不依赖。解决了依赖冲突。



```
<dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.mybatisgroupId>
                <artifactId>mybatisartifactId>
                <version>3.5.14version>
            dependency>
        dependencies>
    dependencyManagement>

```

结果：依旧可以排除掉依赖冲突


![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927083858360-913011855.png)


最后文章有啥不对，欢迎大佬指点！！！
如果感觉对你有帮助就**点赞推荐**或者**关注**一下吧！！！
![](https://img2024.cnblogs.com/blog/2719585/202409/2719585-20240927091023464-1188976011.gif)


 本博客参考[豆荚加速器](https://yirou.org)。转载请注明出处！
