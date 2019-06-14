MultipleOutputs可以自定义输出文件的开头，默认的输出是以part-r-00001这种格式命名的，以wordcount为例，使用方式如下：

### 1.在主类中设置输出文件名，下面的代码设置了a,b,c 3中开头

```java
package com.sugar.mapreduce.wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
import org.apache.hadoop.mapreduce.lib.output.MultipleOutputs;
import org.apache.hadoop.mapreduce.lib.output.TextOutputFormat;

public class WordConutDriver {
    public static void main(String[] args) throws Exception {
        Configuration configuration = new Configuration();
        Job job = Job.getInstance(configuration);

        job.setJarByClass(WordConutDriver.class);

        job.setMapperClass(WordCountMapper.class);
        job.setReducerClass(WordCountReducer.class);
        job.setNumReduceTasks(3);

        job.setMapOutputKeyClass(Text.class);
        job.setMapOutputValueClass(IntWritable.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        MultipleOutputs.addNamedOutput(job,"a",TextOutputFormat.class,Text.class,IntWritable.class);
        MultipleOutputs.addNamedOutput(job,"b",TextOutputFormat.class,Text.class,IntWritable.class);
        MultipleOutputs.addNamedOutput(job,"c",TextOutputFormat.class,Text.class,IntWritable.class);
        FileInputFormat.setInputPaths(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        boolean flag = job.waitForCompletion(true);

        System.exit(flag ? 0 : 1);
    }
}

```

### 2.在reduce中使用

```java
package com.sugar.mapreduce.wordcount;

import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.output.MultipleOutputs;

import java.io.IOException;

public class WordCountReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    private MultipleOutputs outputs;
    private IntWritable outputValue = new IntWritable();
    private String outputPath;

    @Override
    protected void setup(Context context) throws IOException, InterruptedException {
        outputs = new MultipleOutputs(context);  //初始化
        outputPath = context.getConfiguration().get("mapred.output.dir");  //获取输出目录

    }

    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable count : values) {
            sum += count.get();
        }
//        context.write(key,new IntWritable(sum));
        outputValue.set(sum);
        //根据单次的包含的字母将其输出到不同的文件
        if (key.toString().contains("a")) {
            outputs.write("a",key,outputValue);
//            outputs.write(key, outputValue, outputPath+"/a/");
        } else if (key.toString().contains("b")) {
            outputs.write("b",key,outputValue);
//            outputs.write(key, outputValue, outputPath+"/b/");
        } else {
            outputs.write("c",key,outputValue);
//            outputs.write(key, outputValue, outputPath+"/c/");
        }
    }

    @Override
    protected void cleanup(Context context) throws IOException, InterruptedException {
        outputs.close();
    }
}
```

### 3.MultipleOutputs的另外2个write方法

```java
//将结果输出到outputPath+"/a/"下，文件命名方式使用默认
outputs.write(key, outputValue, outputPath+"/a/"); 
//将结果输出到outputPath+"/a/"下，文件以hello开头，
//hello必须在主类中添加才可以使用，MultipleOutputs.addNamedOutput(job,"hello",...)
outputs.write("hello",key, outputValue, outputPath+"/a/");
```