# flume-connector
A proof-of-concept for using [Apache Flume](https://flume.apache.org/) to collect work data from on-premises tools for analysis in [Worklytics](https://www.worklytics.co).

There are several benefits of using Flume to send your data to Worklytics:
  * support ingestion from tools that lack sufficient webhook support or REST APIs
  * use Flume filters to exclude or sanitize data according to your particular data security needs
  * Flume is quite mature and well-understood by many developers

The drawback is that Flume is an extra component that you must deploy and operate somewhere in your infrastructure.

As we consider this a *proof-of-concept*, it is meant merely to demonstrate the viability of using Flume in this way. 
Worklytics makes no warranty about any of the software or example configurations that we've bundled into this repo. YMMV.


## Overview

This repo includes two things:
  * a built Flume distribution in a tar bundle with [HTTP Sink support](https://github.com/hmrc/flume-http-sink) added
  * example Flume configuration files for collecting data for various sources. 


Once you deploy this, your data will flow as follows:
  1. Flume will get data from a source. Flume supports dozens of sources. Here are a few relevant examples for Worklytics use-cases:
    - **system exec call** (reading from arbitrary flat-file or something). Useful if you want to send historical data from
      a tool's logs or database to Worklytics. You can run a dump command or `cat` the flat log files into Flume.
    - **tail a file**. Useful if the your tool writes continually to a local flat log file, as is common for many legacy systems.
    - **http**. (if the tool you're using supports webhooks, but you don't want it dialing Worklytics directly, this is a great solution)
  2. Data is filtered according to your Flume configuration. You can configure one of Flume's built-in filters, or
    add your own to Flume (in Java).
  3. Data that passes the filter is sent to Worklytics ingestion via HTTP `POST` requests. 


## Usage

1. Check out this repo.

2. Unpack that Flume tar.

3. Deploy it (probably /usr/local)

4. Configure it.

## Building from Scratch

High-level build steps:
  1. built Flume [see Getting Started - Building From Source](https://cwiki.apache.org/confluence/display/FLUME/Getting+Started)
  2. cloned the flume-http-sink repo
  3. used sbt to package it
  4. copied the JAR to the `lib` directory of the Flume we built in [1] [details](https://github.com/hmrc/flume-http-sink#installation)

Low-level build steps:
```
$ git clone https://git-wip-us.apache.org/repos/asf/flume.git flume
$ cd flume

# checkout latest stable Flume branch (1.7.x as of 31 July 2017; trunk is 1.8.x)
$ git checkout flume-1.7 

# Flume docs recommend this to avoid memory issues in build process
$ export MAVEN_OPTS="-Xms512m -Xmx1024m"
$ mvn install -DskipTests=true

# unpack built tar
$ cp flume-ng-dist/target/apache-flume-1.7.0-bin.tar.gz .
$ tar -zxvf apache-flume-1.7.0-bin.tar.gz
$ cd apache-flume-1.7.0-bin

# build flume-http-sink
$ git clone https://github.com/hmrc/flume-http-sink.git
$ cd flume-http-sink

# package it with sbt (Scala Build Tool); on OS X, available with Homebrew: brew install sbt
$ sbt clean package

# copy the JAR into flume
$ cp target/flume-http-sink-{version}.jar {flume_home}/lib/flume-http-sink-{version}.jar

# bundle it
$ cd ~
$ tar -cvzf flume.tar.gz {flume_home}
```

### Use something other than Flume

Flume is actually rather old and was specifically designed to collect data for Hadoop.  There are lots of other 
data/log collection solutions out there, such as [Apache Kafka](https://kafka.apache.org/), 
[FluentD](https://www.fluentd.org/), etc - all of which should support fairly similar use-cases. 


## Resources

This was largely developed from the (rather old) Flume documentations. 