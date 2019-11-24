## 说明

## 目录

## idea技巧

### 解决spring的@Autowired报错问题

![1569164678682](images/1569164678682.png)

## 配置项目域名

> - host文件路径：C:\Windows\System32\drivers\etc
>
> - 修改host文件
>
> ```
> 127.0.0.1 user.gmail.cn
> ```
>
> - 刷新host文件，使其生效：ipconfig /flushdns
> - .com后缀无法生效

## 整合通用mapper

### 添加maven依赖

```xml
        <dependency>
            <groupId>tk.mybatis</groupId>
            <artifactId>mapper-spring-boot-starter</artifactId>
            <version>2.0.2</version>
            <exclusions>
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-jdbc</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
```

### 使用方法

> - 配置了通用mapper之后，就不需要使用xml映射文件写sql进行映射，只需要使mapper接口继承通用Mapper的Mapper类即可
>
> - 通用mapper提供了单表大量的操作接口，足以完成简单功能
>
> - 若需要指定javabean的主键，需要使用@id注解声明，若想要实现在插入数据的时候返回自增的主键，需要再配置@GeneratedValue
>
>   ```java
>     @Id
>     @GeneratedValue(strategy = GenerationType.IDENTITY)
>     private long id;
>   ```
>
> - 修改启动类mapper扫描器，使用通用mapper的扫描器
>
>   ```java
>   import org.springframework.boot.SpringApplication;
>   import org.springframework.boot.autoconfigure.SpringBootApplication;
>   import tk.mybatis.spring.annotation.MapperScan;
>   
>   @SpringBootApplication
>   @MapperScan(basePackages = "com.sugar.gmail.user.mapper")
>   public class GmailUserApplication {
>   
>       public static void main(String[] args) {
>           SpringApplication.run(GmailUserApplication.class, args);
>       }
>   
>   }
>   ```

### 通用mapper条件查询

#### 方式1

```java
UmsMemberReceiveAddress umsMemberReceiveAddress = new UmsMemberReceiveAddress();
umsMemberReceiveAddress.setMemberId(Long.valueOf(userId));
return umsMemberReceiveAddressMapper.select(umsMemberReceiveAddress);
```

#### 方式2

```java
Example example = new Example(UmsMemberReceiveAddress.class);
example.createCriteria().andEqualTo("memberId",userId);
return umsMemberReceiveAddressMapper.selectByExample(example);
```

### 注意

> - Javabean的属性不要使用基本类型，要使用包装类型

## dubbo配置注意事项

- 提供者方需要将@Service修改为dubbo的@Service

- 消费者方将@Autowired修改为使用@Reference引用

- 提供者方传输的javabean需要实现序列化接口

- dubbo的消费者在3秒内会每间隔一秒访问服务端，默认1秒超时，3次访问之后就会抛出超时异常，开发时，可以将超时时间设置长一点

  ```properties
  #设置服务调用超时时间
  spring.dubbo.consumer.timeout=600000
  #设置是否检查服务是否存在
  spring.dubbo.consumer.check=false
  ```

- 11

## 启动前端项目

> npm run dev

## 解决跨域

在controller类上添加@CrossOrigin注解

## controller接受json数据

> ```
> 使用@RequestBody注解标识参数即可
> ```

## spu

### spu图片上传策略

> - 在用户选择图片的时候就上传图片到分布式文件服务器，保存时传输图片的元数据信息和url即可
> - 若用户只是选择了图片二没有进行保存就退出了，服务器可以采用定期清理的策略，清除没有引用的图片
> - 图片的储存这里使用fastdfs，它是一个分布式文件系统

### spu销售属性

> - 商品的**平台属性**属于电商网站后台进行管理（针对整个平台）
> - 商品的**销售属性**属于在电商网站上发布商品的商家管理（针对某一个商品）

