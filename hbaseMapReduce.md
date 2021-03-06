
#hbase MapReduce程序样例入门 
http://www.cnblogs.com/end/archive/2012/12/12/2814819.html

------------------------------------------------------------------------------------------------------------------------

##1. 标准的hbase作为数据读取源和输出源的样例

###1.1 创建配置信息和作业对象，设置作业的类

Configuration conf = HBaseConfiguration.create();
Job job = new Job(conf, "job name ");
job.setJarByClass(test.class);
Scan scan = new Scan();

//数据输入源是hbase的inputTable表,
//执行mapper.class进行map过程，
//输出的key/value类型是 ImmutableBytesWritable和Put类型，
//最后一个参数是作业对象。
//需要指出的是需要声明一个扫描读入对象scan，进行表扫描读取数 据用，其中scan可以配置参数，
TableMapReduceUtil.initTableMapperJob(inputTable, scan, mapper.class, Writable.class, Writable.class, job);

//数据输出目标是hbase的outputTable表，输出执行的reduce过程是reducer.class类，操作的作业目标是job。
TableMapReduceUtil.initTableReducerJob(outputTable, reducer.class, job);

job.waitForCompletion(true);



###1.2 mapper类：
//继承的是hbase中提供的TableMapper类，其实这个类也是继承的MapReduce类。
//后边跟的两个泛型参数指定类型是mapper输出的数据类型，该类型必须继承自Writable类，例如可能用到的put和delete就可以。
//需要注意的是要和initTableMapperJob 方法指定的数据类型一直。该过程会自动从指定hbase表内一行一行读取数据进行处理。
public class mapper extends TableMapper<KEYOUT, VALUEOUT> { 
	public void map(Writable key, Writable value, Context context) throws IOException, InterruptedException {
			//mapper逻辑
			context.write(key, value);
		} 
	}
}


###1.3 reducer类：
//reducer继承的是TableReducer类。后边指定三个泛型参数，前两个必须对应map过程的输出key/value类型，第三个必须是 put或者delete。
//write的时候可以把key写null，它是不必要的。这样reducer输出的数据会自动插入outputTable指定的 表内。
public class countUniteRedcuer extends TableReducer<KEYIN, VALUEIN, KEYOUT> {
	public void reduce(Text key, Iterable<VALUEIN> values, Context context)	throws IOException, InterruptedException {
		//reducer逻辑
		context.write(null, put or delete);
	}
}

------------------------------------------------------------------------------------------------------------------------

##2. 标准的hbase作为数据读取源和输出源的样例

###2.1 创建配置信息和作业对象，设置作业的类
Configuration conf = HBaseConfiguration.create();

Job job = new Job(conf, "job name ");
job.setJarByClass(test.class); 
job.setMapperClass(mapper.class);
job.setMapOutputKeyClass(Text.class);
job.setMapOutputValueClass(LongWritable.class);

FileInputFormat.setInputPaths(job, path); //指定FileInputFormat.setInputPaths的数据源路径，输出声明不变。便完成了从hdfs文本读取数据输出到hbase的命令声明过程。
TableMapReduceUtil.initTableReducerJob(tableName, reducer.class, job);


###2.2 mapper类：              
public class mapper extends Mapper<LongWritable,Writable,Writable,Writable> {
	public void map(LongWritable key, Text line, Context context) {
		 //mapper逻辑
		 context.write(k, one);
	}
}

###2.3 reducer类：
public class redcuer extends TableReducer<KEYIN, VALUEIN, KEYOUT> {
	public void reduce(Writable key, Iterable<Writable> values, Context context) throws IOException, InterruptedException {
		//reducer逻辑
		context.write(null, put or delete);
	}
}

------------------------------------------------------------------------------------------------------------------------

##3.从hbase中的表作为数据源读取，hdfs作为数据输出

###3.1 创建配置信息和作业对象，设置作业的类
Configuration conf = HBaseConfiguration.create();

Job job = new Job(conf, "job name ");
job.setJarByClass(test.class);
Scan scan = new Scan();
TableMapReduceUtil.initTableMapperJob(inputTable, scan, mapper.class, Writable.class, Writable.class, job);
job.setOutputKeyClass(Writable.class);
job.setOutputValueClass(Writable.class);

FileOutputFormat.setOutputPath(job, Path);
job.waitForCompletion(true);

###3.2 mapper类：
public class mapper extends	TableMapper<KEYOUT, VALUEOUT>{ 
	public void map(Writable key, Writable value, Context context) throws IOException, InterruptedException {
			//mapper逻辑
			context.write(key, value);
		}
	}
}
 
3.3 reducer类：
public class reducer extends Reducer<Writable,Writable,Writable,Writable>{ 
	public void reducer(Writable key, Writable value, Context context)
			throws IOException, InterruptedException {
			//reducer逻辑
			context.write(key, value);
		}
	}
}


------------------------------------------------------------------------------------------------------------------------
##4.最后说一下TableMapper和TableReducer的本质

其实这俩类就是为了简化一下书写代码，因为传入的4个泛型参数里都会有固定的参数类型，所以是Mapper和Reducer的简化版本，本质他们没有任何区别。

源码如下：
public abstract class TableMapper<KEYOUT, VALUEOUT> extends Mapper<ImmutableBytesWritable, Result, KEYOUT, VALUEOUT> {
}
 
public abstract class TableReducer<KEYIN, VALUEIN, KEYOUT> extends Reducer<KEYIN, VALUEIN, KEYOUT, Writable> {
}


