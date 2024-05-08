EXP1 :
1. Download Hadoop and Java JDK
2. Javac -version
3. Extract Hadoop.tar.gz or Hadoop.zip
4. Set path, HADOOP_HOME (bin path)
5. Set path, JAVA_HOME (bin path)
6. Config edit, /etc/Hadoop/, core-site.xml, mapred-site.xml, hdfs-site.xml, yarn-site.xml
7. Create data inside Hadoop, then "datanode" and "namenode" inside data
8. Edit, /etc/Hadoop/hadoop-env.cmd, set JAVA_HOME=C:/Java/jdk
9. Download Hadoop configuration.zip, replace it with C:\hadoop\bin
10. hdfs namenode -format
11. C:\hadoop\sbin> start-all.cmd
12. Open localhost:8088 and 50070

core-site.xml:
<property>
	<name>fs.defaultFS</name>
	<value>hdfs://localhost:9000</value>
</property>

mapred-site.xml:
<property>
	<name>mapreduce.framework.name</name>
	<value>yarn</value>
</property>

yarn.xml:
<property>
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
</property>
<property>
	<name>yarn.nodemanager.auxservics.mapreduce.shuffle.class</name>
	<value>org.apache.hadoop.mapred.ShuffleHandler</value>
</property>

hdfs-site.xml:
<property>
	<name>dfs.replication</name>
	<value>1</value>
</property>
<property>
	<name>dfs.namenode.name.dir</name>
	<value>C:\hadoop\data\namenode</value>
</property>
<property>
	<name>dfs.datanode.data.dir</name>
	<value>C:\hadoop\data\datanode</value>
</property>







EXP 2:
Adding files and dir to HDFS:
1. Hadoop fs -mkdir /<user>/chuck
2. Hadoop fs -put example.txt /user/chuck

Retrieving Files from HDFS:
1. Hadoop fs -cat example.txt

Deleting files from HDFS:
1. Hadoop fs -rm example.txt

Creating a dir in HDFS:
1. hdfs dfs -mkdir /lendicse

Adding a dir to HDFS:
1. hdfs dfs -put lendi_english /





YOUR CODE EXP 3:

import java.io.IOException;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class MatrixMultiplication {

    public static class MatrixMapper extends Mapper<Object, Text, Text, Text> {

        private Text outKey = new Text();
        private Text outValue = new Text();

        public void map(Object key, Text value, Context context) throws IOException, InterruptedException {
            // Tokenize input matrix
            String[] parts = value.toString().split("\\s+");
            String matrixType = parts[0];
            String rowCol = parts[1];
            String val = parts[2];

            // Emit key-value pairs
            for (int i = 0; i < (matrixType.equals("M") ? Integer.parseInt(rowCol) : Integer.parseInt(rowCol)); i++) {
                outKey.set((matrixType.equals("M") ? parts[1] + "," + i : i + "," + parts[1]));
                outValue.set(matrixType + "," + rowCol + "," + val);
                context.write(outKey, outValue);
            }
        }
    }

    public static class MatrixReducer extends Reducer<Text, Text, Text, Text> {

        private Text outValue = new Text();

        public void reduce(Text key, Iterable<Text> values, Context context)
                throws IOException, InterruptedException {

            int sum = 0;
            int[] valM = new int[10];
            int[] valN = new int[10];
            int k = 0;

            // Multiply and sum
            for (Text val : values) {
                String[] parts = val.toString().split(",");
                if (parts[0].equals("M")) {
                    valM[k] = Integer.parseInt(parts[2]);
                } else {
                    valN[k] = Integer.parseInt(parts[2]);
                }
                k++;
            }

            // Calculate sum
            for (int i = 0; i < k; i++) {
                sum += valM[i] * valN[i];
            }

            // Emit result
            outValue.set(Integer.toString(sum));
            context.write(key, outValue);
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "matrix multiplication");

        job.setJarByClass(MatrixMultiplication.class);
        job.setMapperClass(MatrixMapper.class);
        job.setReducerClass(MatrixReducer.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);

        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));

        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}


JOSA CODE:

import java.io.IOException;
import java.util.StringTokenizer;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.io.*;
import org.apache.hadoop.mapreduce.*;

public class MatrixMultiplication {

    public static class Map extends Mapper<LongWritable, Text, Text, Text> {

        public void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
            String line = value.toString();
            String[] tokens = line.split(",");
            String matrixName = tokens[0];
            int row = Integer.parseInt(tokens[1]);
            int col = Integer.parseInt(tokens[2]);
            int val = Integer.parseInt(tokens[3]);

