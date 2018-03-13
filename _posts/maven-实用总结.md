---
title: maven-实用总结
date: 2017-07-19 00:15:23
tags: maven
categories: tool 
---
# setting 配置  
Maven用户可以选择配置$M2_HOME/conf/settings.xml或者~/.m2/settings.xml。前者是全局范围的，整台机器上的所有用户都会直接受到该配置的影响，而后者是用户范围的，只有当前用户才会受到该配置的影响。  
## 本地资源库位置  
``` xml
<settings>
   <localRepository>F:/liang/maven/localRepository</localRepository>
</settings>
```

<!-- more -->  

## 需要输出的时候提示
interactiveMode 用于决定maven是否在需要输出的时候提示你，默认true。如果是false，它将使用合理的默认值，或者基于一些设置

## 配置远程http代理：
```xml
<settings>
<proxies>
    <proxy>
      <id>my-proxy</id>
      <active>true</active>
      <protocol>http</protocol>
      <host>218.14.227.197</host>
      <port>3128</port>
      <!--
      <username>***</username>
      <password>***</password>
      <nonProxyHosts>repository.mycom.com|*.google.com</nonProxyHosts>
      -->
    </proxy>
  </proxies>
  </settings>
```
说明：  
这段配置十分简单，proxies下可以有多个proxy元素，如果你声明了多个proxy元素，则默认情况下第一个被激活的proxy会生效。
这里声明了一个id为my-proxy的代理，active的值为true表示激活该代理，protocol表示使用的代理协议，这里是http。
当然，最重要的是指定正确的主机名（host元素）和端口（port元素）。
上述XML配置中注释掉了username、password、nonProxyHost几个元素，当你的代理服务需要认证时，就需要配置username和password。
nonProxyHost元素用来指定哪些主机名不需要代理，可以使用 | 符号来分隔多个主机名。
此外，该配置也支持通配符，如*.google.com表示所有以google.com结尾的域名访问都不要通过代理。

## 配置全局
1. settings.xml中时意味着该profile是全局的，所以只能配置范围宽泛一点配置信息，比如远程仓库等。而一些比较细致一点的需要定义在项目的pom.xml中。
2. profile可以让我们定义一系列的配置信息，然后指定其激活条件。根据每个profile对应不同的激活条件和配置信息，从而达到不同环境使用不同配置。
3. 例子：通过profile定义jdk1.5以上使用一套配置，jdk1.5以下使用另外一套配置；或者通过操作系统来使用不同的配置信息。
4. settings.xml中的信息有repositories、pluginRepositories和properties。定义在properties的值可以在pom.xml中使用。

```
<profiles>
    <profile>
              <id>test</id>
              <activation>
                 <activeByDefault>false</activeByDefault>
                 <jdk>1.5</jdk>
                 <os>
                     <name>Windows XP</name>
                     <family>Windows</family>
                     <arch>x86</arch>
                     <version>5.1.2600</version>
                 </os>
                 <property>
                     <name>mavenVersion</name>
                     <value>2.0.3</value>
                 </property>
                 <file>
                <exists>${basedir}/file2.properties</exists>
               <missing>${basedir}/file1.properties</missing>
                </file>
             </activation>
         </profile>
</profiles>
```

* jdk：检测到对应jdk版本就激活
* os：针对不同操作系统
* property：当maven检测到property（pom中如${name}这样的）profile将被激活
* file：如果存在文件，激活，不存在文件激活  

通过以下命令查看哪些profile将生效`mvn help:active-profiles`

**properites** 

Maven的属性是值占位符，就像Ant中的一样。如果X是一个属性的话，在POM中可以使用${X}来进行任意地方的访问。他们来自于五种不同的风格，所有都可以从settings.xml文件中访问到。

1. env.x：“env.”前缀会返回当前的环境变量。如${env.PATH}就是使用了$path环境变量（windosws中的%PATH%）。
2.  project.x：一个点“.”分割的路径，在POM中就是相关的元素的值。例如：<project><version>1.0</version></project>就可以通过${project.version}来访问。
3. settings.x：一个点“.”分割的路径，在settings.xml中就是相对应的元素的值，例如：<settings><offline>false</offline></settings>就可以通过`\${settings.offline}`来访问。
4. Java系统属性：通过java.lang.System.getProperties()来访问的属性都可以像POM中的属性一样访问，例如：${java.home}
5.  x：被<properties/>或者外部文件定义的属性，值可以这样访问${someVar}  

