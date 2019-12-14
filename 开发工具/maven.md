## 说明

## 目录

## maven排除依赖

> 在dependency内使用exclusions排除依赖，若排除的依赖和外部依赖的groupId前面相同，则不需要写，如下面的spark和log4j都是org.apache下的，所以log4j可以简写

```xml
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>2.4.4</version>
            <!--排除日志依赖，统一使用slf4j+logback-->
            <exclusions>
                <exclusion>
                    <groupId>log4j</groupId>
                    <artifactId>log4j</artifactId>
                </exclusion>
                <exclusion>
                    <groupId>org.slf4j</groupId>
                    <artifactId>slf4j-log4j12</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