            if (matrixName.equals("A")) {
                // Emit intermediate key-value pairs for matrix A
                for (int k = 0; k < col; k++) {
                    context.write(new Text(row + "," + k), new Text(matrixName + "," + col + "," + val));
                }
            } else {
                // Emit intermediate key-value pairs for matrix B
                for (int i = 0; i < row; i++) {
                    context.write(new Text(i + "," + col), new Text(matrixName + "," + row + "," + val));
                }
            }
        }
    }

    public static class Reduce extends Reducer<Text, Text, Text, IntWritable> {

        public void reduce(Text key, Iterable<Text> values, Context context) throws IOException, InterruptedException {
            String[] keyParts = key.toString().split(",");
            int row = Integer.parseInt(keyParts[0]);
            int col = Integer.parseInt(keyParts[1]);

            int[] rowA = new int[100]; // Assuming max size for matrix A row
            int[] colB = new int[100]; // Assuming max size for matrix B column
            int sum = 0;

            for (Text val : values) {
                String[] valParts = val.toString().split(",");
                String matrixName = valParts[0];
                int dimension = Integer.parseInt(valParts[1]);
                int value = Integer.parseInt(valParts[2]);

                if (matrixName.equals("A")) {
                    rowA[dimension] = value;
                } else {
                    colB[dimension] = value;
                }
            }

            for (int i = 0; i < rowA.length; i++) {
                sum += rowA[i] * colB[i];
            }

            context.write(new Text(row + "," + col), new IntWritable(sum));
        }
    }

    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "matrix multiplication");
        job.setJarByClass(MatrixMultiplication.class);
        job.setMapperClass(Map.class);
        job.setReducerClass(Reduce.class);
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(Text.class);
        job.setInputFormatClass(TextInputFormat.class);
        job.setOutputFormatClass(TextOutputFormat.class);
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}

input:
A,0,0,1
A,0,1,2
B,0,0,3
B,1,0,4

output:
[1, 2] in matrix A and [3, 4] in matrix B.





EXP 4:


import java.io.IOException;

import java.util.StringTokenizer;

import org.apache.hadoop.io.*;

import org.apache.hadoop. .apache.hadoop.mapreduce.*;

public class WordCount {

public static class TokenizerMapper extends Mapper<Object, Text, Text, IntWritable> {

private final static IntWritable one = new IntWritable(1);

private Text word = new Text();

public void map (Object key, Text value, Context context) throws IOException, InterruptedException {

StringTokenizer itr = new StringTokenizer(value.toString());

while (itr.hasMoreTokens()) {

word.set(itr.nextToken());

context.write(word, one);

}

}

}

public static class IntSumReducer extends Reducer<Text, IntWritable, Text, IntWritable> {

private IntWritable result = new IntWritable();

public void reduce (Text key, Iterable<IntWritable> values, Context context)




throws IOException, InterruptedException {

int sum = 0;

for (IntWritable val values) {

sum += val.get();

}

result.set(sum);

context.write(key, result);
}

}

public static void main(String[] args) throws Exception {

Job job Job.getInstance();

job.setJarByClass (WordCount.class);

job.setMapperClass (TokenizerMapper.class);

job.setCombinerClass (IntSumReducer.class);

job.setReducerClass (IntSumReducer.class);

job.setOutputKeyClass (Text.class);

job.setOutputValueClass (IntWritable.class);

FileInputFormat.addInputPath (job, new Path(args[0]));

FileOutputFormat.setOutputPath (job, new Path(args[1]));

System.exit(job.waitForCompletion(true) ? 0:1);

}

}

1. Compile and package into jar file.
2. save code. WordCount.java
3. run this Simply with command line:
javac WordCount.java
jar cf WordCount.jar WordCount*.class
4. This will give jar file, now lets run it:
hadoop jar WordCount.jar WordCount <input_directory> <output_directory>


EXP 6:

*installation of hbase*
Step 1:
Download hbase in the below site -
hbase.apache.org/downloads.html 
Download version 2.2.5 (bin)
 |
V http://apachemirror.wuchna.com/hbase/2.2.5/hbase-2.2.5-bin.tar.gz

*Step2*

Unzip the downloaded file.

Inside *h-base 2.2.5* create two folders: hbase and zookeeper

*Step 3- setting changes*

- Open *bin* folder in *h-base 2.2.5*.
- inside *bin* open the file *h-base* which is a command script.
- search for the line:
       Set java_arguments=%HEAP_SETTINGS% %SHBASE_OPTS%-classpath "CLASSPATH" CLASS Shbase-command-arguments

- Remove %HEAP_SETTINGS% and save the file

*Step4*