```
  <profiles>
    <profile>
      ...
      <properties>
        <user.install>${user.home}/our-project</user.install>
      </properties>
      ...
    </profile>
  </profiles>
```


上面这个profile如果被激活，那么在pom中${user.install}就可以被访问了。  

**Repositories**  

Repositories是远程项目集合maven用来移植到本地仓库用于构建系统。如果来自本地仓库，Maven调用它的插件和依赖关系。不同的远程仓库可能包含不同的项目，当profile被激活，他们就会需找匹配的release或者snapshot构件。
```
<profiles>
    <profile>
      ...
      <repositories>
        <repository>
          <id>codehausSnapshots</id>
          <name>Codehaus Snapshots</name>
          <releases>
            <enabled>false</enabled>
            <updatePolicy>always</updatePolicy>
            <checksumPolicy>warn</checksumPolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>never</updatePolicy>
            <checksumPolicy>fail</checksumPolicy>
          </snapshots>
          <url>http://snapshots.maven.codehaus.org/maven2</url>
          <layout>default</layout>
        </repository>
      </repositories>
      <pluginRepositories>
        ...
      </pluginRepositories>
      ...
    </profile>
  </profiles>
```
 
1.  releases，snapshots：这是各种构件的策略，release或者snapshot。这两个集合，POM就可以根据独立仓库任意类型的依赖改变策略。如：一个人可能只激活下载snapshot用来开发。
2. enable：true或者false，决定仓库是否对于各自的类型激活(release 或者 snapshot)。
3. updatePolicy: 这个元素决定更新频率。maven将比较本地pom的时间戳（存储在仓库的maven数据文件中）和远程的. 有以下选择: always, daily (默认), interval:X (x是代表分钟的整型) ， never.
4. checksumPolicy：当Maven向仓库部署文件的时候，它也部署了相应的校验和文件。可选的为：ignore，fail，warn，或者不正确的校验和。
5. layout：在上面描述仓库的时候，提到他们有统一的布局。Maven 2有它仓库默认布局。然而，Maven 1.x有不同布局。使用这个元素来表明它是default还是legacy。

## 插件组
```
<pluginGroups>
    <pluginGroup>org.mortbay.jetty</pluginGroup>
</pluginGroups>
```
运行 `mvn jetty run`
我们同样可以在pom文件中看到相似的配置，只是在这配置了就起到全局的作用，而不用每个项目中pom配置jetty

## 认证配置
```
<servers>
    <!--使用登录方式-->
    <server>
          <id>deploymentRepo</id>
          <username>repouser</username>
          <password>repopwd</password>
        </server>

        <!-- 使用秘钥认证 -->
        <server>
          <id>siteServer</id>
          <privateKey>/path/to/private/key</privateKey>
          <passphrase>可空</passphrase>
        </server>
</servers>
```
这是一个认证配置的列表,根据系统中使用的server-id控制。认证配置在maven连接到远程服务时使用。  

## 镜像仓库
```
<mirrors>
    <mirror>
      <id>mirrorId</id>
      <mirrorOf>repositoryId</mirrorOf>
      <name>Human Readable Name for this Mirror.</name>
      <url>http://my.repository.com/repo/path</url>
    </mirror>
</mirrors>
```

* id：用于继承和直接查找，唯一
* mirrorOf：镜像所包含的仓库的Id
* name：唯一标识，用于区分镜像站
* url：镜像路径



# Maven 项目结构
| 目录                          |                           目的 |
| :-----------------------------|------------------------------: | 
| \${basedir}                    |      存放pom.xml和所有的子目录 |
| \${basedir}/src/main/java      |               项目的java源代码 |
| \${basedir}/src/main/resources |   项目的资源，比如说数据库文件 |
| \${basedir}/src/test/java      | 项目的测试类，比如说 JUnit代码 |
| \${basedir}/src/test/resources |                 测试使用的资源 |
| \${basedir}/target             |                   编译好的文件 |



