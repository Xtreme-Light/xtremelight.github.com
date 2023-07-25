---
title: 基于dependency-management插件制作自己的bom
date: 2023-07-19 21:00:00+0800
categories: [IT,gradle]
tags: [gradle,bom,dependency-management]
---

## BOM

BOM (The Bill of Materials)，实际上是maven中的一个概念。使用BOM可以定义一系列依赖及版本，其他项目可以在使用BOM时，无需定义版本号，即可获取版本号，是一种统一版本依赖管理的工具。当然传递的版本号是可以被复写的。



Gradle从5.0开始就添加了BOM支持了，但是在Gradle6.0之前，还是需要Maven来发布对应的pom.xml。从Gradle6.0开始，官方提供 `java-platform`插件，无需复杂的配置，即可生成BOM文件。





## io.spring.dependency-management

`io.spring.dependency-management`是spring团队提供的一个插件，提供了maven like的方式进行bom相关的管理。

他们之间的[差异](https://github.com/spring-gradle-plugins/dependency-management-plugin/issues/211)在仓库上有对应的说明。

spring团队提供的插件，和原生的差别有以下三点：

1. 使用maven like的方式引入BOM，也就是说可以轻松引入多个bom。
2. 使用更简单的方式来处理BOM中传递过去的`properties`。
3. 使用maven的处理方式和表现形式来排除某些依赖的版本，并使得这些限制也能通过BOM传递（原生的至少到8.0还不支持）。



也许第三点gradle团队会在原生中解决，但是，第一点和第二点是不同的风格choice，估计gradle团队永远不会这么干。而且这个插件也是成熟的产品

[详细的手册][io.spring.dependency-management 手册]

原生的方式制作BOM已经在网上有不少的教程。这里基于我的插件选型，选择了spring团体提供的插件来制作和使用BOM，说实话，在排除依赖上和管理方式上spring团队提供的插件非常好用。另外作为java程序员，在使用springCloud时，官方提供的也是基于这个插件的版本管理。

[spring团队提供的使用插件管理依赖版本](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/htmlsingle/#managing-dependencies.dependency-management-plugin)

## 使用spring团推提供的插件制作和发布自己的BOM

这里我假设 引入多个BOM，同时还有自己的依赖又需要指定。

两个BOM分别是junit的和springCloud的，必要说明和build.gradle示例如下：

```groovy
plugins {
    // 版本管理
	id "io.spring.dependency-management" version "1.1.2"
    // 发布工具（发布BOM在文章后续会提到）
	id "maven-publish"
}

group = 'org.example'
version = '1.0-SNAPSHOT'

// 设置属性，建议通过这种方式设置版本，这样使用这个BOM的工程可以通过修改properties来轻松的改变版本
ext {
	set('springCloudVersion', "2022.0.1")
	set('junitBomVersion', "5.9.1")
    set('guavaVersion', "32.1.1-jre")
}
// io.spring.dependency-management插件提供的DSL来管理依赖
dependencyManagement {
    // 管理的依赖
	dependencies {
        // 指定guava的
        dependency "com.google.guava:guava:${guavaVersion}"
	}
	// 导入 既存的BOM，一个是springCloud的另一个是junit的
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
		mavenBom "org.junit:junit-bom:${junitBomVersion}"
	}
}
// maven-publish插件提供的DSL管理发布产出到maven仓库
publishing {
    // 定义maven远端仓库信息
	repositories {
		maven {
            // 从系统变量中获取认证的用户名和密码
			def userName = System.getProperty("un")
			def passWord = System.getProperty("ps")
            // 根据版本号后缀选择远端仓库地址
			def releasesRepoUrl = "https://oss.sonatype.org/service/local/staging/deploy/maven2/"
			def snapshotsRepoUrl = "https://oss.sonatype.org/content/repositories/snapshots/"
			url = version.endsWith('SNAPSHOT') ? snapshotsRepoUrl : releasesRepoUrl
			// 注入认证信息
			credentials {
				username userName
				password passWord
			}
		}
	}
    // 定义发布信息
	publications {
		mavenJava(MavenPublication) {
            // 定义POM文件的一些必备信息，这样就可以使用 generatePomFileForMavenJavaPublication task 任务进行pom文件生成，然后你就可以在 目录build/publications/mavenJava/下看到pom-default文件
			pom {
				name = "bom-test"
				description = "bom for depency-management"
				developers {
					developer {
						id = 'light'
						name = 'xtreme light'
						email = 'xtremelight17@gmail.com'
					}
				}
			}
		}
	}
}
```

产出的`pom-default`文件内容应该如下

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>org.example</groupId>
  <artifactId>bom-test</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging>
  <name>bom-test</name>
  <description>bom for depency-management</description>
  <developers>
    <developer>
      <id>light</id>
      <name>xtreme light</name>
      <email>xtremelight17@gmail.com</email>
    </developer>
  </developers>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.junit</groupId>
        <artifactId>junit-bom</artifactId>
        <version>5.9.1</version>
        <scope>import</scope>
        <type>pom</type>
      </dependency>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>2022.0.1</version>
        <scope>import</scope>
        <type>pom</type>
      </dependency>
      <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>32.1.1-jre</version>
      </dependency>
    </dependencies>
  </dependencyManagement>
</project>

```

这个pom 文件就可以给其他的项目使用了

### 修改版本

因为用了ext的配置，那么就可以在具体的项目上直接用

```groovy
ext['slf4j.version'] = '1.7.20'
```

来修改某个版本。

如果某个版本是间接依赖的，那么在BOM中显式声明这个依赖，在具体的项目中可以通过ext来修改。

## 原生BOM的使用和制作

参考spring团队如何使用原生BOM的，源码：https://github.com/spring-projects/spring-framework

**java-platform** 插件不能和 `java` 或者 `java-library`公用，也就是理论上一个模块要不是 project 要不是 platform。

```groovy

plugins {
    id 'maven-publish'
    id 'java-platform'
}

version '0.1.1-SNAPSHOT'

javaPlatform {
    // 允许增加依赖传递而不仅仅是定义约束，在引入第三方的BOM时需要开启
    allowDependencies()
}
dependencies {
    // 引入其他的BOM
    api platform('org.springframework.boot:spring-boot-dependencies:2.2.6.RELEASE')
    api platform('org.springframework.cloud:spring-cloud-dependencies:Greenwich.SR3')
    api platform('org.springframework.cloud:spring-cloud-contract-dependencies:2.2.3.RELEASE')
    api platform('org.junit:junit-bom:5.3.2')
    // 自己的约束
    constraints {
        api 'com.google.guava:guava:27.0.1-jre'

        api 'ch.vorburger.mariaDB4j:mariaDB4j-springboot:2.4.0'
        api 'org.mariadb.jdbc:mariadb-java-client:2.2.5'

        api 'org.mockito:mockito-core:2.22.0'
        api 'org.mockito:mockito-junit-jupiter:2.22.0'
        api 'org.assertj:assertj-core:3.11.1'
    }
}

publishing {
    repositories {
        maven {
            credentials {
                username = 'xxxx'
                password = 'xxxx'
            }

            def releasesRepoUrl = 'http://xxxxxxx/artifactory/libs-release/'
            def snapshotsRepoUrl = 'http://xxxxxx/artifactory/libs-snapshot/'
            url = version.endsWith('SNAPSHOT') ? snapshotsRepoUrl : releasesRepoUrl
        }
    }
	// 原生发布
    publications {
        myPlatform(MavenPublication) {
            from components.javaPlatform
        }
    }
}

```

Maven BOM 和 gradle `Java platform` 主要不一样是， Gradle dependencies 扩展了 `constraints`用以更细粒度的灵活配置, 佐以runtime/api 进行使用。 API 在使用platform 引入的时候在编译时候使用， runtime 运行时， 举例：

```groovy
dependencies {
    constraints {
        api 'commons-httpclient:commons-httpclient:3.1'
        runtime 'org.postgresql:postgresql:42.2.5'
    }
}
```



这里用 `constraints` 而不是 `dependencies` ， `constraints` 只有在依赖被直接或者间接引入的时候才生效， 如上面， 如果项目没有依赖 `commons-httpclient` 那么不会生效， 如果项目引入了 `commons-httpclient` 并且是 `3.0` 版本， 不管是直接还是间接的， 那么会强制使用 `3.1` 版本而不是 `3.0`。

默认防止混淆在platform 里面误操添加 `dependency` 而不是 `constraint`, 这样会导致 Gradle 检验错误. 所以如果你想 `dependencies` 和 `constraints`, 都可以用， 你可以添加显示的声明:

```groovy
javaPlatform {
    allowDependencies()
}
```

如果要定义自己的BOM，那么就可能需要引入第三方的BOM，上述的配置就一定需要

参考上面的spring团队是如何使用BOM的，就可以看到。



### version-catalog

使用 `version-catalog`也可以达到`platform`一样的效果，相比较而言还更加轻量。

需要用的是`version-catalog`插件

定义方式变为

```groovy
catalog {
    // declare the aliases, bundles and versions in this block
    versionCatalog {
        library('my-lib', 'com.mycompany:mylib:1.2')
    }
}
```

发布方式变为（当然也是用`maven-publish`插件进行发布）：

```groovy
catalog {
    // declare the aliases, bundles and versions in this block
    versionCatalog {
        library('my-lib', 'com.mycompany:mylib:1.2')
    }
}
```

但是产出物就会是`libs.versions.toml`文件，然后其他的gradle应用可以使用这个文件。

toml文件有点像：

```toml
[versions]
common = "1.4"

[libraries]

my-lib = "com.mycompany:mylib:1.4"
my-other-lib = { module = "com.mycompany:other", version = "1.4" }
my-other-lib2 = { group = "com.mycompany", name = "alternate", version = "1.4" }
mylib-full-format = { group = "com.mycompany", name = "alternate", version = { require = "1.4" } }
[plugins]
short-notation = "some.plugin.id:1.4"
long-notation = { id = "some.plugin.id", version = "1.4" }
reference-notation = { id = "some.plugin.id", version.ref = "common" }

```

使用方式

```groovy
dependencyResolutionManagement {
    versionCatalogs {
        libs {
            from(files("../gradle/libs.versions.toml"))
        }
    }
}
```

可以用这种方式，但是这样得考虑是不是要兼容maven的BOM



### 自定义依赖版本

```groovy
configurations.all {
	resolutionStrategy.eachDependency { DependencyResolveDetails details ->
		if (details.requested.group == 'org.slf4j') {
			details.useVersion '1.7.20'
		}
	}
}
```





## 拓展

[io.spring.dependency-management 手册]:https://github.com/spring-gradle-plugins/dependency-management-plugin/issues/211

[gradle 高级功能]: https://dearxue.com/open/book/tool/gradle/009_Spring_gradle.html#init



