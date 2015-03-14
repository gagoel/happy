# Hadoop + Python = Happy #

**Happy** is a framework for writing map-reduce programs for Hadoop using Jython.  It files off the sharp edges on Hadoop and makes writing map-reduce programs a breeze.

The current release is 0.1, but we've been using it for a long time at http://www.freebase.com for mining data, so rest assured that it is stable and full-featured.

The 0.1 release is compiled against Hadoop 0.17.2.  We'll be releasing a version compiled against 0.18.1 as soon as we upgrade our cluster.

You can download Happy [here](http://happy.googlecode.com/files/happy-0.1.tgz).  Documentation and examples are [here](http://www.mqlx.com/~colin/happy.html).

Join the Happy discussion group [here](http://groups.google.com/group/happy-user) if you have questions or find a bug.

## Happy Overview ##

**Happy** is a framework that allows [Hadoop](http://hadoop.apache.org/core/) jobs to be written and run in [Python 2.2](http://www.python.org/doc/2.2.1/) using [Jython](http://www.jython.org/Project/index.html).  It is an easy way to write map-reduce programs for Hadoop, and includes some new useful features as well.  The current release supports Hadoop 0.17.2.

Map-reduce jobs in Happy are defined by sub-classing `happy.HappyJob` and implementing a `map(records, task)` and `reduce(key, values, task)` function.  Then you create an instance of the class, set the job parameters (such as inputs and outputs) and call `run()`.

When you call `run()`, Happy serializes your job instance and copies it and all accompanying libraries out to the Hadoop cluster.  Then for each task in the Hadoop job, your job instance is de-serialized and `map` or `reduce` is called.

The task results are written out using a collector, but aggregate statistics and other roll-up information can be stored in the `happy.results` dictionary, which is returned from the `run()` call.

Jython modules and Java jar files that are being called by your code can be specified using the environment variable `HAPPY_PATH`.  These are added to the Python path at startup, and are also automatically included when jobs are sent to Hadoop.  The path is stored in `happy.path` and can be edited at runtime.


## Obligatory Wordcount Example ##

Here's an example of word count implemented in Happy:

```
import sys, happy, happy.log

happy.log.setLevel("debug")
log = happy.log.getLogger("wordcount")

class WordCount(happy.HappyJob):  
    def __init__(self, inputpath, outputpath):
        happy.HappyJob.__init__(self)
        self.inputpaths = inputpath
        self.outputpath = outputpath
        self.inputformat = "text"
      
    def map(self, records, task):
        for _, value in records:
            for word in value.split():
                task.collect(word, "1")
    
    def reduce(self, key, values, task):
        count = 0;
        for _ in values: count += 1
        task.collect(key, str(count))
        log.debug(key + ":" + str(count))
        happy.results["words"] = happy.results.setdefault("words", 0) + count
        happy.results["unique"] = happy.results.setdefault("unique", 0) + 1

if __name__ == "__main__":
    if len(sys.argv) < 3:
        print "Usage: <inputpath> <outputpath>"
        sys.exit(-1)
    wc = WordCount(sys.argv[1], sys.argv[2])
    results = wc.run()
    print str(sum(results["words"])) + " total words"
    print str(sum(results["unique"])) + " unique words"
```