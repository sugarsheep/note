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

## spark java-api使用

> 注意要使用和安装的scala对应版本兼容的spark
>
> ![1576319230850](images/1576319230850.png)

### 创建java maven工程

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
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.slf4j/log4j-over-slf4j -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
            <version>1.7.24</version>
        </dependency>
```

### spark报任务没有序列化问题

> spark会把当前作为一个任务，所以该类需要实现java.io.Serializable接口，并且，若程序内部使用了主类的成员变量，则该成员变量也不要是可以序列化的，否则会报错

### rdd api解析

#### join

##### 说明

> - 对2个rdd进行join操作，将2个rdd中key相同的数据聚合到一起，形成（k,<v1,v2>）的二元组
>
> - 将一组数据转化为RDD后，分别创造出两个PairRDD，然后再对两个PairRDD进行归约（即合并相同Key对应的Value）
>
> - 若rdd1有一条数据（1,1），rdd2有2条数据（1，A）,(1,B),则会产生<1,<1,A>>,<1,<1,B>>，相当于做**笛卡尔积**，产生1*2=2条结果
>
> - 如图所示
>
>   ![1576328659231](images/1576328659231.png)

##### 代码

```java
@Test
    public void testJoin() {
        SparkConf conf = new SparkConf().setAppName("SparkRDD").setMaster("local");
        JavaSparkContext sc = new JavaSparkContext(conf);
        List<Integer> data = Arrays.asList(1, 2, 3, 4, 5);
        JavaRDD<Integer> rdd = sc.parallelize(data);

        //FirstRDD
        JavaPairRDD<Integer, Integer> firstRDD = rdd.mapToPair(new PairFunction<Integer, Integer, Integer>() {
            @Override
            public Tuple2<Integer, Integer> call(Integer num) throws Exception {
                return new Tuple2<>(num, num * num);
            }
        });

        //SecondRDD
        JavaPairRDD<Integer, String> secondRDD = rdd.mapToPair(new PairFunction<Integer, Integer, String>() {
            @Override
            public Tuple2<Integer, String> call(Integer num) throws Exception {
                return new Tuple2<>(num, String.valueOf((char) (64 + num * num)));
            }
        });

        JavaPairRDD<Integer, Tuple2<Integer, String>> joinRDD = firstRDD.join(secondRDD);

        JavaRDD<String> res = joinRDD.map(new Function<Tuple2<Integer, Tuple2<Integer, String>>, String>() {
            @Override
            public String call(Tuple2<Integer, Tuple2<Integer, String>> integerTuple2Tuple2) throws Exception {
                int key = integerTuple2Tuple2._1();
                int value1 = integerTuple2Tuple2._2()._1();
                String value2 = integerTuple2Tuple2._2()._2();
                return "<" + key + ",<" + value1 + "," + value2 + ">>";
            }
        });

        List<String> resList = res.collect();
        for (String str : resList) {
            System.out.println(str);
        }

        sc.close();
    }
```

#### cogroup

##### 说明

> - 有两个元组Tuple的集合A与B,先对A组集合中key相同的value进行聚合,然后对B组集合中key相同的value进行聚合,之后对A组与B组进行"join"操作
> - 即2个rdd（元素是元组）先各自内部根据key进行聚合，然后将聚合的结果进行join操作

##### 代码

```java
@Test
    public void testCogroup(){
        SparkConf conf=new SparkConf().setAppName("spark WordCount!").setMaster("local");
        JavaSparkContext sContext=new JavaSparkContext(conf);
        List<Tuple2<Integer,String>> namesList=Arrays.asList(
                new Tuple2<Integer, String>(1,"Spark"),
                new Tuple2<Integer, String>(3,"Tachyon"),
                new Tuple2<Integer, String>(4,"Sqoop"),
                new Tuple2<Integer, String>(2,"Hadoop"),
                new Tuple2<Integer, String>(2,"Hadoop2")
        );

        List<Tuple2<Integer,Integer>> scoresList=Arrays.asList(
                new Tuple2<Integer, Integer>(1,100),
                new Tuple2<Integer, Integer>(3,70),
                new Tuple2<Integer, Integer>(3,77),
                new Tuple2<Integer, Integer>(2,90),
                new Tuple2<Integer, Integer>(2,80)
        );
        JavaPairRDD<Integer, String> names=sContext.parallelizePairs(namesList);
        JavaPairRDD<Integer, Integer> scores=sContext.parallelizePairs(scoresList);
        /**
         * <Integer> JavaPairRDD<Integer, Tuple2<Iterable<String>, Iterable<Integer>>>
         * org.apache.spark.api.java.JavaPairRDD.cogroup(JavaPairRDD<Integer, Integer> other)
         */
        JavaPairRDD<Integer, Tuple2<Iterable<String>, Iterable<Integer>>> nameScores=names.cogroup(scores);

        nameScores.foreach(new VoidFunction<Tuple2<Integer, Tuple2<Iterable<String>, Iterable<Integer>>>>() {
            private static final long serialVersionUID = 1L;
            int i=1;
            @Override
            public void call(
                    Tuple2<Integer, Tuple2<Iterable<String>, Iterable<Integer>>> t)
                    throws Exception {
                String string="ID:"+t._1+" , "+"Name:"+t._2._1+" , "+"Score:"+t._2._2;
                string+="     count:"+i;
                System.out.println(string);
                i++;
            }
        });

        sContext.close();
    }