# pom.xml配置  
## 项目基本pom.xml  
```xml
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0  
http://maven.apache.org/maven-v4_0_0.xsd">  
  <modelVersion>4.0.0</modelVersion>  
      <!-- The Basics -->  
    <groupId>...</groupId>  
    <artifactId>...</artifactId>  
    <version>...</version>  
    <packaging>...</packaging>  
    <dependencies>...</dependencies>  
    <parent>...</parent>  
    <dependencyManagement>...</dependencyManagement>  
    <modules>...</modules>  
    <properties>...</properties>  
  
    <!-- Build Settings -->  
    <build>...</build>  
    <reporting>...</reporting>  
  
    <!-- More Project Information -->  
    <name>...</name>  
    <description>...</description>  
    <url>...</url>  
    <inceptionYear>...</inceptionYear>  
    <licenses>...</licenses>  
    <organization>...</organization>  
    <developers>...</developers>  
    <contributors>...</contributors>  
  
    <!-- Environment Settings -->  
    <issueManagement>...</issueManagement>  
    <ciManagement>...</ciManagement>  
    <mailingLists>...</mailingLists>  
    <scm>...</scm>  
    <prerequisites>...</prerequisites>  
    <repositories>...</repositories>  
    <pluginRepositories>...</pluginRepositories>  
    <distributionManagement>...</distributionManagement>  
    <profiles>...</profiles> 
</project>   

```
说明：  
代码的第一行是XML头，指定了该xml文档的版本和编码方式。  
代码的第二行是project元素，project是所有pom.xml的根元素，它还声明了一些POM相关的命名空间及xsd元素，虽然这些属性不是必须的，但使用这些属性能够让第三方工具（如IDE中的XML编辑器）帮助我们快速编辑POM。  
这段代码中最重要的是groupId，artifactId和version三行。这三个元素定义了一个项目基本的坐标。  
* modelVersion指定了当前POM模型的版本，对于Maven2及Maven 3来说，它只能是4.0.0。
* <!-- The Basics -->
* groupId定义了项目属于哪个组，你会使用项目的java包的根名称作为group ID。也许选择与项目相关的名称作为group ID  
* artifactId定义了当前Maven项目在组中唯一的ID。  
* version指定了项目当前的版本1.0-SNAPSHOT。SNAPSHOT意为快照，说明该项目还处于开发中，是不稳定的版本。 
* packaging定义artifact打包的方式，如pom, jar, maven-plugin, ejb, war, ear, rar, par。默认为jar。这个不仅表示项目最终产生何种后缀的文件，也表示build过程使用什么样的lifecycle。  
* dependencies 表示依赖，在子节点dependencies中添加具体依赖的groupId artifactId和version
* parent 表示父pom，命令行中执行 mvn help:effective-pom来查看Super POM对你的POM的影响。
* dependencyManagement 用于在parent项目定义一个依赖
* modules 在多模块的项目中，可以将一个模块的集合配置到modules中。列出的模块不需要考虑顺序.
* properties  定义变量属性。
* <!-- Build Settings --> 
* build 表示 分为基础构件和项目构件。项目构件放到`profiles->profile->build`  
* reporting 包含的属性对应到site阶段（见Maven生命周期）。特定的Maven插件能产生定义和配置在reporting元素下的报告，例如：产生Javadoc报告。
* <!-- More Project Information --> 
* name元素声明了一个对于用户更为友好的项目名称。 
* description 表示项目的描述，在maven生成的文档中使用
* url：指定项目站点，通常用于maven产生的文档中。  
* inceptionYear 项目创建年份，4位数字。当产生版权信息时需要使用这个值
* licenses 是定义一个项目（或部分项目）怎么和什么时候可以被使用的法律文件。
* organization 项目所在组织的基本信息。
* developers 项目的核心开发者信息。
* contributors 项目的辅助人员信息。
* <!-- Environment Settings --> 
* issueManagement 使用的缺陷跟踪系统（Bugzilla，TestTrack，ClearQuest，等）信息，主要用于产生项目文档。
* ciManagement 持续构建系统配置。
* mailingLists 定义了该项目需要保持联系的人员，主要是开发人员和用户的邮件列表。
* scm 这里是你存放版本管理信息到POM的地方。
* prerequisites 为了让该POM正确的执行所需要的先决条件.
* repositories 定义Maven的仓库地址.
* pluginRepositories 插件地址
* distributionManagement 管理build生成的组件和资源文件和分发。
* profiles POM 4.0的一个新特征是改变项目被构建的环境的settings的能力。

