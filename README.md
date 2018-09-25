hathi-client
============

This repository contains client configuration for the SURFsara Hadoop cluster
Hathi. At the moment it contains configuration for Hadoop 2.7.2 and Spark 2.3.1.

You need a working Java 7 SDK installed in your system (properly set JAVA_HOME 
and onclude java in your PATH). NOT: java 8! Surfsra is not ready for that yet.

Local Hadoop and Spark
----------------------

If you develop on your laptop (which is recommended) then you need to install
hadoop-2.7 and spark-2.1.3 (NOT spark 2.3.1 or anything higher than 2.1.3
--- this is because spark-2.2.* and beyond require java 8, and the SurfSara 
cluster workers only have java 7 - so there is no way that will ever work)

The first time you need to download the official Hadoop/Pig/Spark software from
Apache and put the SURFsara configuration in the right location. We provide a
helper script that will do this automatically:

```
> git clone --depth 1 https://github.com/peterboncz/hathi-client
> hathi-client/bin/get.sh hadoop
> hathi-client/bin/get.sh spark
```

This script only works for MacOS and Linux -- developing Hadoop on windows
is not recommended or supported. If you have a Windows laptop, use Linux
in a VirtualBox. You need Linux with a GUI, so not the LSDE VM images.

e.g.
sf.net/projects/virtualappliances/files/Linux/Fedora/Fedora-20-amd64-gui.ova/download
user/passwd: fedora/nimda  
user/passwd: root/toor

Kerberos setup
--------------

The Hadoop Surfsara cluster uses Kerberos authentication. This is a pain, but
you (must* master this, otherwise you cannot see the status of your jobs in
your web browser.

For OSX:

    > cp hathi-client/conf/krb5.conf $HOME/Library/Preferences/edu.mit.Kerberos

For Linux:

    > sudo cp hathi-client/conf/krb5.conf /etc/


Now you can authenticate using Kerberos:
```
> kinit <USERNAME>
```

(USERNAME = lsdeXX)

Usage
-----

Whenever you want to use the cluster you need to perform the following once per
session.

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
                 $SPARK_HOME/examples/jars/spark-examples_2.11-2.3.1.jar 10
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

Firefox instructions:

Go the about:config (promise to be careful). Search for the key
`network.negotiate-auth.trusted-uris` and add the value `.hathi.surfsara.nl`.

Google Chrome instructions:

Add the domain .hathi.surfsara.nl in the --auth-server-whitelist and  
--auth-negotiate-delegate-whitelist command line options on startup (requires 
closing all Chrome windows, and restarting from a termina/command shell).

On Linux: chrome --auth-server-whitelist=".hathi.surfsara.nl" --auth-negotiate-delegate-whitelist=".hathi.surfsara.nl" http://head05.hathi.surfsara.nl 2>/dev/null >/dev/null&
On MacOS: /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --auth-server-whitelist=".hathi.surfsara.nl" --auth-negotiate-delegate-whitelist=".hathi.surfsara.nl" http://head05.hathi.surfsara.nl 2>/dev/null >/dev/null&

Without doing this, checking the status, outouts and errors will really be a puzzle.


Support
-------

For more information about the SURFsara Hadoop cluster see
<https://userinfo.surfsara.nl/systems/hadoop>.

Please seek help on the LSDE slack channel first.

For persistent questions using Hadoop at SURFsara contact the [SURFsara
helpdesk](mailto:helpdesk@surfsara.nl?subject=Help%20with%20Hadoop%20hathi-client).

License
-------

Copyright 2014 SURFsara BV

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

<http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
