# bigdata
# Missing-Poker-Cards
This is an homework exercise of my Big Data course. In this MapReduce code to you have to find missing poker cards in a deck of 52 cards.

# How to Run:
* Copy Poker Input File.txt to HDFS using command:
```
 hadoop fs -rm -r -f -skipTrash /MissingPokerCards_input.txt
```
```
  hadoop fs -rm -r -f -skipTrash /output
```
```
hadoop fs -put MissingPokerCards_input.txt /
```
* Make the .jar file of MissingPokerCards.java

```
mkdir ~/MissingPokerCards
```
```
javac -classpath `hadoop classpath` -d MissingPokerCards poker.java
```
```
jar -cvf MissingPokerFinder.jar -C MissingPokerCards
```

* Run MissingPokerFinder.jar file on Poker Input File.txt using the command:
```
hadoop jar MissingPokerFinder.jar finalPoker.poker /MissingPokerCards_input.txt /output
```


* To copy jar file from ec2 to local: Note (Please change the pem file and Public IPv4 DNS as per yours)
```
scp -i ubuntu_aws_key.pem ubuntu@ec2-34-229-7-41.compute-1.amazonaws.com:/home/ubuntu/MissingPokerFinder.jar .
```
