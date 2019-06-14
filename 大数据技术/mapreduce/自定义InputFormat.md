1.若输入是文件，可以继承FileInputFormat

```java
package com.sugar.mapreduce.wordcount;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.InputSplit;
import org.apache.hadoop.mapreduce.JobContext;
import org.apache.hadoop.mapreduce.RecordReader;
import org.apache.hadoop.mapreduce.TaskAttemptContext;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.input.FileSplit;
import org.apache.hadoop.util.LineReader;

import java.io.IOException;

public class MyInputFormat extends FileInputFormat<LongWritable, Text> {

    /**
     * 判断一个文件是否可以切分，默认情况下，若一个文件的大小达到默认切片大小的1.1倍，就会被切片
     * 默认情况下，切片大小=blocksize(128M)
     *
     * @param context  上下文对象
     * @param filename 当前需要判断是否需要分片的文件路径
     * @return 是否切片
     */
    @Override
    protected boolean isSplitable(JobContext context, Path filename) {
        return super.isSplitable(context, filename);
    }

    /**
     * 返回一个对分片数据进行读取的对象
     *
     * @param inputSplit         分片
     * @param taskAttemptContext 上下文对象
     * @return
     * @throws IOException
     * @throws InterruptedException
     */
    @Override
    public RecordReader<LongWritable, Text> createRecordReader(InputSplit inputSplit, TaskAttemptContext taskAttemptContext) throws IOException, InterruptedException {
        MyRecordReader recordReader = new MyRecordReader();
        recordReader.initialize(inputSplit, taskAttemptContext);
        return recordReader;
    }

    public static class MyRecordReader extends RecordReader<LongWritable, Text> {
        /**
         * 按行读取的工具类
         */
        private LineReader lineReader;
        private LongWritable lineNumber = new LongWritable(0);
        private Text line;

        @Override
        public void initialize(InputSplit inputSplit, TaskAttemptContext context) throws IOException, InterruptedException {
            FileSplit split = (FileSplit) inputSplit;
            Configuration configuration = context.getConfiguration();
            Path path = split.getPath();
            FileSystem fileSystem = path.getFileSystem(configuration);  //获取文件系统
            FSDataInputStream inputStream = fileSystem.open(path);

            lineReader = new LineReader(inputStream, configuration);

        }

        /**
         * 判断是否有下一行输入
         *
         * @return
         * @throws IOException
         * @throws InterruptedException
         */
        @Override
        public boolean nextKeyValue() throws IOException, InterruptedException {
            int count = lineReader.readLine(line);
            //没有读取到下一行则返回false
            if (count == 0) {
                return false;
            }
            lineNumber.set(lineNumber.get() + 1);
            return true;
        }

        @Override
        public LongWritable getCurrentKey() throws IOException, InterruptedException {
            return lineNumber;
        }

        @Override
        public Text getCurrentValue() throws IOException, InterruptedException {
            return line;
        }

        @Override
        public float getProgress() throws IOException, InterruptedException {
            return 0;
        }

        @Override
        public void close() throws IOException {
            lineReader.close();
        }
    }
}

```

2.若输入是数据库或kafka之类的，可以继承InputFormat，其余的与方式一类似