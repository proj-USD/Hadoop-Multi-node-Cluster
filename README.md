# Hadoop-Multi-node-Cluster

Installation of Hadoop in Ubuntu

Prerequisite to Hadoop Installation

1.	 You should have installed Ubuntu Desktop version in your Virtual Machine
2.	 You have installed Java(jdk 1.8) in your Ubuntu system.

JAVA_HOME=/usr/local/java/jdk1.8.0_31

$ sudo apt-get install vim

4.	Check your hostname is Ubuntu
$ hostname //Will output Ubuntu

Linux Configuration Before Hadoop Installation

Lets first install single node Hadoop cluster on Ubuntu and then we can go ahead with Multi-node cluster. 

You are expected to know basic UNIX commands and VI editor commands. 
 
We will setup single node Hadoop cluster using a dedicated Hadoop user 'hduser'
1.	Login as Root
$ sudo su
# whoami         should give root

2.	Create a Group called hadoop
# sudo addgroup hadoop

3.	Adding a dedicated Hadoop system user  called hduser
We will use a dedicated Hadoop user account for running Hadoop. While that’s not required it is recommended because it helps to separate the Hadoop installation from other software applications and user accounts running on the same machine (think: security, permissions, backups, etc). 

# sudo adduser hduser
              It will ask you to enter password 2 times followed by some details,just press enter and Yes
              We have given password hadoop

4.	Add hduser to hadoop group
# sudo  adduser  hduser hadoop  


5.	Add the ‘hduser’ to sudoers list so that hduser can do admin tasks.
$ sudo visudo
	Add a line under ##Allow member of group sudo to execute any command anywhere in the 	format. 
	hduser ALL=(ALL) ALL 
	Press ctrl+x, Y enter enter 
This will add the user hduser and the group hadoop to your local machine.

6.	Logout Your System and login again as hduser

7.	Change your display settings to 1440*900

8.	Configuring SSH
Hadoop requires SSH access to manage its nodes, i.e. remote machines plus your local machine if you want to use Hadoop on it. For our single-node setup of Hadoop, we therefore need to configure SSH access to localhost for the hduser user we created in the previous section. 
I assume that you have SSH up and running on your machine and configured it to allow SSH public key authentication. If not, there are several guides available. 
First, we have to generate an SSH key for the hduser user.  
	#Install ssh server on your computer 
      hduser@ubuntu:~$ sudo apt-get install openssh-server

If this didnot work,then install openssh-server using Ubuntu Software center by searching for openssh-server.

9.	Generate SSH for communication
hduser@ubuntu:~$ ssh-keygen  
Just press Enter for what ever is asked.

Generating public/private rsa key pair. 
Enter file in which to save the key (/home/hduser/.ssh/id_rsa):
 Created directory '/home/hduser/.ssh'. 
Your identification has been saved in /home/hduser/.ssh/id_rsa. 
Your public key has been saved in /home/hduser/.ssh/id_rsa.pub.
 The key fingerprint is: 9b:82:ea:58:b4:e0:35:d7:ff:19:66:a6:ef:ae:0e:d2hduser@localhost The key's randomart image is:
 [...snipp...] 
hduser@ubuntu:~$

The final step is to test the SSH setup by connecting to your local machine with the hduser user. The step is also needed to save your local machine’s host key fingerprint to the hduser user’s known_hosts file. If you have any special SSH configuration for your local machine like a non-standard SSH port, you can define host-specific SSH options in $HOME/.ssh/config (see man ssh_config for more information).

10.	Copy Public Key to Authorized_key file & edit the permission
#now copy the public key to the authorized_keys file, so that ssh should not require passwords every time  
hduser@ubuntu:~$cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys   
#Change permissions of the authorized_keys fie to have all permissions for hduser  
hduser@ubuntu:~$chmod 700 ~/.ssh/authorized_keys   


11.	Start SSH
If ssh is not running, then run it by giving the below command  
hduser@ubuntu:~$ sudo /etc/init.d/ssh  restart

12.	Test Your SSH Connectivity
hduser@ubuntu:~$ ssh  localhost
Type 'Yes',when asked for.You should be able to connect without password

13.	Disable IPV6
Hadoop and IPV6 do not agree on the meaning of 0.0.0.0 address, thus it is advisable to disable IPV6 adding the following lines at the end of /etc/sysctl.conf
hduser@ubuntu:~$ sudo vim /etc/sysctl.conf
      	# disable ipv6			
	net.ipv6.conf.all.disable_ipv6 = 1	
	net.ipv6.conf.default.disable_ipv6 = 1	
    net.ipv6.conf.lo.disable_ipv6 = 1	
14.	Check if IPv6 is disabled.
After a system reboot the output of 
hduser@ubuntu:~$ cat /proc/sys/net/ipv6/conf/all/disable_ipv6 
should be 1, meaning that IPV6 is actually disabled


Hadoop Installation

1.	Download Hadoop
Download hadoop-xxx.tar.gz and save it to hduser/Desktop.

2.	move the zip file to /usr/local/
 
$ cd Desktop
$ sudo mv ~/Desktop/hadoop-2.7.3.tar.gz /usr/local/
$ cd /usr/local
$ sudo gunzip hadoop-2.7.3.tar.gz
$ sudo tar -xvf hadoop-2.7.3.tar
$ sudo rm hadoop-2.7.3.tar
$ sudo ln -s hadoop-2.7.3 hadoop
$ sudo chown -R hduser:hadoop hadoop-2.7.3
$ sudo chmod 777 hadoop-2.7.3


