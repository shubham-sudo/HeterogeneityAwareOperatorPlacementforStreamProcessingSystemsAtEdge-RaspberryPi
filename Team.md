# Team 4


# Getting Started with Flink-Placement
Follow these steps to set up and deploy the Flink-Placement project on your system.

## Step 1: Clone the Repository
Clone the Flink-Placement repository using the following link:

    git clone  https://cs551-gitlab.bu.edu/cs551/spring23/group-4/flink-placement

## Step 2: Load and Build the Project
Load the project in IntelliJ and build it using Maven configuration:

    clean package -DskipTests

## Step 3: Copy Build Contents
Copy the contents from the `build-target` folder to the `/opt/flink` folder.

## Step 4: Copy Scripts Folder
Copy the scripts folder to the `/opt/flink` directory.

## Step 5: Modify Configuration Files
In the scripts folder, modify the following files:

    workers
    masters
    slots
    flink-conf.yaml

## Step 6: Run the Deployment Script
Run the `deployflink.py` script. This script will automatically copy the Flink folder and respective configs to every worker (Raspberry Pis in our case).

## Step 7: Start the Flink Cluster
Start the Flink cluster using the following command:

    ./bin/start-cluster.sh

## Step 8: Modify the Scheduler Configuration
Modify the `schedulercfg` in the `scripts` folder according to your job and placement. Make sure you have the correct `schedulercfg` and `flink-conf.yaml`.

## Step 9: Submit the Job
Submit the job, and you can monitor the JobManager logs for now to see logs related to our cost model.
<br/>

# Experiments
Use the `flink_exp.py` script to run the experiments after submitting the job. This script would change the bandwidth, as well as manually switch the operator to each location `server` or the `Raspberry Pis`. This script would also add the timestamps, so that we can use it analyse the metrics collected in the `metrics.json` file in `/opt/flink/scripts` folder.


# Tests
We have implemented a query `Query11` in the example folder of the [Flink placement project (Team 4)](https://cs551-gitlab.bu.edu/cs551/spring23/group-4/flink-placement) repository. Running a job of this query tests our cost model and switching of the operators automatically. It can be easily customized for different complexity and selectivity for the tests. There is also a version of it `Query11S70` with a different selectivity inside the same folder.

# Auxiliary codes and scripts

In the `scripts` folder in our [flink placement project](https://cs551-gitlab.bu.edu/cs551/spring23/group-4/flink-placement), we have the following helper scripts -
1. deploy_flink.py - Used to deploy the newly build target of the flink project to all the master and worker nodes, and place the correct configurations.
2. flink_exp.py - Used to run experiments, while varying the bandwidth and placement of the operators (the choice value).
3. clean_exp.py - Resets the bandwidth of the Raspberry Pis.
4. limitcpu.py - Limits the CPU performance of the Raspberry Pis incase they are faster than your server to simulate real world.
5. switch_operator.py - Switches the filter operator from Server to Raspberry Pi or vice-versa based on the choice value sent.

---

# Apache Flink

Apache Flink is an open source stream processing framework with powerful stream- and batch-processing capabilities.

Learn more about Flink at [https://flink.apache.org/](https://flink.apache.org/)


### Features

* A streaming-first runtime that supports both batch processing and data streaming programs

* Elegant and fluent APIs in Java and Scala

* A runtime that supports very high throughput and low event latency at the same time

* Support for *event time* and *out-of-order* processing in the DataStream API, based on the *Dataflow Model*

* Flexible windowing (time, count, sessions, custom triggers) across different time semantics (event time, processing time)

* Fault-tolerance with *exactly-once* processing guarantees

* Natural back-pressure in streaming programs

* Libraries for Graph processing (batch), Machine Learning (batch), and Complex Event Processing (streaming)

* Built-in support for iterative programs (BSP) in the DataSet (batch) API

* Custom memory management for efficient and robust switching between in-memory and out-of-core data processing algorithms

* Compatibility layers for Apache Hadoop MapReduce

* Integration with YARN, HDFS, HBase, and other components of the Apache Hadoop ecosystem


### Streaming Example
```scala
case class WordWithCount(word: String, count: Long)

val text = env.socketTextStream(host, port, '\n')

val windowCounts = text.flatMap { w => w.split("\\s") }
  .map { w => WordWithCount(w, 1) }
  .keyBy("word")
  .window(TumblingProcessingTimeWindow.of(Time.seconds(5)))
  .sum("count")

windowCounts.print()
```

### Batch Example
```scala
case class WordWithCount(word: String, count: Long)

val text = env.readTextFile(path)

val counts = text.flatMap { w => w.split("\\s") }
  .map { w => WordWithCount(w, 1) }
  .groupBy("word")
  .sum("count")

counts.writeAsCsv(outputPath)
```



## Building Apache Flink from Source

Prerequisites for building Flink:

* Unix-like environment (we use Linux, Mac OS X, Cygwin, WSL)
* Git
* Maven (we recommend version 3.2.5 and require at least 3.1.1)
* Java 8 or 11 (Java 9 or 10 may work)

```
git clone https://github.com/apache/flink.git
cd flink
mvn clean package -DskipTests # this will take up to 10 minutes
```

Flink is now installed in `build-target`.

*NOTE: Maven 3.3.x can build Flink, but will not properly shade away certain dependencies. Maven 3.1.1 creates the libraries properly.
To build unit tests with Java 8, use Java 8u51 or above to prevent failures in unit tests that use the PowerMock runner.*

## Developing Flink

The Flink committers use IntelliJ IDEA to develop the Flink codebase.
We recommend IntelliJ IDEA for developing projects that involve Scala code.

Minimal requirements for an IDE are:
* Support for Java and Scala (also mixed projects)
* Support for Maven with Java and Scala


### IntelliJ IDEA

The IntelliJ IDE supports Maven out of the box and offers a plugin for Scala development.

* IntelliJ download: [https://www.jetbrains.com/idea/](https://www.jetbrains.com/idea/)
* IntelliJ Scala Plugin: [https://plugins.jetbrains.com/plugin/?id=1347](https://plugins.jetbrains.com/plugin/?id=1347)

Check out our [Setting up IntelliJ](https://nightlies.apache.org/flink/flink-docs-master/flinkDev/ide_setup.html#intellij-idea) guide for details.

### Eclipse Scala IDE

**NOTE:** From our experience, this setup does not work with Flink
due to deficiencies of the old Eclipse version bundled with Scala IDE 3.0.3 or
due to version incompatibilities with the bundled Scala version in Scala IDE 4.4.1.

**We recommend to use IntelliJ instead (see above)**

## Support

Don’t hesitate to ask!

Contact the developers and community on the [mailing lists](https://flink.apache.org/community.html#mailing-lists) if you need any help.

[Open an issue](https://issues.apache.org/jira/browse/FLINK) if you found a bug in Flink.


## Documentation

The documentation of Apache Flink is located on the website: [https://flink.apache.org](https://flink.apache.org)
or in the `docs/` directory of the source code.


## Fork and Contribute

This is an active open-source project. We are always open to people who want to use the system or contribute to it.
Contact us if you are looking for implementation tasks that fit your skills.
This article describes [how to contribute to Apache Flink](https://flink.apache.org/contributing/how-to-contribute.html).


## About

Apache Flink is an open source project of The Apache Software Foundation (ASF).
The Apache Flink project originated from the [Stratosphere](http://stratosphere.eu) research project.