**其中groupId:artifactId:version唯一确定了一个artifact**

##  添加类库
### 直接添加
```xml
<dependencies>  
    <dependency>  
       <groupId></groupId>  
       <artifactId></artifactId>  
       <version></version>  
       <scope></scope>  
       <systemPath></systemPath>
       <optional></optional>
    </dependency>  
  </dependencies>
```
说明：  
代码中添加了dependencies元素，该元素下可以包含多个dependency元素以声明项目的依赖。添加最基本的groupId、artifactId和version。scope为依赖范围，主要有五个值:  

 1. compile，缺省值，适用于所有阶段，会随着项目一起发布。
 2. provided，类似compile，期望JDK、容器或使用者会提供这个依赖。如servlet.jar。
 3. runtime，只在运行时使用，如JDBC驱动，适用运行和测试阶段。
 4. test，只在测试时使用，用于编译和运行测试代码。不会随项目发布。
 5. system，类似provided，需要显式提供包含依赖的jar，Maven不会在Repository中查找它。

 systemPath：当scop配置为system时就需要它了,\$(object.path)/lib/junit.jar
 optional：设置为true，标识该依赖只对该项目有效，如果其他项目依赖该项目，该依赖将不会传递。

### 依赖添加：
``` xml
<dependencies>  
    <dependency>  
        <groupId>org.apache.maven</groupId>  
        <artifactId>maven-embedder</artifactId>  
        <version>2.0</version>  
        <exclusions>  
            <exclusion>  
                <groupId>org.apache.maven</groupId>  
                <artifactId>maven-core</artifactId>  
		<!--<groupId>*</groupId>  
                <artifactId>*</artifactId>-->
            </exclusion>  
        </exclusions>  
    </dependency>  
    ...  
</dependencies> 
```
该配置告诉Maven你不想包含的该依赖的依赖（即，依赖传递的依赖）。也可以使用通配符，表示排除所有传递的依赖。

## 增加父pom
```xml
<parent>
	<groupId></groupId>
	<artifactId></artifactId>
	<version></version>
	<relativePath></relativePath>
</parent>
```
说明：
所有的Maven pom文件都继承自一个父pom。如果没有指定父pom，则该pom文件继承自根pom。  
子pom文件的设置可以覆盖父pom文件的设置，只需要在子pom文件里指定新的设置即可。  
relativePath是可选的，指定parent的搜索路径，如果配置了，Maven将首先搜索relativePath，然后本地仓库，最后远程仓库查找parentPOM。

### 子项目增加parent的依赖
parent 增加dependency，子项目中添加dependencies。
``` xml
    <dependencyManagement>  
        <dependencies >  
            <dependency>  
                <groupId>junit</groupId>  
                <artifactId>junit</artifactId>  
                <version>4.0</version>  
                <type>jar</type>  
                <scope>compile</scope>  
                <optional>true</optional>  
            </dependency>  
        </dependencies >  
    </dependencyManagement>  
```
继承的pom 就能设置他们的依赖为:
```xml
<dependencies >  
    <dependency >  
        <groupId>junit</groupId>  
        <artifactId>junit</artifactId>  
    </dependency >  
</dependencies > 
```
继承的POM的dependency的其它信息可以从parent的dependencyManagement中获得，这样的好处在于可以将依赖的细节放在一个控制中心，子项目就不用在关心依赖的细节，只需要设置依赖。  

## 多模块的项目集合到modules中
```xml
    <modules>  
        <module>my-project</module>  
        <module>another-project</module>  
    </modules>  
```
父项目聚合很多子项目。每个项目，不管是父子，都含有一个pom.xml文件。而且要注意的是，小括号中标出了每个项目的打包类型。父项目是pom,也只能是pom。子项目有jar，或者war。根据它包含的内容具体考虑。