3.	Add Java location to Hadoop so that it can recognize Java

Add the following to /usr/local/hadoop/conf/hadoop-env.sh
   $ vim /usr/local/hadoop/conf/hadoop-env.sh
	 export HADOOP_OPTS=-Djava.net.preferIPv4Stack=true 
	 export HADOOP_HOME_WARN_SUPPRESS="TRUE" 
	 export JAVA_HOME=/usr/local/java/jdk1.8.0_31  
	
The first Export will disable IPV6
4.	Update $HOME/.bashrc
Add the following lines to the end of the $HOME/.bashrc file of user hduser. If you use a shell other than bash, you should of course update its appropriate configuration files instead of .bashrc.
$ vim ~/.bashrc  

# Set Hadoop-related environment variables 
export HADOOP_HOME=/usr/local/hadoop  
# Set JAVA_HOME (we will also configure JAVA_HOME directly for Hadoop later on) 
export JAVA_HOME=/usr/local/java/jdk1.8.0_31  
# Some convenient aliases and functions for running Hadoop-related commands 

unaliasfs&> /dev/null 
aliasfs="hadoop fs" 
unaliashls&> /dev/null 
aliashls="fs -ls"  

# Conveniently inspect an LZOP compressed file from the command 
# line; run via: 
# 
# $ lzohead /hdfs/path/to/lzop/compressed/file.lzo 
#
# Requires installed 'lzop' command. 
# lzohead () { hadoopfs -cat $1 | lzop -dc | head -1000 | less }  
# Add Hadoop bin/ directory to PATH 
export PATH=$PATH:$HADOOP_HOME/bin:$PATH:$JAVA_HOME/bin

You need to close the terminal and open a new terminal to have the bash changes into effect. The shortcut to open the terminal is (Ctrl+Atl+t).

	
5.	Create a temporary directory which will be used as base location for DFS.
Now we create the directory and set the required ownerships and permissions:
$ sudo mkdir -p /app/hadoop/tmp 
$ sudo chown -R hduser:hadoop /app/hadoop/tmp 
$ sudo chmod -R 777 /app/hadoop/tmp
If you forget to set the required ownerships and permissions, you will see a java.io.IOException when you try to format the name node in the next section).
6.	Update core-site.xml file
Add the following snippets between the <configuration> ... </configuration> tags in/usr/local/hadoop/conf/core-site.xml:
	vim  /usr/local/hadoop/conf/core-site.xml  
	<!-- In: conf/core-site.xml --> 

	<property> 
		<name>hadoop.tmp.dir</name> 
		<value>/app/hadoop/tmp</value> 
		<description>A base for other temporary directories.</description> 	
	</property>  
	 <property>
		 <name>fs.default.name</name> 				
		<value>hdfs://localhost:54310</value> 
		<description>The name of the default file system.  
		A URI whose scheme and authority determine the FileSystem 
		implementation.  The uri's scheme determines the config property
		 (fs.SCHEME.impl) naming theFileSystem implementation class.  
		The uri's authority is used to determine the host, port, etc. for 
		a filesystem.
		</description> 
	</property>

7.	Update mapred-site.xml file
Add the following to /usr/local/hadoop/conf/mapred-site.xml  between<configuration> ... </configuration>
vim /usr/local/hadoop/conf/mapred-site.xml  
	<property> 
		<name>mapred.job.tracker</name>
		 <value>localhost:54311</value> 
		<description>The host and port that the MapReduce job tracker runs at.  		If "local", then jobs are run in-process as a single map and reduce task.
		 </description> 
	</property>  

8.	Update hdfs-site.xml file
Add the following to /usr/local/hadoop/conf/hdfs-site.xml  between<configuration> ... </configuration>

vim /usr/local/hadoop/conf/hdfs-site.xml  

<property>
   <name>dfs.replication</name>
   <value>1</value>
   <description>Default block replication.   The actual number of replications can be specified when the file is created.   The default is used if replication is not specified in create time.</description>
</property>

9.	Format your namenode
Open a new Terminal as the hadoop command will not work

Format hdfs cluster with below command 
[hduser@localhost ~]$hadoop namenode -format  
If the format is not working, double check your entries in .bashrc file. The .bashrc updating come into force only if you have opened a new terminal.

10.	Starting your single-node cluster
Congratulations, your Hadoop single node cluster is ready to use. Test your cluster by running the following commands.

 [hduser@localhost ~]$start-all.sh  
A command line utility to check whether all the hadoop demons are running or not is: jps  
Give the jps at command prompt and you should see something like below.  
[hduser@localhost ~]$ jps 
9168 Jps 
9127 TaskTracker 
8824 DataNode 
8714 NameNode 
8935 SecondaryNameNode 
9017 JobTracker  
Check if the hadoop is accessible through browser by hitting the below URLs 
For mapreduce - http://localhost:50030 
For hdfs - http://localhost:50070   

11.Check if you have /user/hduser folder in your HDFS 
$ hadoop fs -ls
ls: Cannot access .: No such file or directory.

If you get the above error, then manually create the directory
$hadoop fs -mkdir /user/hduser
Again issue the above command 
$hadoop fs -ls
You should not get any error now

Errors:
If you get the below error
2015-09-13 20:30:14,664 WARN org.apache.hadoop.hdfs.server.datanode.DataNode: Invalid directory in dfs.data.dir: Incorrect permission for /app/hadoop/tmp/dfs/data, expected: rwxr-xr-x, while actual: rwxrwxrwx
$ sudo chmod 755 /app/hadoop/tmp/dfs/data


