# bigdata
# AWS instance Creation


# How to Run:
* Instance launch EC2- ubuntu 20.04:
* instance type- t2 micro
* Create key pair
```
 security groups
Type		Protocol		Port Range		Source 		Description
SSh			TCP				22				Anywhere	SSH for Admin Desktop
Custom TCP	TCP				8000 - 9000		Anywhere
Custom TCP 	TCP				50000 - 60000	Anywhere	
All traffic	All				All				Anywhere
```
* Instance create
* On Window: change keyname
```
  Right click on .pem file -> Properties -> Security -> For Special permisison (Advanced)-> Disable inheritance-> Add(computer name)->  Slect a principal  -> check your name--> Read  -> apply
```
* To connect to AWS nodes
```
ssh -i "awskey.pem" ubuntu@ec2-3-95-28-233.compute-1.amazonaws.com
ssh -i "awskey.pem" ubuntu@ec2-52-87-181-170.compute-1.amazonaws.com
ssh -i "awskey.pem" ubuntu@ec2-44-202-36-84.compute-1.amazonaws.com
ssh -i "awskey.pem" ubuntu@ec2-3-83-152-178.compute-1.amazonaws.com

```
* ssh passphraseless on all the nodes

```
ssh-keygen -t rsa -N "" -f /home/ubuntu/.ssh/id_rsa
cd .ssh
cat id_rsa.pub 
vi authorized_keys  ---Keep from all the nodes "id_rsa.pub" value

```
* Update /etc/hosts file (User your Private address)
```
vi /etc/hosts
172.31.85.189 master
172.31.94.198 slave1
172.31.85.218 slave2
172.31.89.84 slave3
```
# To install Hadoop 2.6.5
```
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

* Add below information in ~/.bash_profile
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_HOME=/home/ubuntu/hadoop-2.6.5
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
```


* Get file Hadoop-2.6.5 form internet and untar
```
wget https://archive.apache.org/dist/hadoop/common/hadoop-2.6.5/hadoop-2.6.5.tar.gz
tar -xvzf hadoop-2.6.5.tar.gz

```
* Need to update below files on master node  from /home/ubuntu/hadoop-2.6.5/etc/hadoop
```
cd /home/ubuntu/hadoop-2.6.5/etc/hadoop
```
* core-site.xml
```
<configuration>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://master:9000/</value>
        </property>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>file:/home/ubuntu/hadoop-2.6.5/tmp</value>
        </property>
</configuration>
```
* hdfs-site.xml
```
<configuration>
        <property>
                <name>dfs.namenode.name.dir</name>
                <value>file:/home/ubuntu/hadoop-2.6.5/dfs/name</value>
        </property>
        <property>
                <name>dfs.datanode.data.dir</name>
                <value>file:/home/ubuntu/hadoop-2.6.5/dfs/data</value>
        </property>
        <property>
                <name>dfs.replication</name>
                <value>3</value>
        </property>
</configuration>
```
* mapred-site.xml.template
```
<configuration>
        <property>
                <name>mapreduce.framework.name</name>
                <value>yarn</value>
        </property>
</configuration>
```
* yarn-site.xml
```
-->
<configuration>

<!-- Site specific YARN configuration properties -->
         <property>
                <name>yarn.nodemanager.aux-services</name>
                <value>mapreduce_shuffle</value>
        </property>
        <property>
                <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
                <value>org.apache.hadoop.mapred.ShuffleHandler</value>
        </property>
        <property>
                <name>yarn.resourcemanager.address</name>
                <value>master:8032</value>
        </property>
        <property>
                <name>yarn.resourcemanager.sheduler.address</name>
                <value>master:8030</value>
        </property>
        <property>
                <name>yarn.resourcemanager.resource-tracker.address</name>
                <value>master:8035</value>
        </property>
        <property>
                <name>yarn.resourcemanager.admin.address</name>
                <value>master:8033</value>
        </property>
        <property>
                <name>yarn.resourcemanager.webapp.address</name>
                <value>master:8088</value>
        </property>

</configuration>
```
* hadoop-env.sh
```
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
export HADOOP_PREFIX=/home/ubuntu/hadoop-2.6.5

export HADOOP_CONF_DIR=/home/ubuntu/hadoop-2.6.5/etc/hadoop
export HADOOP_HOME=/home/ubuntu/hadoop-2.6.5

for file in $HADOOP_HOME/share/hadoop/*/*.jar;
do
    export CLASSPATH=$CLASSPATH:$file
done
for file in $HADOOP_HOME/share/hadoop/*/lib/*.jar;
do
  export CLASSPATH=$CLASSPATH:$file
done
```
```
source hadoop-env.sh
```
* master
```
vi master
master
```
* slaves
```
vi slaves
slave1
slave2
slave3
```
* send hadoop files update to all other nodes
```
scp -r  hadoop-2.6.5 slave1:~
scp -r  hadoop-2.6.5 slave2:~
scp -r  hadoop-2.6.5 slave3:~
```
* Format namenode on master (Only one time)
```
bin/hdfs namenode -format
```
* Start dfs and yarn on msater
```
sbin/start-dfs.sh
sbin/start-yarn.sh
jps
```
# Oozie Install

