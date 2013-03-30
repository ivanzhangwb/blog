---
layout: post
title: "用Maven构建Scala"
permalink: "/2013/scala/scala-with-maven.html"
date: 2013-03-15 
categories: 
- Scala
---

Scala常用的构建方式是采用 SBT, 不过对于我们用熟了maven的Javaer来说，还是对maven的感觉更熟悉一点。 最近正好将 [来往](www.laiwang.com) 的一些代码进行Scala化，就整理了一下。

## Scala的仓库的指定

第一个要做的事情是需要告知maven，去哪里寻找我们需要的scala库以及Scala插件
需要在pom.xml 中添加上Scala的仓库地址. 

```
  <repositories>           
    <repository>           
      <id>scala-tools.org</id>           
      <name>Scala-tools Maven2 Repository</name>          
      <url>https://oss.sonatype.org/content/groups/scala-tools/</url>           
    </repository>           
  </repositories>       

  <pluginRepositories>           
    <pluginRepository>           
      <id>scala-tools.org</id>           
      <name>Scala-tools Maven2 Repository</name>           
      <url>https://oss.sonatype.org/content/groups/scala-tools/</url>           
    </pluginRepository>          
  </pluginRepositories>   
```

接下来就可以添加Scala的依赖啦， 比如: 
(scalatest是针对我们公司的仓库添加的版本)

```
<dependency>           
      <groupId>org.scala-lang</groupId>           
      <artifactId>scala-library</artifactId>           
      <version>2.10.0</version>           
    </dependency>   
    <dependency>           
      <groupId>org.scalatest</groupId>           
      <artifactId>scalatest_2.9.2</artifactId>           
      <version>1.7.2</version>
      <scope>test</scope>     
    </dependency>   
    <dependency>           
      <groupId>org.mockito</groupId>           
      <artifactId>mockito-all</artifactId>           
      <version>1.9.0</version>  
      <scope>test</scope>         
    </dependency>
```

## 添加Scala对应的插件 

主要是eclipse项目生成插件以及Scala的编译的插件。


```
<plugin>
              <groupId>org.scala-tools</groupId>  
              <artifactId>maven-scala-plugin</artifactId>  
              <version>2.15.0</version>  
              <executions>  
                <execution>  
                  <goals>  
                    <goal>compile</goal>  
                    <goal>testCompile</goal>  
                  </goals>  
                  <configuration>  
                    <args>  
                      <arg>-make:transitive</arg>  
                      <arg>-dependencyfile</arg>  
                      <arg>${project.build.directory}/.scala_dependencies</arg>  
                    </args>  
                  </configuration>  
                </execution>  
              </executions>  
</plugin>  

<plugin>
              <groupId>com.alibaba.org.apache.maven.plugins</groupId>
              <artifactId>maven-eclipse-plugin</artifactId>
              <configuration>
                <downloadSources>true</downloadSources>
                <downloadJavadocs>true</downloadJavadocs>
                <projectnatures>
                  <projectnature>org.scala-ide.sdt.core.scalanature</projectnature>
                  <projectnature>org.eclipse.jdt.core.javanature</projectnature>
                </projectnatures>
                <buildcommands>
                  <buildcommand>org.scala-ide.sdt.core.scalabuilder</buildcommand>
                </buildcommands>
                <classpathContainers>
                  <classpathContainer>org.scala-ide.sdt.launching.SCALA_CONTAINER</classpathContainer>
                  <classpathContainer>org.eclipse.jdt.launching.JRE_CONTAINER</classpathContainer>
                </classpathContainers>
                <sourceIncludes>
                  <sourceInclude>**/*.scala</sourceInclude>
                </sourceIncludes>
              </configuration>
</plugin>
```
最终你就可以利用maven常见的命令来进行Scala开发了。 
 如图：

![enter image description here][1]


  [1]: http://i01.lw.aliimg.com/tx/yj/yjtx_d0502729_505_410.520x400x75x2.jpg