## 构件配置
### 基本构件配置
```xml
	<build>  
		<defaultGoal>install</defaultGoal>  
		 <directory>${basedir}/target</directory>  
		 <finalName>${artifactId}-${version}</finalName>  
		<filters>  
			 <filter>filters/filter1.properties</filter>  
		</filters>  
	 ... 
	</build> 
```
1. defaultGoal：执行build任务时，如果没有指定目标，将使用的默认值，如：在命令行中执行mvn，则相当于执行mvn install；
2. directory：build目标文件的存放目录，默认在${basedir}/target目录；
3. finalName：build目标文件的文件名，默认情况下为${artifactId}-${version}；
4. filter：定义*.properties文件，包含一个properties列表，该列表会应用的支持filter的resources中。  
也就是说，定义在filter的文件中的"name=value"值对会在build时代替${name}值应用到resources中。
Maven的默认filter文件夹是\${basedir}/src/main/filters/。 
### 资源位置
```xml
<build>  
    ...  
    <resources>  
         <resource>  
            <targetPath>META-INF/plexus</targetPath>  
            <filtering>false</filtering>  
            <directory>${basedir}/src/main/plexus</directory>  
            <includes>  
                <include>configuration.xml</include>  
            </includes>  
            <excludes>  
                <exclude>**/*.properties</exclude>  
            </excludes>  
         </resource>  
    </resources>  
    <testResources>  
        ...  
    </testResources>  
    ...  
</build> 
```
1. resources：一个resource元素的列表，每一个都描述与项目关联的文件是什么和在哪里；
2. targetPath：指定build后的resource存放的文件夹。该路径默认是basedir。通常被打包在JAR中的resources的目标路径为META-INF；
3. filtering：true/false，表示为这个resource，filter是否激活。
4. directory：定义resource所在的文件夹，默认为${basedir}/src/main/resources；
5. includes：指定作为resource的文件的匹配模式，用*作为通配符；
6. excludes：指定哪些文件被忽略，如果一个文件同时符合includes和excludes，则excludes生效；
7. testResources：定义和resource类似，但只在test时使用，默认的test resource文件夹路径是${basedir}/src/test/resources，test resource不被部署。

### 添加插件
```xml
<project>  
  <build>  
    <plugins>  
       <plugin>  
            <groupId>org.apache.maven.plugins</groupId>  
            <artifactId>maven-jar-plugin</artifactId>  
            <version>2.0</version>  
            <extensions>false</extensions>  
            <inherited>true</inherited>  
            <configuration>  
                <classifier>test</classifier>  
            </configuration>  
            <dependencies>...</dependencies>  
            <executions>...</executions>  
       </plugin>  
    </plugins>  
  </build>  
</project> 
```
除了groupId:artifactId:version标准坐标，plugin还需要如下属性：
1. extensions：true/false，是否加载plugin的extensions，默认为false；
2. inherited：true/false，这个plugin是否应用到该POM的孩子POM，默认true；
3. configuration：配置该plugin期望得到的properies，如上面的例子，我们为maven-jar-plugin的Mojo设置了classifier属性；
4. dependencies：同base build中的dependencies有同样的结构和功能，但这里是作为plugin的依赖，而不是项目的依赖。
5. executions：plugin可以有多个目标，每一个目标都可以有一个分开的配置，甚至可以绑定一个plugin的目标到一个不同的阶段。executions配置一个plugin的目标的execution。

