# Introduction
This tutorial is designed to help setup a technology stack using Spark and Jupyter Notebooks to enable processing batch data. For this particular application our Spark cluster will be installed in a EC2 instance in AWS while we run our analysis locally.

## What is Spark?
Spark is a in-memory distributed computing engine. As such, he is used to process large data sets and thus **does not directly depends on Hadoop (HDFS) and distributed processing (YARN)**. These file systems and clustering features work together with Spark in case we need a more robust solution to store/retrieve data and distribute it, respectively. [^1]

Further explanation about Apache tech stack can be found in the following video: https://www.youtube.com/watch?v=aReuLtY0YMI.

# Tech Stack
* AWS EC2 (Spark) [^2]
* AWS S3 (Data Storage)
* Jupyter Notebook (Local Development)

# Setting up your first Spark node

This tutorial comprises the following key steps:<br>

1. [Setup EC2 Environment](#setup-ec2-environment)
2. [Configure Spark](#configure-spark)
2. [Setup Local Development Environment](#setup-local-development-environment)
3. [Code Project & Test](#code-project-&-test)

## Setup EC2 Environment
We are going through the following key steps:
1. Create EC2 free-tier instance
2. Install Spark
3. Install OpenJDK (Java)
4. Configure Spark node

### **Create EC2 free-tier instance**
In order to create an EC2 instance, go to the EC2 service in your AWS Management Console and click Launch instance (orange button). You should choose **Amazon Linux 2 5.10 - 64-bit (x86)**.

After choosing this machine, keep selecting options with "Free tier". At the step 6 (Configure Security Group), you should open the following ports:

(Image)

When you launch the instance, a dialogue asking about key pair opens up. **Create a new pair**, naming it whatever you want. I'm calling it "gra_admin". Download Key pair (.pem file) and save it somewhere you will remember.

Launch your instance. Now, AWS is doing the setup and should take a few minutes.

After waiting for a few minutes, we will use the pem file we downloaded to SSH inside our AWS instance. Let's go back to our WSL2 (local development environment).

In order to copy your pem file to the correct place, change the following terminal command to match your paths:
```
$ cp <path to downloaded pem file> ~/.ssh/
```

For example:
```
$ cp /mnt/c/Users/Vitor/Desktop/gra_admin.pem ~/.ssh/
```

In my case, I have saved the file at my User's desktop (Windows) and I wanted to copy that file to WSL2 and follow the steps below that you may find when clicking "Connect" in the instance Dashboard.

(Image)

After going through all these steps you should see an EC2 icon after running the following command (Make sure to change the server as needed):

```
$ ssh -i "gra_admin.pem" ec2-user@ec2-54-157-49-180.compute-1.amazonaws.com
```

(Image) 

### **Install Spark**
We are going to download Spark from its official website directly from the EC2's instance terminal. We are also downloading a version with Hadoop included but we are not going to use it. Hadoop is just a file system used to transfer and store data but we are using S3 for this specific purpose.

Inside your EC2 instance (You may get an updated link at Apache's website):
```
$ wget https://dlcdn.apache.org/spark/spark-3.2.0/spark-3.2.0-bin-hadoop3.2.tgz
```

Now we must unpack the files to a folder:
```
$ tar zxvf spark-3.2.0-bin-hadoop3.2.tgz
```

To make it easier, let's rename the folder to a friendly name:
```
$ mv spark-3.2.0-bin-hadoop3.2 spark
$ rm spark-3.2.0-bin-hadoop3.2.tgz
```

With that, Spark is installed but we still need to install Java.

### **Install OpenJDK (Java)**
Spark needs a running Java in the background so it can invoke jar files. My EC2 instance was missing this service so we will go through this installation:
```
$ sudo yum install java-1.8.0-openjdk
```

After installing, check if it is working by using the following command: 
```
$ java -version
```

(Image)

## Configure Spark node
Although Spark is installed, we need to configure some paths and files so that it runs smoothly as a service in our EC2 machine. To achieve that, we will go through these steps:
1. Configure paths
2. Check ```jars``` folder
3. Configure ```spark-defaults.conf```

### **Configure paths**
First we must configure paths for both Spark and Python (PySpark) inside ```.bashrc``` file. You need to take note of two paths: Spark and Python's folders. We have extracted Spark in this tutorial and you may use this command to track where is Python:

```
$ which python3
```

Now take account of the following variable names, do not change them, only the paths value.

SPARK_HOME
: Path to Spark's folder
PYSPARK_PYTHON
: Path to Python's folder

Remember that ec2-user is the default user created by AWS and is written at the beginning of the console lines (Which is the user logged in currently):

(Image)

 With these paths saved, run the following command:

```
$ nano ~/.bashrc
```

Insert at the end pf .bashrc file:
```
export SPARK_HOME=/home/ec2-user/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export PYSPARK_PYTHON=/usr/bin/python3
```

Then use the following command to update the .bashrc loaded into our EC2 instance:
```
$ source ~/.bashrc
```

After running ``` source ``` try this command to see if it worked (Shows the value you have set in the above step):
```
$ echo $SPARK_HOME
```

(Image)

### **Check ```jars``` folder**
One very important step is to make sure your jar folder contains all required jar files to run a Spark node. This step usually causes trouble as some files are missing from that folder. **It is fundamental to understand that all versions of the following jar files must be the same** [^3][^4].

### **Configure ```spark-defaults.conf```**

## Setup Local Development Environment

## Code Project & Test

```
sudo yum install git-all
```

# References
[^1]: https://spark.apache.org/
[^2]: https://spark.apache.org/docs/1.6.2/ec2-scripts.html
[^3]: https://stackoverflow.com/questions/44411493/java-lang-noclassdeffounderror-org-apache-hadoop-fs-storagestatistics
[^4]: https://stackoverflow.com/questions/49158612/java-lang-nosuchmethoderror-org-apache-hadoop-conf-configuration-reloadexisting