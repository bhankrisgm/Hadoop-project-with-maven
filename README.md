# Hadoop-project-with-maven
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fbhankrisgm%2FHadoop-project-with-maven.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fbhankrisgm%2FHadoop-project-with-maven?ref=badge_shield)

A Java WordCount example with Hadoop maven dependencies set
This is an exercise that will help you install and run hadoop program written in Java, first in your IDE in local mode, and then in an Hadoop cluster that you will build yourself

## Prerequisites
1. IDE (eclipse or IntelliJ)
2. Java SDK
3. Maven
4. Need to have enough disk space in your root folder, otherwise hadoop job will not run.
5. Install [docker](https://docs.docker.com/engine/installation/) (20 minutes)
(Mac or Windows machines: Docker must be installed inside a virtual machine. You start docker from a docker terminal)
6. Install [Cloudera Docker Container](http://www.cloudera.com/content/www/en-us/documentation/enterprise/latest/topics/quickstart_docker_container.html) (download 4GB)
7. Optional (not necessary for this exercise): if you want to have your hadoop env saved with your changes, you can use a VM instead of docker. See: Cloudera Quickstart VM.

## Installation
After installing Cloudera Manager Container as explained [here](http://www.cloudera.com/content/www/en-us/documentation/enterprise/latest/topics/quickstart_docker_container.html) follow those steps (also explained in the cloudera document)
Note: docker requires that you run it as sudoer (~$ sudo docker) unless you configure differently.

1. Run: `~$ sudo docker images`
2. Obtain the IMAGE ID of REPOSITORY: `cloudera/quickstart`
3. Mac and Windows users: after opening docker terminal, run the command in the terminal: `eval "$(docker-machine env default)"`
4. Run: `~$ sudo docker run --hostname=quickstart.cloudera --privileged=true -v /tmp:/tmp -p 8888:8888 -t -i <IMAGE ID> /usr/bin/docker-quickstart`
(-v is used to share volume between docker and physical disk. -p is used to forward port from docker to phisical machine. In this case port 8888 for Hue)
5. After the container is up, you have a command prompt in cloudera container.
6. Switch user to 'cloudera': `~$ su cloudera`
7. Obtain ip of your container. Open a new console from you PC and type: `~$ sudo docker ps`
8. Linux users: Obtain the CONTAINER ID of the docker process and run: `~$ sudo docker inspect <CONTAINER_ID> | grep IPAddress`. Use the **IPAddress** later in the exercise
9. Mac and Windows users: use the command to obtain docker host **IPAddress**: `docker-machine ls`
 
## Getting to know hadoop tools

1. Use “Hue” UI tool: open browser and enter the url: [http://IPAddress:8888](). (in linux you can use localhost, as in the “docker run” command you forwarded the port 8888 to localhost) log in using: cloudera/cloudera.
2. On the top right of the screen you see two important tools: 
  1. **Manage HDFS** will allow you to browse the HDFS file system
  2. **Job Browser** will let you view the status of running Map/Reduce jobs.
3. On the top Left of the screen there is another important tool “Query Editor” that will allow you to run queries for Hive and Imapala.
 
## Exercise 1: Using Hadoop ecosystem
**Goal:** In this exercise you will get familiar with some of Hadoop ecosystem tools. E.g., **Hue**, **Sqoop**, **Hive** and **Impapla**.
(Partially taken from [Cloudera getting started tutorial](http://www.cloudera.com/developers/get-started-with-hadoop-tutorial.html))

1. Go to the Cloudera docker console to start with Sqoop import
2. type:
```
sqoop import-all-tables \
    -m 1 \
    --connect jdbc:mysql://quickstart.cloudera:3306/retail_db \
    --username=retail_dba \
    --password=cloudera \
    --compression-codec=snappy \
    --as-parquetfile \
    --warehouse-dir=/user/hive/warehouse \
    --hive-import
```
This command may take a while to complete, but it is doing a lot. It is launching MapReduce jobs to pull the data from our MySQL database and write the data to HDFS in parallel, distributed across the cluster in **Apache Parquet** format. 
It is also creating tables to represent the HDFS files in Impala/Apache Hive with matching schema.

### Verification
When this command is complete, confirm that your data files exist in HDFS.
```
hadoop fs -ls /user/hive/warehouse/
hadoop fs -ls /user/hive/warehouse/categories/
```
These commands will show the directories and the files inside them that make up your tables

### Run Hive and Imapala queries on imported data
1. Open Hue web UI in URL: [http://IPAddress:8888/]()
2. In the top menu go to: **Query Editors** > **Hive**
3. In the SQL editor type `show tables;` and execute. You will see the new added tables in the result set.
4. Run the SQL that will retrieve the most popular product categories:
```
-- Most popular product categories
select c.category_name, count(order_item_quantity) as count
from order_items oi
inner join products p on oi.order_item_product_id = p.product_id
inner join categories c on c.category_id = p.product_category_id
group by c.category_name
order by count desc
limit 10;
```
5. While the Hive query is running, open Hue in another browser tab and go to the **Job Browser**. Hive performs MapReduce jobs that can be monitored in Hadoop.
6. Perform steps #2 to #5 but this time use **Imapa** Query Editor instead of **Hive**
7. When running Impala, check the following:
 1. Is Impala faster or slower?
 2. Do you see anything in the Job Browser when Impala runs? Why?

### Command line tools
Hue is a nice web interface, but anything you can do with it you can do in command line

1. Open Hive shell by typing `hive` in the cloudera docker console
2. Repeat step #4 from previous section in the hive console
3. type `exit` and press `Enter` to exit Hive shell.
4. To view jobs run in Hadoop, type:

```
mapred job -list all 
yarn application -list
```
## Exercise 2: Write and run Java MapReduce job in local mode
**Goal**: In this exercise we will use code of Java MapReduce and run it in IDE and debug it in local mode.

1. Clone or copy the source files from this guthub project: **pom.xml** and **WordCount.java**.
2. Prepare some files in HDFS to work on:
3. In the Cloudera docker console type:

 ```
 sudo su cloudera
 cd
 #here we create directories in HDFS:
 hdfs dfs -mkdir /user/cloudera/wordcount /user/cloudera/wordcount/input
 #here we create three files in local file system:
 echo "Hadoop is an elephant" > file0
 echo "Hadoop is as Yellow as can be" > file1
 echo "Oh what a yellow fellow is Hadoop" > file2
 #here we copy the local files to HDFS:
 hdfs dfs -put file* /user/cloudera/wordcount/input
 ```
4. Run WordCount from your IDE on your PC that will access the docker container on which Hadoop is running. You need to pass two program arguments: input and output folders: `hdfs://<IPAddress>:8020/user/cloudera/wordcount/input hdfs://<IPAddress>:8020/user/cloudera/wordcount/output`
5. Check that the new files were created using Hue:  [http://<IP_ADDRESS>:8888]() “Manage HDFS”. locate the file: __/user/cloudera/wordcount/output/part-00000__ and view its content
6. Check the Job Browser in Hue. Was there a job running? why?
7. Where did the program run? where is the data? how did the program read the data?

### Modify WordCount program
As you probably noticed, the program distinguishes between uppercase and lowercase letters.

1. Change the program to be case insensitive.
2. Did you make your change in the Mapper or Reducer code?
3. Run the program with debugger and add some breakpoints to stop in the code.
 
## Exercise 3: Deploy MapReduce program on a cluster
**Goal:** Hadoop MapReduce programs do not run in local mode. In production they always run on a cluster. Now we will deploy our program on the cluster.
**Note:** Although we did not install Hadoop cluster with several machines and we run one docker machine, it behaves like a real cluster of one node.

1. The maven **pom.xml** in our project contains a shadow plugin that creates uber jar with all dependencies in it. it will be created in target directory with name “hadoop-tutorial-1.0-SNAPSHOT.jar” 
2. Run `maven package` to create the jar file.
3. Copy the jar file to `/tmp/` which is shared directory with the docker container. 
4. On the docker container machine, run the job using “yarn jar” command:

 ```
 su cloudera
 cd
 yarn jar /tmp/hadoop-tutorial-1.0-SNAPSHOT.jar main.java.org.tikalk.WordCount hdfs://<IPAddress>:8020/user/cloudera/wordcount/input hdfs://<IPAddress>:8020/user/cloudera/wordcount/output
 ```
5. Check the Job Browser in Hue to see your job running.


## License
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fbhankrisgm%2FHadoop-project-with-maven.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2Fbhankrisgm%2FHadoop-project-with-maven?ref=badge_large)