```

#### groupByKey

##### 说明

> - 即根据数据的key进行聚合，所以需要rdd的元素是元组
>
> - 在spark中，我们知道一切的操作都是基于RDD的。在使用中，RDD有一种非常特殊也是非常实用的format——pair RDD，即RDD的每一行是（key, value）的格式。这种格式很像Python的字典类型，便于针对key进行一些处理
>
> - groupByKey也是对每个key进行操作，但只生成一个sequence。需要特别注意“Note”中的话，它告诉我们：如果需要对sequence进行aggregation操作（注意，groupByKey本身不能自定义操作函数），那么，选择reduceByKey/aggregateByKey更好。这是因为groupByKey不能自定义函数，我们需要先用groupByKey生成RDD，然后才能对此RDD通过map进行自定义函数操作
>
> - 当采用groupByKey时，由于它不接收函数，spark只能先将所有的键值对(key-value pair)都移动，这样的后果是集群节点之间的开销很大，导致传输延时。整个过程如下
>
>   ![1576332782687](images/1576332782687.png)
>
> - 如果仅仅是group处理，那么以下函数应该优先于 groupByKey ：
>
>   > （1）、combineByKey 组合数据，但是组合之后的数据类型与输入时值的类型不一样。
>   > （2）、foldByKey合并每一个 key 的所有值，在级联函数和“零值”中使用。
>
> - 1

##### 代码

```java
SparkConf sparkConf = new SparkConf();  
        sparkConf.setAppName("Spark_GroupByKey_Sample");  
        sparkConf.setMaster("local");  
  
        JavaSparkContext context = new JavaSparkContext(sparkConf);  
  
        List<Integer> data = Arrays.asList(1,1,2,2,1);  
        JavaRDD<Integer> distData= context.parallelize(data);  
  
        JavaPairRDD<Integer, Integer> firstRDD = distData.mapToPair(new PairFunction<Integer, Integer, Integer>() {  
            @Override  
            public Tuple2<Integer, Integer> call(Integer integer) throws Exception {  
                return new Tuple2(integer, integer*integer);  
            }  
        });  
  
		//得到<1,[1,1,1]>,<2,[4,4,]>
        JavaPairRDD<Integer, Iterable<Integer>> secondRDD = firstRDD.groupByKey();  
  
		//得到<1,1 1 1>,<2,4 4>
        List<Tuple2<Integer, String>> reslist = secondRDD.map(new Function<Tuple2<Integer, Iterable<Integer>>, Tuple2<Integer, String>>() {  
            @Override  
            public Tuple2<Integer, String> call(Tuple2<Integer, Iterable<Integer>> integerIterableTuple2) throws Exception {  
                int key = integerIterableTuple2._1();  
                StringBuffer sb = new StringBuffer();  
                Iterable<Integer> iter = integerIterableTuple2._2();  
                for (Integer integer : iter) {  
                        sb.append(integer).append(" ");  
                }  
                return new Tuple2(key, sb.toString().trim());  
            }  
        }).collect();  
  
  
        for(Tuple2<Integer, String> str : reslist) {  
            System.out.println(str._1() + "\t" + str._2() );  
        }  
        context.stop(); 
