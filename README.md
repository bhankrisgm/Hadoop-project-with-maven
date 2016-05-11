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
8. Linux users: Obtain the CONTAINER ID of the docker process and run: `~$ sudo docker inspect <CONTAINER_ID> | grep IPAddress`
9. Mac and Windows users: use the command to obtain docker host IP: `docker-machine ls`
 
## Getting to know hadoop tools

1. Use “Hue” UI tool: open browser and enter the url: http://<IP_ADDRESS>:8888. (in linux you can use localhost, as in the “docker run” command you forwarded the port 8888 to localhost) log in using: cloudera/cloudera.
2. On the top right of the screen you see two important tools: 
  1. **Manage HDFS** will allow you to browse the HDFS file system
  2. **Job Browser** will let you view the status of running Map/Reduce jobs.
3. On the top Left of the screen there is another important tool “Query Editor” that will allow you to run queries for Hive and Imapala.
 