### 添加主函数位置
```xml
    <plugin>  
      <groupId>org.apache.maven.plugins</groupId>  
      <artifactId>maven-shade-plugin</artifactId>  
      <version>1.2.1</version>  
      <executions>  
        <execution>  
          <phase>package</phase>  
          <goals>  
            <goal>shade</goal>  
          </goals>  
          <configuration>  
            <transformers>  
              <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
		    <mainClass>com.liang.controller.Main</mainClass>  
             </transformer>  
           </transformers>  
         </configuration>  
         </execution>  
      </executions>  
    </plugin>  
```
## 添加法律文献
```
    <licenses>  
        <license>  
            <name>The Apache Software License, Version 2.0</name>  
            <url>http://www.apache.org/licenses/LICENSE-2.0.txt</url>  
            <distribution>repo</distribution>  
            <comments>A business-friendly OSS license</comments>  
        </license>  
    </licenses>  
```
licenses是定义一个项目（或部分项目）怎么和什么时候可以被使用的法律文件。项目只能列出直接应用到该项目的licenses，不应该列出项目所依赖的组件的licenses。Maven当前只是将这些licenses展示在项目生成的站点上。
1）name、url和comments：自我说明；
2）distribution：说明该项目在什么情况下可以合法传播。 

## 增加项目核心开发者信息
```
<developers>  
    <developer>  
        <id>eric</id>  
        <name>Eric</name>  
        <email>eredmond@codehaus.org</email>  
        <url>http://eric.propellors.net</url>  
        <organization>Codehaus</organization>  
        <organizationUrl>http://mojo.codehaus.org</organizationUrl>  
        <roles>  
            <role>architect</role>  
            <role>developer</role>  
        </roles>  
        <timezone>-6</timezone>  
        <properties>  
            <picUrl>http://tinyurl.com/prv4t</picUrl>  
        </properties>  
    </developer>  
</developers>
```
* id, name, email：开发者的信息；
* organization, organizationUrl：开发者的组织信息；
* roles：开发者所承担的角色，可能有多个；
* timezone：所在时区；
* properties：开发者的其它信息； 

## 定义Maven的仓库地址
```
<repositories>  
    <repository>  
        <releases>  
            <enabled>false</enabled>  
            <updatePolicy>always</updatePolicy>  
            <checksumPolicy>warn</checksumPolicy>  
        </releases>  
        <snapshots>  
            <enabled>true</enabled>  
            <updatePolicy>never</updatePolicy>  
            <checksumPolicy>fail</checksumPolicy>  
        </snapshots>  
        <id>codehausSnapshots</id>  
        <name>Codehaus Snapshots</name>  
        <url>http://snapshots.maven.codehaus.org/maven2</url>  
        <layout>default</layout>  
    </repository>  
</repositories>  
```
1. releases、snapshots：针对组件的发布类型能干采取的策略（release或者snapshot）；
2. enabled：true/false，仓库对于对应的类型是否激活；
3. updatePolicy：指定Maven尝试更新的频率，Maven将比较本地POM的时间戳（存储在仓库的maven-metadata文件中）和远端POM的时间戳，选择为：always，daily（默认），interval:X（X为整数，单位分钟）或者never；
4. checksumPolicy：当Maven部署文件到仓库时，它也部署了对应的校验和文件。你可以选择ignore、fail或者warn来响应当文件丢失或者不正确后的操作；
5. layout：default/legacy，Maven 2和Maven 3都是用default layout。
pluginRepositories的配置方式和repositories一致。

## Distribution Management
管理build生成的组件和资源文件和分发。
### 基本属性
downloadUrl：另一个POM配置用于获取该POM的组件的仓库的url；
status：这个值不能手动配置，当该组件上传到仓库时，Maven将设置这个项目的状态，它的有效值为：
none：没有特殊状态，默认；
converted：仓库的管理人员从一个对Maven 2来说更早的版本转换到这个POM；
partner：这个组件有被同步到一个合作仓库；
deployed：最通用的状态，这个组件从Maven2或者Maven2部署；
verified：这个项目被查证了，应该被终止。

### Repository
distributionManagement通过repository定义当组件部署时，项目可以从哪里（和怎么）获取远端仓库。

id：仓库的唯一标识；
name：人类可读的名称；
uniqueVersion：true/false，表示部署到该仓库的组件是否应该获取一个唯一产生的版本号，并用这个版本号作为地址的一部分；
url：指定用于传送构建成的组件（包括POM文件和校验和数据）到仓库的网络地址和传输协议，是Repository的核心；
layout：default/legacy，同project下的repository 元素。

# maven 命令

