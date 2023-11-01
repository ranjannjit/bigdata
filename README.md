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
* To install Hadoop 2.6.5
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