```

#### map

> 将一个元素映射成另一个元素，1对1映射

#### flatmap

> 1对多映射

#### mapToPair

> 将映射映射成一个元组

#### mapPartitions

##### 说明

> - mapPartitions函数会对每个分区依次调用分区函数处理，然后将处理的结果(若干个Iterator)生成新的RDDs。mapPartitions与map类似，但是如果在映射的过程中需要频繁创建额外的对象，使用mapPartitions要比map高效的过。比如，将RDD中的所有数据通过JDBC连接写入数据库，如果使用map函数，可能要为每一个元素都创建一个connection，这样开销很大，如果使用mapPartitions，那么只需要针对每一个分区建立一个connection
> - **两者的主要区别是调用的粒度不一样：map的输入变换函数是应用于RDD中每个元素，而mapPartitions的输入函数是应用于每个分区**
> - 假设一个rdd有10个元素，分成3个分区。如果使用map方法，map中的输入函数会被调用10次；而使用mapPartitions方法的话，其输入函数会只会被调用3次，每个分区调用1次

##### 代码

```java
SparkConf conf = new SparkConf().setAppName("SparkRDD").setMaster("local");
        JavaSparkContext sc = new JavaSparkContext(conf);
        List<Integer> data = Arrays.asList(1, 2, 4, 3, 5, 6, 7);
        //RDD有两个分区
        JavaRDD<Integer> javaRDD = sc.parallelize(data, 2);
        //计算每个分区的合计
        JavaRDD<Integer> mapPartitionsRDD = javaRDD.mapPartitions(new FlatMapFunction<Iterator<Integer>, Integer>() {
            @Override
            public Iterator<Integer> call(Iterator<Integer> integerIterator) throws Exception {
                int isum = 0;
                while (integerIterator.hasNext())
                    isum += integerIterator.next();
                LinkedList<Integer> linkedList = new LinkedList<Integer>();
                linkedList.add(isum);
                return linkedList.iterator();
            }
        });

        System.out.println("mapPartitionsRDD~~~~~~~~~~~~~~~~~~~~~~" + mapPartitionsRDD.collect());
        sc.close();
```

#### mapPartitionsWithIndex

##### 说明

> - mapPartitionsWithIndex与mapPartitions基本相同，只是在处理函数的参数是一个**二元元组**，元组的第一个元素是当前处理的**分区的index**，元组的第二个元素是当前处理的分区元素组成的Iterator
> - 分区号从0开始

##### 代码

```java
List<Integer> data = Arrays.asList(1, 2, 4, 3, 5, 6, 7);
//RDD有两个分区
JavaRDD<Integer> javaRDD = javaSparkContext.parallelize(data,2);
//分区index、元素值、元素编号输出
JavaRDD<String> mapPartitionsWithIndexRDD = javaRDD.mapPartitionsWithIndex(new Function2<Integer, Iterator<Integer>, Iterator<String>>() {
 @Override 
public Iterator<String> call(Integer v1, Iterator<Integer> v2) throws Exception {        
  LinkedList<String> linkedList = new LinkedList<String>();        
  int i = 0;        
  while (v2.hasNext())            
   linkedList.add(Integer.toString(v1) + "|" + v2.next().toString() + Integer.toString(i++));        
  return linkedList.iterator();    
  }
},false);

System.out.println("mapPartitionsWithIndexRDD~~~~~~~~~~~~~~~~~~~~~~" + mapPartitionsWithIndexRDD.collect());
```

#### sortBy

##### 说明

> - sortBy根据给定的f函数将RDD中的元素进行排序

##### 代码

```java
List<Integer> data = Arrays.asList(5, 1, 1, 4, 4, 2, 2);
JavaRDD<Integer> javaRDD = javaSparkContext.parallelize(data, 3);
final Random random = new Random(100);
//对RDD进行转换，每个元素有两部分组成
JavaRDD<String> javaRDD1 = javaRDD.map(new Function<Integer, String>() {    
  @Override    
  public String call(Integer v1) throws Exception {        
    return v1.toString() + "_" + random.nextInt(100);    
  }
});
System.out.println(javaRDD1.collect());
//按RDD中每个元素的第二部分进行排序
JavaRDD<String> resultRDD = javaRDD1.sortBy(new Function<String, Object>() {    
  @Override    
  public Object call(String v1) throws Exception {        
    return v1.split("_")[1];    
  }
},false,3);
System.out.println("result--------------" + resultRDD.collect());
```

#### takeOrdered

##### 说明

> - takeOrdered函数用于从RDD中，按照默认（升序）或指定排序规则，返回前num个元素。

##### 代码

```java
    public static class TakeOrderedComparator implements Serializable, Comparator<Integer> {
        @Override
        public int compare(Integer o1, Integer o2) {
            return -o1.compareTo(o2);
        }
    }

    @Test
    public void testTakeOrdered() {
        SparkConf conf = new SparkConf().setAppName("SparkRDD").setMaster("local");
        JavaSparkContext sc = new JavaSparkContext(conf);
        List<Integer> data = Arrays.asList(5, 1, 0, 4, 4, 2, 2);
        JavaRDD<Integer> javaRDD = sc.parallelize(data, 3);
        System.out.println("takeOrdered-----1-------------" + javaRDD.takeOrdered(2));
        List<Integer> list = javaRDD.takeOrdered(2, new TakeOrderedComparator());
        System.out.println("takeOrdered----2--------------" + list);
        sc.close();
    }
