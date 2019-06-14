1. combiner是MR程序中Mapper和Reducer之外的一种组件

2. combiner组件的父类就是Reducer

3. combiner和reducer的区别在于运行的位置：

   Combiner是在每一个maptask所在的节点运行

   Reducer是接收全局所有Mapper的输出结果；

4. combiner的意义就是对每一个maptask的输出进行局部汇总，以减小网络传输量

5. 自定义Combiner实现步骤：

   1. 自定义一个combiner继承Reducer，重写reduce方法

      ```java
      public class WordcountCombiner extends Reducer<Text, IntWritable, Text, IntWritable>{
      	@Override
      	protected void reduce(Text key, Iterable<IntWritable> values,
      			Context context) throws IOException, InterruptedException {
      		int count = 0;
      		for(IntWritable v :values){
      			count += v.get();
      		}
      		context.write(key, new IntWritable(count));
      	}
      }
      ```

   2. 在job中设置：

      ```java
      job.setCombinerClass(WordcountCombiner.class);
      ```

6. combiner能够应用的前提是不能影响最终的业务逻辑，而且，combiner的输出kv应该跟reducer的输入kv类型要对应起来

   