## 说明

## 目录

## DataSet创建和删除临时表

### 创建临时表

```java
// 创建它的SparkSession对象终止前有效
df.createOrReplaceTempView("tempViewName")  

// spark应用程序终止前有效
df.createOrReplaceGlobalTempView("tempViewName")  
```

### 删除临时表

```java
spark.catalog.dropTempView("tempViewName")
spark.catalog.dropGlobalTempView("tempViewName")
```