* maven and other software installed 
```
sudo apt-get update
sudo apt-get install maven
sudo apt-get install unzip
sudo apt-get install zip
```
* Download Oozie 4.1.0 from the Apache URL (http://archive.apache.org/dist/oozie/4.1.0/oozie-4.1.0.tar.gz) and save the tarball to any directory
```
wget http://archive.apache.org/dist/oozie/4.1.0/oozie-4.1.0.tar.gz
tar -zvxf oozie-4.1.0.tar.gz
```
* Update the pom.xml to change the default hadoop version to 2.3.0. The reason we’re not changing it tohadoop version 2.6.0 here is because 2.3.0-oozie-4.1.0.jar is the latest available jar fi le. Luckily it works with higher versions in 2.x series
```
cd oozie-4.1.0
vi pom.xml
--Search for
<hadoop.version>3.0.0-SNAPSHOT</hadoop.version>
--Replace it with
<hadoop.version>2.3.0</hadoop.version>
```
* Add below line under mirrors
```
sudo vi /etc/maven/settings.xml
<mirror>
    <id>central</id>
    <url>https://repo.maven.apache.org/maven2</url>
    <mirrorOf>central</mirrorOf>
</mirror>
```
* Build Oozie executable
```
mvn clean package assembly:single -P hadoop-2 -DskipTests
```
* The executable will be generated in the target sub directory under distro dir.
```
cd distro/target/
tar -zvxf oozie-4.1.0-distro.tar.gz
sudo mv oozie-4.1.0 /usr/local/oozie-4.1.0
```
* Download ext-2.2.zip, this file is no longer available on the extjs.com ,that’s why downloading from cloudera archive https://archive.cloudera.com/gplextras/misc/ext-2.2.zip
```
cd /usr/local/oozie-4.1.0
mkdir libext
cd libext
wget https://archive.cloudera.com/gplextras/misc/ext-2.2.zip
```
* Copy the hadoop lib files
```
cp -R  ~/oozie-4.1.0/hadooplibs/hadoop-2/target/hadooplibs/hadooplib-2.3.0.oozie-4.1.0/* /usr/local/oozie-4.1.0/libext/
```
* Edit core-site.xml under hadoop configuration directory to add some lines (substitute hduser with ubuntu or whichever user owns oozie. Replace, if required, hadoop with the group the user belongs to eg., ubuntu):
```
vi /home/ubuntu/hadoop-2.6.5/etc/hadoop/core-site.xml

-- Add under <configuration>
<property>
	<name>hadoop.proxyuser.ubuntu.hosts</name>
	<value>*</value>
</property>
<property>
	<name>hadoop.proxyuser.ubuntu.groups</name>
	<value>*</value>
</property>

```
* Edit ~/.bashrc to add path to new oozie bin directory:
```
vi ~/.bashrc

export OOZIE_VERSION=4.1.0
export OOZIE_HOME=/usr/local/oozie-4.1.0
export PATH=$PATH:$OOZIE_HOME/bin
source ~/.bashrc
```
* Build the war file for oozie web console
```
cd /usr/local/oozie-4.1.0
./bin/oozie-setup.sh prepare-war
```
* Setup metastore using Derby 
```
./bin/ooziedb.sh create -sqlfile oozie.sql -run
```
* To start Oozie, run the following command from the Oozie home directory:
```
bin/oozied.sh start

--You have to replace with your Public DNS name
http://ec2-54-196-228-248.compute-1.amazonaws.com:11000/oozie/

```
