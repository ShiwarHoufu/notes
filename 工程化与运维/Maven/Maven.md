Maven 是一个基于 **Java** 的**项目管理与构建工具**，由 Apache 开发。核心用来**自动化编译、打包、依赖管理、项目构建**生命周期管理，是 Java 后端项目最主流的构建工具。
# 一、Maven 的功能

![[IMG_20261018_164034.jpg]]
## 1. 依赖管理
### POM（Project Object Model）

项目对象模型，对应文件 **`pom.xml`**。 写好这段，Maven 会自动从中央仓库下载这个 jar
所有信息写在这个 xml 里：项目坐标、依赖、插件、打包方式。
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>2.0.32</version>
</dependency>
```
- **groupId**：组织 / 公司
- **artifactId**：项目 / 模块名
- **version**：版本号

>**自动传递依赖**：引入 A，A 依赖 B，Maven 自动把 B 也下载
### 仓库

先找本地仓库，找不到再去远程仓库（私服优先，再中央仓库）下载到本地
- **本地仓库**：电脑本地文件夹，下载过的 jar 存在这里
- **中央仓库**：Maven 官方远程仓库
- **私服**：企业内部搭建的私有仓库，存放公司内部 jar

## 2. 项目构建

提供**跨平台**自动项目构建：编译、测试、打包、发布

## 3. 统一项目结构
![[IMG_20261018_165509.jpg|287]]

# 二、Maven 的生命周期

IDEA 中侧边栏展示：（双击即可执行）
![[Pasted image 20261018171654.png|372]]

| 生命周期        | 作用                   |
| ----------- | -------------------- |
| **clean**   | 清理工作，移除上一次构建生成的文件    |
| **default** | 核心工作：编译、测试、打包、安装、部署等 |
| **site**    | 生成报告、发布站点            |

- 三套生命周期**相互独立**：执行 default 生命周期的时候，不会自动执行 clean
- **每套内部**包含若干阶段，阶段**有顺序**，后面的阶段**依赖**于前面的阶段。

> 每一套内部其实还有很多细分的阶段
>![[IMG_20261018_171755.jpg|511]]

其中标红的5个关键阶段：

| 阶段        | 作用                        |
| --------- | ------------------------- |
| `clean`   | 移除上一次构建生成的文件              |
| `compile` | 编译项目源代码                   |
| `test`    | 用合适的单元测试框架运行测试（如 JUnit）   |
| `package` | 将编译后的文件打包，如 `jar`、`war` 等 |
| `install` | 安装项目到本地仓库                 |

# 三、配置 Maven

**1. 配置本地仓库**
在合适的位置创建本地仓库，比如在D盘创建`maven-repository`
然后在配置文件中指定本地仓库位置：
> 配置文件位置`apache-maven-3.9.16\conf\settings.xml`
```xml
<localRepository>D:\maven-repository</localRepository>
```

**2. 配置阿里云私服**
在配置文件中修改`<mirrors>`标签：
```xml
<mirrors> 
	<mirror> 
		<id>aliyunmaven</id> 
		<mirrorOf>central</mirrorOf> 
		<name>阿里云公共仓库</name>
		<url>https://maven.aliyun.com/repository/public</url> 
	</mirror>
</mirrors>
```

**3. 配置环境变量**
配置系统变量 **MAVEN_HOME** ,并将其 bin 目录加入 PATH 

**4.  IDEA 中配置 Maven 环境**
- 需要配置：安装目录、配置文件位置、仓库目录
![[Pasted image 20261105152237.png]]