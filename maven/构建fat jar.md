# 构建fat jar

#### maven 里引入

```java
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>2.4.3</version>
    <configuration>
    </configuration>
    <executions>
     <execution>
      <phase>package</phase>
      <goals>
       <goal>shade</goal>
      </goals>
      <configuration>
       <shadedArtifactAttached>true</shadedArtifactAttached>
       <shadedClassifierName>biz</shadedClassifierName>
      </configuration>

     </execution>
    </executions>
   </plugin>
```

