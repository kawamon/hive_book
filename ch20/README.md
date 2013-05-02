Programmin Hive Oozie Samples
=============

I found it is difficult to run Oozie labs in my environment. hope this information helps you.

Build Application
-----
GIT Clone 'Fixed' git.

$ git clone https://github.com/kawamon/m6d_oozie.git
$ cd m6d_oozie;
$ mvn install

$ mkdir -p myapp/lib
$ cp $HIVE_HOME/lib/*.jar myapp/lib
$ cp m6d_oozie/target/m6d_oozie-2.0.0-SNAPSHOT.jar myapp/lib/
$ cp hive_test/target/hive_test-4.0.0-SNAPSHOT.jar myapp/lib/


job.properties
-----
I modified the job.properties as below. You should modifiy below properties.
 - nameNode
 - jobTracker
 - baseDir (if needed)

Here is my sample.

nameNode=hdfs://localhost:8020<br/>
jobTracker=localhost:8021<br/>
queueName=default<br/>
baseDir=${nameNode}/user/${user.name}/myapp<br/>
oozie.wf.application.path=${baseDir}/job<br/>
oozie.libpath=${oozie.wf.application.path}/lib<br/>

workflow.xml
-----
I also modified the workflow as below. You should modifiy below properties.
 - <arg>localhost</arg>  : These two argument should be hiveserver hostname


<workflow-app xmlns="uri:oozie:workflow:0.2" name="java-main-wf">
    <start to="create-node"/>
    <action name="create-node">
        <java>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
            <main-class>com.m6d.oozie.HiveServiceBAction</main-class>
            <arg>localhost</arg>
            <arg>10000</arg>
            <arg>CREATE TABLE IF NOT EXISTS zz_zz_abc (a int, b int)</arg>
        </java>
        <ok to="query_node"/>
        <error to="fail"/>
    </action>

    <action name="query_node">
        <java>
            <job-tracker>${jobTracker}</job-tracker>
            <name-node>${nameNode}</name-node>
            <configuration>
                <property>
                    <name>mapred.job.queue.name</name>
                    <value>${queueName}</value>
                </property>
            </configuration>
            <main-class>com.m6d.oozie.HiveServiceBAction</main-class>
            <arg>localhost</arg>
            <arg>10000</arg>
            <arg>INSERT OVERWRITE TABLE zz_zz_abc SELECT dma_code,site_id
            FROM BCO WHERE dt=20120426 AND offer=4159 LIMIT 10</arg>
        </java>
        <ok to="end"/>
        <error to="fail"/>
    </action>

    <kill name="fail">
        <message>Java failed, error message
        [${wf:errorMessage(wf:lastErrorNode())}]</message>
    </kill>
    <end name="end"/>
</workflow-app>


Setup BCO Hive Table
----------
You should create BCO Table before you run workflow! Please see bco.q and bco.txt files.
$ hive -f bco.q



Copy files
----------
Copy the job.properties and the workflow.xml into correct directory.
myapp
└── job
    ├── job.properties
    ├── lib
    │   ├── antlr-runtime-3.0.1.jar
    │           :
    │   ├── hive_test-4.0.0-SNAPSHOT.jar
    │           :
    │   ├── m6d_oozie-2.0.0-SNAPSHOT.jar
    │   └── zookeeper-3.4.3.jar
    └── workflow.xml



Upload files into HDFS
----------
$ hadoop fs -put myapp


Run Oozie workflow
------------
(You have to run oozie-server)

$ oozie job -oozie http://localhost:11000/oozie -config myapp/job/job.properties -auth simple -run


Hue and Oozie
------------
Please try Hue
<img src="http://github.com/kawamon/hive_book/blob/master/ch20/hue_oozie.png" alt="Hue and Oozie" title="Hue and Oozie">
