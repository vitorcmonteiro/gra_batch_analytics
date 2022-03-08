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
2. [Configure Spark node](#configure-spark-node)
2. [Setup Local Development Environment](#setup-local-development-environment)
3. [Code Project & Test](#code-project-&-test)

## Setup EC2 Environment
We are going through the following key steps:
1. Create EC2 free-tier instance
2. Configure ports
3. Connect to SSH
4. Install Spark
5. Install OpenJDK (Java)
6. Configure Spark node


### **Create EC2 free-tier instance**
In order to create an EC2 instance, go to the EC2 service in your AWS Management Console and click Launch instance (orange button). You should choose **Amazon Linux 2 5.10 - 64-bit (x86)**.

![Instances Dashboard](https://user-images.githubusercontent.com/22838513/148470698-3d8f1e86-4f25-455e-8aae-d3fe3b194459.png)

After choosing this machine, keep selecting options with "Free tier". At the step 6 (Configure Security Group), you should **make sure that port 22 is open**.

When you launch the instance, a dialogue asking about key pair opens up. **Create a new pair**, naming it whatever you want. I'm calling it "gra_admin". Download Key pair (.pem file) and save it somewhere you will remember.

![Key Pair Pop Up](https://user-images.githubusercontent.com/22838513/148470907-45b0f638-f0b3-466e-810b-c931944c9ff8.png)

Launch your instance. Now, AWS is doing the setup and should take a few minutes.

![Instance running](https://user-images.githubusercontent.com/22838513/148471014-1c90a323-4cc3-4dcf-b95e-3107442d8029.png)

We will use the pem file we downloaded to SSH inside our AWS instance. Let's go back to our WSL2 (local development environment). In order to copy your pem file to the correct place, change the following terminal command to match your paths:
```
$ cp <path to downloaded pem file> ~/.ssh/
```

For example:
```
$ cp /mnt/c/Users/Vitor/Desktop/gra_admin.pem ~/.ssh/
```

In my case, I have saved the file at my user's desktop (Windows) and I wanted to copy that file to WSL2 and follow the steps below that you may find when clicking "Connect" in the instance Dashboard.

![SSH Steps](https://user-images.githubusercontent.com/22838513/148471106-b7cdf1d7-347f-428c-a43b-118c77169380.png)


### **Configure ports**
After creating the instance and saving your credentials to access your instance from your local desktop, we need to open the ports so we can connect to the server and to the spark instance we will create in the future.

On the left menu go to the Network & Security section and you will find Security Groups.

![security_groups_1](https://user-images.githubusercontent.com/22838513/157165712-fceb550a-57ce-4b38-9512-d78eaddef4f3.png)

Make sure you select the same security group attached to your instance and in actions you are able to edit Inbound Rules.

Add the following:

![ports](https://user-images.githubusercontent.com/22838513/157165780-9c699b70-019c-43d0-ae77-be1141cc7da7.png)

Save your editions and move to the next section.


### **Connect to SSH**

After going through all these steps you should see an EC2 icon after running the following command (Make sure to change the server as needed):
```
$ ssh -i "gra_admin.pem" ec2-user@ec2-54-157-49-180.compute-1.amazonaws.com
```
If everything worked correctly, you may see the EC2 icon and your user/machine has changed:
![EC2 Terminal](https://user-images.githubusercontent.com/22838513/148471148-1d03d090-2315-45c0-b627-a9397354dda0.png)

### **Install Spark**
We are going to download Spark from its official website directly from the EC2's instance terminal. We are also downloading a version with Hadoop included but we are not going to use it. Hadoop is just a file system used to transfer and store data but we are using S3 for this specific purpose.

Make sure you go into Spark's website (https://spark.apache.org/downloads.html) and download the latest version

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

![java_-version](https://user-images.githubusercontent.com/22838513/148471677-91c9108b-1a59-4173-86cc-b2346f104763.png)

## Configure Spark node
Although Spark is installed, we need to configure some paths and files so that it runs smoothly as a service in our EC2 machine. To achieve that, we will go through these steps:
1. Configure paths
2. Check ```jars``` folder
3. Configure ```spark-defaults.conf```
4. Create credentials file at ```.aws``` folder
5. Run Spark instance

### **Configure paths**
First we must configure paths for both Spark and Python (PySpark) inside ```.bashrc``` file. You need to take note of two paths: Spark and Python's folders. We have extracted Spark in this tutorial and you may use this command to track where is Python:

```
$ which python3
```

Take note of the path as we will use later. We also know where we extracted Spark's installation, also take note of that path.

SPARK_HOME
: Path to Spark's folder <br>
PYSPARK_PYTHON
: Path to Python's folder

Remember that ec2-user is the default user created by AWS and is written at the beginning of the console lines (Which is the user logged in currently):

![ec2-user](https://user-images.githubusercontent.com/22838513/148471722-f27450e2-40a9-4230-98c4-98a0533dc8da.png)

With these paths saved, run the following command:

```
$ nano ~/.bashrc
```

Insert at the end of ```.bashrc``` file:
```
export SPARK_HOME=/home/ec2-user/spark
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin
export PYSPARK_PYTHON=/usr/bin/python3
```

Close and save the file and use the following command to update the .bashrc loaded into our EC2 instance:
```
$ source ~/.bashrc
```

After running ``` source ``` try this command to see if it worked (Shows the value you have set in the above step):
```
$ echo $SPARK_HOME
```

![echo_$SPARK_HOME](https://user-images.githubusercontent.com/22838513/148471749-390a8d6f-dfde-4e08-bb87-1f702208a102.png)

### **Check ```jars``` folder**
One very important step is to make sure your jar folder contains all required jar files to run a Spark node. This step usually causes trouble as some files are missing from that folder. **It is fundamental to understand that all versions of the following jar files must be the same** [^3][^4].

### **Configure ```spark-defaults.conf```**
The main objective of changing this file is to **safely** store AWS credentials that will be used by Spark to access S3 or any other AWS service. With this file configured you don't need to include your keys in your code.

Go to IAM service and create a new user:

![new user](https://user-images.githubusercontent.com/22838513/157165841-25877654-48ef-44f5-b405-f37425122413.png)

Name it as you prefer but make sure to select "Access Key" as the AWS credential type. You may add that user to a group but since it it a small scale solution you can attach the security policy directly to that user:

![access](https://user-images.githubusercontent.com/22838513/157165887-bd173440-7b84-4e65-aa87-0432765ea41c.png)

Add these permissions ```AmazonS3FullAccess``` and ```AmazonEC2FullAccess```. After advancing until the end, you will get a key pair, take note of them:

![key](https://user-images.githubusercontent.com/22838513/157165938-3b6ad68f-1287-4867-a056-def1f66edefd.png)

Now refer back to your ```spark-defaults.conf``` file at the ```~/spark/conf``` folder:

```
$ cp spark-defaults.conf.template spark-defaults.conf
$ nano spark-defaults.conf
```

Add the following to the end of the file:

```
spark.hadoop.fs.s3a.aws.credentials.provider com.amazonaws.auth.profile.ProfileCredentialsProvider
```

### **Create credentials file at ```.aws``` folder
At the user root (~) create a folder called ```.aws```. Inside that folder create 2 files: config and credentials.

```
$ mkdir ~/.aws
$ cd ~/.aws
$ touch config
$ touch credentials
```

```
$ nano config
```

Add the following (Depends on your S3 region, make sure they match):
```
[default]
region = us-east-2
```

Save and exit.

```
$ nano credentials
```

Add the following (Depends on your S3 region, make sure they match):
```
[default]
aws_access_key_id=AKIAQEIXZNBNVEY22GYH
aws_secret_access_key=kP3FHPt5zjGEu0BQEt2DsNVqQ5b56m8a+mebJcDz
```

Save and exit.

Now your credentials should be saved in a safe way and you don't need to include them in your code.

### **Run Spark instance**
To run the spark cluster you should type the following command:

```
$ start-master.sh
```

![start-master](https://user-images.githubusercontent.com/22838513/157165969-bf801511-2635-4500-9556-979089e40dd1.png)

To confirm the web UI port, grab the "logging to" part from the output and verify what is in the file:

![cat](https://user-images.githubusercontent.com/22838513/157166142-5b74ff07-55ce-436b-a94f-aa9023655b8c.png)

You will be able to access through your browser using both ```Public IPv4 DNS``` or ```Public IPv4 address```. **Make sure you access them using only http instead of https and that the port 8080 is added at the end.**

The next ip address needed is the ip from the spark master cluster. When you access the webpage below in the header you will find the ip address:

![spark master](https://user-images.githubusercontent.com/22838513/157165997-633dfd78-1c9f-41fa-b0dc-c51cfe4e870b.png)

```
$ start-worker.sh spark://ip-172-31-30-209.ec2.internal:7077
```

Refresh the page and you will see now that there's one worker alive:

![worker](https://user-images.githubusercontent.com/22838513/157166017-bd0c3ed6-8297-42be-b3b8-8cb78f8b1585.png)

You can always run a temporary spark cluster when using ```spark-submit.sh <python script>```. When you submit to a cluster you point to that server when running spark-submit.

## Setup Local Development Environment
For local development I personally use vsCode in conjunction with Git.

## Code Project & Test

```
sudo yum install git-all
```

# References
[^1]: https://spark.apache.org/
[^2]: https://spark.apache.org/docs/1.6.2/ec2-scripts.html
[^3]: https://stackoverflow.com/questions/44411493/java-lang-noclassdeffounderror-org-apache-hadoop-fs-storagestatistics
[^4]: https://stackoverflow.com/questions/49158612/java-lang-nosuchmethoderror-org-apache-hadoop-conf-configuration-reloadexisting