```

#### takeSample

##### 说明

> - takeSample函数返回一个数组，在数据集中随机采样 num 个元素组成
> - takeSample函数类似于[sample](https://www.jianshu.com/p/abe1755220b2)函数，该函数接受三个参数，第一个参数withReplacement ，表示采样是否放回，true表示有放回的采样，false表示无放回采样；第二个参数num，表示返回的采样数据的个数，这个也是takeSample函数和sample函数的区别；第三个参数seed，表示用于指定的随机数生成器种子。
> - 另外，takeSample函数先是计算fraction，也就是采样比例，然后调用sample函数进行采样，并对采样后的数据进行collect()，最后调用take函数返回num个元素。注意，如果采样个数大于RDD的元素个数，且选择的无放回采样，则返回原始rdd的数据。

##### 代码

```java
List<Integer> data = Arrays.asList(5, 1, 0, 4, 4, 2, 2);
JavaRDD<Integer> javaRDD = javaSparkContext.parallelize(data, 3);
System.out.println("takeSample-----1-------------" + javaRDD.takeSample(true,2));
System.out.println("takeSample-----2-------------" + javaRDD.takeSample(true,2,100));
//返回20个元素
System.out.println("takeSample-----3-------------" + javaRDD.takeSample(true,20,100));
//无放回采样，采样个数大于数据集个数，返回7个元素
System.out.println("takeSample-----4-------------" + javaRDD.takeSample(false,20,100));
```

#### distinct

##### 说明

> - 对rdd元素进行去重
> - 原理：distinct() 功能是 deduplicate RDD 中的所有的重复数据。由于重复数据可能分散在不同的 partition 里面，因此需要 shuffle 来进行 aggregate 后再去重。然而，shuffle 要求数据类型是 <K, V> 。如果原始数据只有 Key（比如例子中 record 只有一个整数），那么需要补充成 <K, null> 。这个补充过程由 map() 操作完成，生成 MappedRDD。然后调用上面的 reduceByKey() 来进行 shuffle，在 map 端进行 combine，然后 reduce 进一步去重，生成 MapPartitionsRDD。最后，将 <K, null> 还原成 K，仍然由 map() 完成，生成 MappedRDD

##### 代码

```java
List<Integer> data = Arrays.asList(1, 2, 4, 3, 5, 6, 7, 1, 2);
JavaRDD<Integer> javaRDD = javaSparkContext.parallelize(data);

JavaRDD<Integer> distinctRDD1 = javaRDD.distinct();
System.out.println(distinctRDD1.collect());
JavaRDD<Integer> distinctRDD2 = javaRDD.distinct(2);
System.out.println(distinctRDD2.collect());
```

#### cartesian

##### 说明

> - **Cartesian 对两个 RDD 做笛卡尔集，生成的 CartesianRDD 中 partition 个数 = partitionNum(RDD a) * partitionNum(RDD b)**。
> - 从getDependencies分析可知，这里的依赖关系与前面的不太一样，CartesianRDD中每个partition依赖两个parent RDD，而且其中每个 partition 完全依赖(NarrowDependency) RDD a 中一个 partition，同时又完全依赖(NarrowDependency) RDD b 中另一个 partition。具体如下CartesianRDD 中的 partiton i 依赖于 (RDD a).List(i / numPartitionsInRDDb) 和 (RDD b).List(i % numPartitionsInRDDb)

##### 代码

```java
List<Integer> data = Arrays.asList(1, 2, 4, 3, 5, 6, 7);
JavaRDD<Integer> javaRDD = javaSparkContext.parallelize(data);

JavaPairRDD<Integer,Integer> cartesianRDD = javaRDD.cartesian(javaRDD); System.out.println(cartesianRDD.collect());
```

