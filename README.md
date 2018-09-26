hathi-client
============

This repository contains client configuration for the SURFsara Hadoop cluster
Hathi. At the moment it contains configuration for Hadoop 2.7.2 and Spark 2.1.1.

Please note that Spark 2.1.1 is a relatively old Spark version. 

The restriction to Spark 2.1.1 is due to the (non-perfect) configuration of the 
SurfSara cluster, and the fact that its worker machines only have Java 7. 
(newer versions of Spark like 2.2.X and beyond require Java 8).

Prerequisites
-------------

Windows is not supported so in that case uou need to use Linux in VirtualBox. We 
need Linux with a GUI, so not the lsde VM images. 
Use the VM http://event.cwi.nl/lsde/surfsara.zip (passwd: nimda) 

This VM has the hathi-client software and this repo and all Linux packages
menioned below preinstalled, Hadoop and Spark preinstalled and Firefox configured.

You can also directly install this software on a Linux or MacOS laptop.

If you have a Linux laptop, you need to make sure that Git, Java 7 and the Kerberos 
client libraries are installed. 

Debian-based Linux (Debian, Ubuntu, Mint):
```
    > apt-get install wget git openjdk-7-jre-headless krb5-user firefox
```
Enterprise Linux (Redhat, CentOS, Fedora):
```
    > yum install wget git java-1.7.0-openjdk krb5-workstation firefox
```

Make sure JAVA_HOME points to this Java 7 installation directory. 
You can also keep using a new java (and scala) environment, but then yiu
must pass extra compilation flags when building jars, to emit only
1.76 bytecode. Please see the geturls/build.sbt makefile for these flags.

For MacOS there is no more support or downloads for Java 7, so you must 
pass these flags.

Another way to ensure this is to clone your repo on the login.hathi.surfsara.nl
node, pull your code when you want to run something and compile the jars there 
(the login node has a Java 7 SDK). On the login node you can use the sbt
installed in /home/pboncz/sbt/bin


Local Hadoop and Spark
----------------------

In order to develop on your laptop (i.e. not on the cluster, this is required) 
then you need to install hadoop-2.7 and spark-2.1.1:

The first time you need to download the official Hadoop/Spark software from
Apache and put the SURFsara configuration in the right location. We provide a
helper script that will do this automatically:

```
> git clone --depth 1 https://github.com/peterboncz/hathi-client
> hathi-client/bin/get.sh hadoop
> hathi-client/bin/get.sh spark
```

(this script has already been executed in the homedir in the surfsara VM)

Kerberos setup
--------------

The Hadoop Surfsara cluster uses Kerberos authentication. This is a pain, but
you *must* master this, otherwise you cannot see the status of your jobs in
your web browser (and see the log files).

For Linux: (this step has already been done in the surfsara VM)
```
    > sudo cp hathi-client/conf/krb5.conf /etc/
```

For MacOS:
```
    > cp hathi-client/conf/krb5.conf $HOME/Library/Preferences/edu.mit.Kerberos
```


Usage
-----

Whenever you want to use the cluster you need to perform the following once per
session.

0) re-authenticate using Kerberos:
```
> kinit <USERNAME>
```
(USERNAME = lsdeXX)

1) Setup the environment:
```
> eval $(hathi-client/bin/env.sh)
```
(You can add this line to your `~/.profile` so that it is run automatically on
login).

2) use the Hadoop and Spark utilities:
```
> hdfs dfs -ls /

> yarn jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-*.jar pi 5 5

> spark-submit --class org.apache.spark.examples.SparkPi \
                 --master yarn  --deploy-mode cluster \
                 $SPARK_HOME/examples/jars/spark-examples_2.11-2.1.1.jar 10
```

Browser setup
-------------

The status of HDFS can be checked in: <http://head02.hathi.surfsara.nl>.
The status of the cluster (YARN) is in: <http://head05.hathi.surfsara.nl>.
Job outpouts and log files can be reached by clicking through there, you
will end up at workerXXX.hathi.surfsara.nl nodes.

Browsers need to be instructed to use Kerberos authentications for all
web addresses in the hathi.surfsara.nl domain. Please use either FireFox
or Chrome.

Firefox instructions: (this has already been done in the surfsara VM)

Go the about:config (promise to be careful). Search for the key
`network.negotiate-auth.trusted-uris` and add the value `.hathi.surfsara.nl`.

Google Chrome instructions:

Add the domain .hathi.surfsara.nl in the --auth-server-whitelist and  
--auth-negotiate-delegate-whitelist command line options on startup (requires 
closing all Chrome windows, and restarting from a termina/command shell).

On Linux: chrome --auth-server-whitelist=".hathi.surfsara.nl" \
                 --auth-negotiate-delegate-whitelist=".hathi.surfsara.nl" \
                 http://head05.hathi.surfsara.nl 2>/dev/null >/dev/null&

On MacOS: /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
                 --auth-server-whitelist=".hathi.surfsara.nl" \
                 --auth-negotiate-delegate-whitelist=".hathi.surfsara.nl" \
                 http://head05.hathi.surfsara.nl 2>/dev/null >/dev/null&

Without doing this, checking the status, outputs and errors will really be a puzzle.


Developing
----------

To help you get off the ground, there is a geturls/ demo spark application. 
This program reads a HDFS file (*not* a local file) named landsat_small.txt and 
then downloads the URLs in these, outputting the files in HDFS. By doing this 
through Spark executors, in parallel, you could download a whole lot of files 
at the same time and store them all in HDFS.

The tool to build the .jar here is sbt. This tool is specifically created 
for scala/java projects. You may of course also use Maven or even Ant if you 
are more familiar with those. sbt will install the right scala version (2.11) 
automatically.

The code is in src/main/scala/geturls.scala

You can build with "sbt package" and then you can run the job on the cluster 
using spark-submit, asking yarn to schedule it and deploy on the cluster with 
(here just eight) executors.

```
cd geturls
sbt package
spark-submit --master yarn --deploy-mode cluster --num-executors 8 \
             --class geturls target/scala-2.11/geturls_2.11-1.0.jar
```

In order to run this, you will have to change some file paths in geturls.scala. 
Also, after running once, you may want to remove the landsat directory (with 
force and recursively: hdfs dfs -rm -f -R landsat)


Support
-------

For more information about the SURFsara Hadoop cluster see
<https://userinfo.surfsara.nl/systems/hadoop>.

Please seek help on the LSDE slack channel first.

For persistent questions using Hadoop at SURFsara contact the [SURFsara
helpdesk](mailto:helpdesk@surfsara.nl?subject=Help%20with%20Hadoop%20hathi-client).

License
-------

Copyright 2014-2017 SURFsara BV
Copyright 2018 CWI

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

<http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
