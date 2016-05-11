# Hadoop-project-with-maven
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

1. Use “Hue” UI tool: open browser and enter the url: [http://<IPAddress>:8888](). (in linux you can use localhost, as in the “docker run” command you forwarded the port 8888 to localhost) log in using: cloudera/cloudera.
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

