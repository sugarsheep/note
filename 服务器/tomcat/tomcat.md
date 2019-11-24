## 说明

## 目录

## tomcat引用外部项目

### 方式一

> 在Tomcat\conf\server.xml中配置
>
> ```xml
> <Host name="www.***.cn" debug="0" appBase="webapps"  
>             unpackWARs="true" autoDeploy="true"  
>             xmlValidation="false" xmlNamespaceAware="false">  
> <Context docBase="D:\\yaodaqing\\workspace\\fc\\WebRoot\\" path=""   
> privileged="true" reloadable="true" caseSensitive="false" debug="0" crossContext="true">  
> </Context>  
> </Host>  
> ```
>
> 配置了此信息，tomcat即可直接读取正在开发的项目。  

### 方式二

> Tomcat\conf\Catalina\localhost
>
> localhost表示具体的域名，也就是<Host name="www.***.cn"这里配置的名字。  
> 在此文件夹里加入一个项目名字.xml的文件。在里面添上：  
> 文件名：fc.xml（fc表示项目）,path为访问路径，一般配置为项目名
> 内容如下：  
>
> ```xml
> <Context docBase="D:\\yaodaqing\\workspace\\fc\\WebRoot\\" path="/fc"   
> privileged="true" reloadable="true" caseSensitive="false" debug="0" crossContext="true"> 
> ```

