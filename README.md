# Hadoop-Map-Reduce-Analysis
I configure single-node Hadoop on MAC and use two MapReduce example WordCount and Grep to get more familiar with Hadoop Map Reduce.

## Steps
1.First Install Hadoop-2.7.2 on MAC (<https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html>)

2.For each example program, I need to preset the data on the HDFS.
```
hadoop fs -put /dataset/* /dataset
```

3.Run apache.hadoop.examples.WordCount program to count the words in the dataset.
```
hadoop jar hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar wordcount /dataset/ /output
```

4.Run apache.hadoop.examples.Grep program to grep certain string that matches regular expression.
```
hadoop jar hadoop-2.7.2/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.2.jar grep /dataset/ /output "Microsoft"
```

5.Analyze the data.


## Deliverable
| Directory | Meaning | 
| --------- |:-------|
| grep | Both small and big dataset are stored in grep/dataset. The output is in the grep/output. The statistics results are in the grep/statistics.txt|
| wordcount | Both small and big dataset are stored in wordcount/dataset. The output is in the wordcount/output. The statistics results are in the wordcount/statistics.txt |
| org/apache/hadoop/examples | Source code of Hadoop MapReduce examples including Grep.java and WordCount.java |
| screenshot | Runtime terminal screenshots |

### (a)	URL to the MapReduce codes and the datasets used

<https://github.gatech.edu/wguo64/Hadoop-Map-Reduce-Analysis/>

I am using hadoop-mapreduce-examples-2.7.2-sources.jar to do this experiment. In this jar file, WordCount.java and Grep.java are the two example programs to be used in the experiment.

### (b)	screen shots of your execution process.

<img src="https://github.com/Peleus1992/Hadoop-Map-Reduce-Analysis/blob/master/screenshot/1.png" height="320" width="480"/>
<img src="https://github.com/Peleus1992/Hadoop-Map-Reduce-Analysis/blob/master/screenshot/2.png" height="320" width="480"/>
### (c)	Runtime statistics in tabular format.

| Parameter | WordCount small dataset | WordCount big dataset | Grep small dataset | Grep big dataset |
| --------- |:-----------------------:| ---------------------:|-------------------:|-----------------:|
Map input records | 2 | 4778 | 3 | 146       
Map output records |  201581 | 3 | 146
Map output bytes | 139 | 1997745 | 70 | 100604
Map output materialized bytes | 142 | 1456899 | 82 | 101130
Input split bytes | 190 | 38897 | 130 | 130
Combine input records | 14 | 201581 | 0 | 0
Combine output records | 11 | 112748 | 0 | 0
Reduce input groups | 9 | 21706 | 1 | 2
Reduce shuffle bytes | 142 | 1456899 | 82 | 101130
Reduce input records | 11 | 112748 | 3 | 146
Reduce output records | 9 | 21706 | 3 | 146
Spilled Records | 22 | 225496 | 6 | 292
Shuffled Maps | 2 | 401 | 1 | 1
Failed Shuffles | 0 | 0 | 0 | 0
Merged Map outputs | 2 | 401 | 1 | 1
GC time elapsed (ms) | 0 | 7011 | 0 | 7
Total committed heap usage (bytes) | 962592768 | 204483330048 | 934281216 | 1029701632
Time used | 4.32s | 21.43s | 5.67s | 29.76s

### (d)	Analysis.
I use apache/hadoop/examples/WordCount.java as an example to analyze the Mapper and Reducer.

```Java

  public static class TokenizerMapper 
       extends Mapper<Object, Text, Text, IntWritable>{
    
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();
      
    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
      StringTokenizer itr = new StringTokenizer(value.toString());
      while (itr.hasMoreTokens()) {
        word.set(itr.nextToken());
        context.write(word, one);
      }
    }
  }
```
Map stage : The TokenizerMapper’s job is to process the input data. All the input data is stored in the /dataset in Hadoop file system (HDFS). The input file is passed to the TokenizerMapper function line by line. It use StringTokenizer to split the string using space. Then the mapper processes the tokens and write them in Context.

```Java
public static class IntSumReducer 
       extends Reducer<Text,IntWritable,Text,IntWritable> {
    private IntWritable result = new IntWritable();

    public void reduce(Text key, Iterable<IntWritable> values, 
                       Context context
                       ) throws IOException, InterruptedException {
      int sum = 0;
      for (IntWritable val : values) {
        sum += val.get();
      }
      result.set(sum);
      context.write(key, result);
    }
  }
```

Reduce stage : Reducer's job is to process the data that comes from the mapper. In this case, IntSumReducer sum up all the word count from TokenizerMapper and add it to the result set. Then the new set of output will be stored in the HDFS.

Because of the MapReduce framework, the wordcount and grep program can be very efficient when there are many nodes in Hadoop cluster. Due to the fact that I do not have so many nodes, I only install single-node on MAC. As a result, the performance is not very good.
