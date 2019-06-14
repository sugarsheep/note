> 解决方案：将字节流包装成字符流，并使用与需要读取的文件相同的字符集读取文件
>

```java
Properties prop=new Properties();         
prop.load(new InputStreamReader(Client.class.getClassLoader().getResourceAsStream("config.properties"), "UTF-8"));   
```