- Open *config* in *h-base 2.2.5*
- modify two files
- 1st *h-base env* file
- inside add thesep parameters:
     set JAVA HOME-JAVA HOME

set HBASE CLASSPATH=HBASE_HOME%\lib\client-facing-thirdparty\*

set HBASE HEAPSIZE-8000

set HBASE_OPTS="-XX:+UseConcMarkSweepGC" "-Djava.net.preferIPv4Stack=true"

set SERVER_GC_OPTS="-verbose:gc" "-XX:+PrintGCDetails" "-XX:+PrintGCDateStamps" SHBASE_GC_OPTS

set HBASE USE GC LOGFILE=true

set HBASE JMX BASE="-Dcom.sun.management.jmxremote.ssl=false" "-Dcom.sun.management.jmxremote.authenticate=false"

set HBASE MASTER_OPTS=SHBASE JMX_BASES "-Dcom.sun.management.jmxremote.port=10101"

set HBASE_REGIONSERVER_OPTS=HBASE_JMX_BASE "-Dcom.sun.management.jmxremote.port=10102" set HBASE THRIFT_OPTS=SHBASE_JMX_BASES "-Dcom.sun.management.jmxremote.port=10103"

set HBASE ZOOKEEPER_OPTS=SHBASE JMX BASE -Dcom.sun.management.jmxremote.port=10104"

set HBASE_REGIONSERVERS=HBASE_HOME%\conf\regionservers

set HBASE LOG DIR-HBASE HOME%\logs

set HBASE IDENT STRING=%USERNAME

set HBASE MANAGES_ZK=true

- save the file

- next open *h-base site*
- add :

<property>

<name>hbase.cluster.distributed</name>

<value>false</value>

</property>

<property>

<name>hbase.tmp.dir</name>

<value>./tmp</value>

</property>

<property>

<name>hbase.unsafe.stream.capability.enforce</name>

<value>false</value>

</property>

*Root directory:*

<property>

<name>hbase.rootdir</name>

<value>file:///C:/Users/as5272/Documents/hbase-2.2.5/hbase</value>

</property>

*Data directory:*

<name>hbase.zookeeper.property.dataDir</name>

<value>/C:/Users/as5272/Documents/hbase-2.2.5/zookeeper</value>

</property>

*Zookeeper's quorem*

property>

<name> hbase.zookeeper.quorum</name>

<value>localhost</value>

</property>

- save the file


*Step 5- environment variables*

- search for  *edit system environment variables*  in start
- in that open *environment variables*
- in user variables
- add variable name : *HBASE_HOME*
- variable value : add the file path
- inside path variable add h-base bin path

*Step 6*
- Open command prompt
- cd h-base bin path
- to start h-base server:
     *start-hbase.cmd*
- to check installation:
     *hbase version*
    Outcome: HBase 2.2.5
- to check what are all the server running in the command:
    *jps*
    Outcome:
     2663184 Jps
     2632692 SparkSubmit
     2664036 HMaster
     2464136 SparkSubmit


Eg:
- try creating small table:
 
- *hbase shell*
- create 'demo1' , 'check'
- to check how may tables are there command: *list* ( will list the avl tables)
- to insert records:
    put 'demo1' , 'row1' , 'check:learntospark' , 'subcribe'

Where :
Demo1 - table name
Row1 - which row
KeyValue - check:learntospark
Value - subcribe


- to check row added use command:
      *scan 'demo1'*
Outcome:





Exp 7: Importing and exporting data from various databases:

Importing data from MySQL to HDFS:
1. Login to MySQL:
MySQL -u root -pcloudera
2. Create a database and table and insert data:
create database IT;
create table IT.IT(name varchar(20), reg_no int);
insert into IT values("Josan",34);
3. Create database and table in hive where data should be imported
4. Run below command in Hadoop:
sqoop import --connect \
jdbc:mysql://127.0.0.1:3306/IT \
--username root --password cloudera \
--table IT \
--hive-import --hive-table database_Name_Hive.tablename_hive \
--m 1
5. Check hive data imported successfully

Exporting data from HDFS to MySQL:
1. Create databse and table in hive
2. Insert data into hive table
insert into IT values("Josan",34);
3. Create database and table in MySQL in which data must be exported
4. Run following command in Hadoop:
sqoop export --connect \
jdbc:mysql://127.0.0.1"3306/IT \
--table IT \
--username root --password cloudera \
--export-dir /user/hive/warehouse/databasename.db/tablename \
--m 1 \
-- driver com.mysql.jdbc.Driver
--input-fields-terminated-by ','
5. Check MySQL data exported successfully or not

