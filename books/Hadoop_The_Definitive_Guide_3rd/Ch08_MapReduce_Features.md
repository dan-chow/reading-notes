## Chapter 08: MapReduce Features

### Counters

Counters are a useful channel for gathering statistics about the job: for quality control
or for application-level statistics. 

Built-in counter groups
- MapReduce task counters
- Filesystem
counters
- FileInputFormat
counters
- FileOutputFormat
counters
- Job counters

Task counters gather information about tasks over the course of their execution, and
the results are aggregated over all the tasks in a job. 
Task counters are maintained by each task attempt, and periodically sent to the tasktracker and then to the jobtracker so they can be globally aggregated.


Job counters are maintained by the jobtracker (or application master in
YARN), so they don’t need to be sent across the network, unlike all other counters,
including user-defined ones. They measure job-level statistics, not values that change
while a task is running.


MapReduce allows user code to define a set of counters, which are then incremented
as desired in the mapper or reducer. Counters are defined by a Java enum, which serves
to group related counters. A job may define an arbitrary number of enums, each with
an arbitrary number of fields. The name of the enum is the group name, and the enum’s
fields are the counter names. Counters are global: the MapReduce framework aggregates them across all maps and reduces to produce a grand total at the end of the job.


Application to run the maximum temperature job, including counting missing and malformed fields and quality codes
```java
public class MaxTemperatureWithCounters extends Configured implements Tool {
  enum Temperature {
    MISSING,
    MALFORMED
  }
  static class MaxTemperatureMapperWithCounters
      extends Mapper<LongWritable, Text, Text, IntWritable> {
    private NcdcRecordParser parser = new NcdcRecordParser();
    @Override
    protected void map(LongWritable key, Text value, Context context)
        throws IOException, InterruptedException {
      parser.parse(value);
      if (parser.isValidTemperature()) {
        int airTemperature = parser.getAirTemperature();
        context.write(new Text(parser.getYear()), new IntWritable(airTemperature));
      } else if (parser.isMalformedTemperature()) {
        System.err.println("Ignoring possibly corrupt input: " + value);
        context.getCounter(Temperature.MALFORMED).increment(1);
      } else if (parser.isMissingTemperature()) {
        context.getCounter(Temperature.MISSING).increment(1);
      }
      // dynamic counter
      context.getCounter("TemperatureQuality", parser.getQuality()).increment(1);
    }
    // implementation elided
  }
}
```

Because a Java enum’s fields are defined at compile time, you can’t create new counters
on the fly using enums.
The method we use on the Reporter object takes a group and
counter name using String names:
```java
public void incrCounter(String group, String counter, long amount)
```
The two ways of creating and accessing counters—using enums and using strings—
are actually equivalent because Hadoop turns enums into strings to send counters over
RPC.


By default, a counter’s name is the enum’s fully qualified Java classname. These names
are not very readable when they appear on the web UI or in the console, so Hadoop
provides a way to change the display names using resource bundles.
The recipe to provide readable names is as follows. Create a properties file named after
the enum, using an underscore as a separator for nested classes. The properties file
should be in the same directory as the top-level class containing the enum. 

The properties file should contain a single property named CounterGroupName, whose
value is the display name for the whole group. Then each field in the enum should have
a corresponding property defined for it, whose name is the name of the field suffixed
with .name and whose value is the display name for the counter. Here are the contents
of MaxTemperatureWithCounters_Temperature.properties:
```
CounterGroupName=Air Temperature Records
MISSING.name=Missing
MALFORMED.name=Malformed
```

In addition to being available via the web UI and the command line (using hadoop job
-counter), you can retrieve counter values using the Java API.
First we retrieve a RunningJob object from a JobClient by calling the getJob() method
with the job ID. We check whether there is actually a job with the given ID.
After confirming that the job has completed, we call the RunningJob’s getCounters()
method, which returns a Counters object, encapsulating all the counters for a job. The
Counters class provides various methods for finding the names and values of counters.


A Streaming MapReduce program can increment counters by sending a specially formatted line to the standard error stream, which is co-opted as a control channel in this
case. The line must have the following format:
```
reporter:counter:group,counter,amount
```


### Sorting


The sort order for keys is controlled by a RawComparator, which is found as follows:
1. If the property mapred.output.key.comparator.class is set, either explicitly or by
calling setSortComparatorClass() on Job, then an instance of that class is used. (In
the old API, the equivalent method is setOutputKeyComparatorClass() on JobConf.)
2. Otherwise, keys must be a subclass of WritableComparable, and the registered
comparator for the key class is used.
3. If there is no registered comparator, then a RawComparator is used that deserializes
the byte streams being compared into objects and delegates to the WritableCompar
able’s compareTo() method.


MapFileOutputFormat provides a pair of convenience static methods for performing
lookups against MapReduce output.
```java
Reader[] readers = MapFileOutputFormat.getReaders(path, getConf());
Writable entry = MapFileOutputFormat.getEntry(readers, partitioner, key, val);
```
The getReaders() method opens a MapFile.Reader for each of the output files created
by the MapReduce job. The getEntry() method then uses the partitioner to choose the
reader for the key and finds the value for that key by calling Reader’s get() method.
We can also use the readers directly to get all the records for a given key. The array of
readers that is returned is ordered by partition, so that the reader for a given key may
be found using the same partitioner that was used in the MapReduce job:
```java
Reader reader = readers[partitioner.getPartition(key, val, readers.length)];
```
Once we have the reader, we get the first key using MapFile’s get() method, then repeatedly call next() to retrieve the next key and value until the key changes.




