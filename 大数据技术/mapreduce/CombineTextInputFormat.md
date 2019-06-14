CombineTextInputFormat可以解决输入是大量小文件的问题，默认情况下mapreduce的切片机制是按照文件进行切片，若文件只有1kb,也会单独作为一个切片，这样就浪费资源，CombineTextInputFormat就可以解决这个问题，使用方式如下：

```java
job.setInputFormatClass(CombineTextInputFormat.class);
CombineTextInputFormat.setMinInputSplitSize(job,2097152);  //2M
CombineTextInputFormat.setMaxInputSplitSize(job,4194304);  //4M
```

进行上面的设置，CombineTextInputFormat就会将2M的文件逻辑上作为一个切片