## mvn 常用参数
* mvn -e  显示详细错误
* mvn -U  强制更新snapshot类型的插件或依赖库（否则maven一天只会更新一次snapshot依赖）
* mvn -o  运行offline模式，不联网更新依赖
* mvn -N  仅在当前项目模块执行命令，关闭reactor
* mvn -pl  module_name在指定模块上执行命令
* mvn -ff  在递归执行命令过程中，一旦发生错误就直接退出
* mvn -Dxxx=yyy 指定java全局属性
* mvn -Pxxx 引用profile xxx

## mvn 简单命令
### mvn test-compile 
编译测试代码
### mvn test 
运行程序中的单元测试
### mvn validate 
验证项目是否正确以及必须的信息是否可用
### mvn compile
编译源代码
### mvn test 
测试编译后的代码，即执行单元测试代码
### mvn package
打包编译后的代码，在target目录下生成package文件
### mvn package -Prelease
打包，并生成部署用的包，比如deploy/*.tgz
### mvn integration-test 
处理package以便需要时可以部署到集成测试环境
### mvn verify 
检验package是否有效并且达到质量标准
### mvn install 
安装package到本地仓库，方便本地其它项目使用, 该命令执行`install`阶段（是默认构建阶段的一部分），编译项目，将打包的JAR文件复制到本地的Maven仓库。事实上，该命令在执行install之前，会执行在构建周期序列中位于install之前的所有阶段。
### mvn deploy
部署，拷贝最终的package到远程仓库和替他开发这或项目共享，在集成或发布环境完成
### mvn site
生成项目相关信息的网站

## mvn 组合命令
### mvn clean compile
clean告诉Maven清理输出目录target/，compile告诉Maven编译项目主代码，输出中我们看到Maven首先执行了clean:clean任务，删除target/目录，默认情况下Maven构建的所有输出都在target/目录中；接着执行resources:resources任务（未定义项目资源，暂且略过）；最后执行compiler:compile任务，将项目主代码编译至target/classes目录(编译好的类为com/juvenxu/mvnbook/helloworld/HelloWorld.Class）。  
上文提到的clean:clean、resources:resources，以及compiler:compile对应了一些Maven插件及插件目标，比如clean:clean是clean插件的clean目标，compiler:compile是compiler插件的compile目标。  

### mvn clean test
Maven实际执行的可不止这两个任务，还有clean:clean、resources:resources、compiler:compile、resources:testResources以及compiler:testCompile。我们需要了解的是，在Maven执行测试（test）之前，它会先自动执行项目主资源处理，主代码编译，测试资源处理，测试代码编译等工作，这是Maven生命周期的一个特性。

### mvn clean package
POM中没有指定打包类型，使用默认打包类型jar。jar:jar任务负责打包  

### mvn clean install
执行了安装任务install:install，打开相应的文件夹看到Hello World项目的pom和jar。只有构件被下载到本地仓库后，才能由所有Maven项目使用。  
### mvn install:install-file 
* -Dfile=non-maven-proj.jar 
* -DgroupId=some.group 
* -DartifactId=non-maven-proj
* -Dversion=1 
* -Dpackaging=jar
用安装插件安装本地依赖

## mvn eclipse 命令
### mvn eclipse:eclipse 
生成eclipse项目文件
### mvn eclipse:clean
清除eclipse项目文件

## mvn archetype 命令
### mvn archetype:generate
Maven提供了Archetype以帮助我们快速勾勒出项目骨架。实际上是在运行插件maven-archetype-plugin。  
有很多可用的archetype供我们选择，包括著名的Appfuse项目的archetype，JPA项目的archetype等等。每一个archetype前面都会对应有一个编号，同时命令行会提示一个默认的编号，其对应的archetype为maven-archetype-quickstart，我们直接回车以选择该archetype，紧接着Maven会提示我们输入要创建项目的groupId、artifactId、 version、以及包名package  
*  -DgroupId=com.liang
*  -DartifactId=liang
*  -DarchetypeArtifactId=maven-archetype-quickstart    指定ArchetypeId   maven-archetype-webapp
*  -DinteractiveMode=false   表示是否使用交互模式,交互模式会让用户填写版本信息之类的,非交互模式采用默认值
### mvn archetype:create
创建一个maven项目提供参数如下：  

* -DgroupId=com.liang  
* -DartifactId=hello  
* -DpackageName=com.liang.hello 
* -Dversion=1.0

## mvn help 命令
### mvn help:system
该命令会打印出所有的java系统属性和环境变量。这些信息对我们日常的编程工作很有帮且。  

### mvn help:effective-pom
考虑到pom文件的继承关系，当Maven执行的时候可能很难确定最终的pom文件的内容。总的pom文件（所有继承关系生效后）被称为有效pom（effective pom）。  
执行以上命令，Maven会将有效pom输出到命令行。

## mvn 插件常用参数
### mvn -Dwtpversion=3.0 
指定maven版本
### mvn -Dmaven.test.skip=true 
如果命令包含了test phase，则忽略单元测试
### mvn -DuserProp=filePath 
指定用户自定义配置文件位置
### mvn -DdownloadSources=true -Declipse.addVersionToProjectName=true eclipse:eclipse 
生成eclipse项目文件，尝试从仓库下载源代码，并且生成的项目包含模块版本（注意如果使用公用POM，上述的开关缺省已打开）

## maven简单故障排除
### mvn -Dsurefire.useFile=false
如果执行单元测试出错，用该命令可以在console输出失败的单元测试及相关信息
set MAVEN_OPTS=-Xmx512m -XX:MaxPermSize=256m 调大jvm内存和持久代，maven/jvm out of memory error
### mvn -X maven log level
设定为debug在运行
### mvn debug 
运行jpda允许remote debug
### mvn –help 
不会就找这个吧。。

## mvn dependency:copy-dependencies
该命令执行`dependency`构建阶段中的`copy-dependencies`目标。



# maven 常用插件介绍

#专业名称解析

## Artifact
这个有点不好解释，大致说就是一个项目将要产生的文件，可以是jar文件，源文件，二进制文件，war文件，甚至是pom文件。每个artifact都由groupId:artifactId:version组成的标识符唯一识别。需要被使用(依赖)的artifact都要放在仓库(见Repository)中  
## Repositories
Repositories是用来存储Artifact的。如果说我们的项目产生的Artifact是一个个小工具，那么Repositories就是一个仓库，里面有我们自己创建的工具，也可以储存别人造的工具，我们在项目中需要使用某种工具时，在pom中声明dependency，编译代码时就会根据dependency去下载工具（Artifact），供自己使用。
对于自己的项目完成后可以通过mvn install命令将项目放到仓库（Repositories）中
仓库分为本地仓库和远程仓库，远程仓库是指远程服务器上用于存储Artifact的仓库，本地仓库是指本机存储Artifact的仓库，对于windows机器本地仓库地址为系统用户的.m2/repository下面。
对于需要的依赖，在pom中添加dependency即可，可以在maven的仓库中搜索：http://mvnrepository.com/
### Build Lifecycle
是指一个项目build的过程。maven的Build Lifecycle分为三种，分别为default（处理项目的部署）、clean（处理项目的清理）、site（处理项目的文档生成）。他们都包含不同的lifecycle。
Build Lifecycle是由phases构成的，下面重点介绍default Build Lifecycle几个重要的phase  

1. validate 验证项目是否正确以及必须的信息是否可用
2. compile 编译源代码
3. test 测试编译后的代码，即执行单元测试代码
4. package 打包编译后的代码，在target目录下生成package文件
5. integration-test 处理package以便需要时可以部署到集成测试环境
6. verify 检验package是否有效并且达到质量标准
7. install 安装package到本地仓库，方便本地其它项目使用
8. deploy 部署，拷贝最终的package到远程仓库和替他开发这或项目共享，在集成或发布环境完成  

以上的phase是有序的（注意实际两个相邻phase之间还有其他phase被省略，完整phase见lifecycle），下面一个phase的执行必须在上一个phase完成后。若直接以某一个phase为goal，将先执行完它之前的phase，如mvn install
将会先validate、compile、test、package、integration-test、verify最后再执行install phase

## Goal
goal代表一个特定任务
`A goal represents a specific task (finer than a build phase) which contributes to the building and managing of a project.`
目标表示特定任务 (比构建阶段更精细), 它有助于项目的构建和管理。
