# Hadoop MapReduce: Word Count Application

**Dataset:** [*Alice's Adventures in Wonderland - Direct Download Link*](https://www.gutenberg.org/cache/epub/11/pg11.txt)   
**Environment:** Pseudo-Distributed Hadoop Cluster via Windows Subsystem for Linux (WSL)

## Phase 1: System Environment Setup  
Configure the base operating system, Java development kit and passwordless SSH required for Hadoop's background daemons.

**1. Initialize WSL & Ubuntu**  
Open Windows PowerShell as Administrator:
```powershell
wsl --install
```

**2. Install Java 8 & SSH**  
Open Ubuntu terminal and install the required dependencies:
```bash
sudo apt update
sudo apt install openjdk-8-jdk ssh -y
java -version
```

**3. Configure Passwordless SSH**  
Hadoop requires SSH to manage nodes without constant password prompts. Run these commands sequentially (press Enter for all prompts to leave passphrases blank):
```bash
sudo service ssh start
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 0600 ~/.ssh/authorized_keys
```

## Phase 2: Hadoop Installation & Configuration  
Download Hadoop 3.3.6 and configure it for a Pseudo-Distributed (single-node cluster) setup.

**1. Download and Extract Hadoop**
```bash
wget [https://archive.apache.org/dist/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz](https://archive.apache.org/dist/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz)
tar -xzvf hadoop-3.3.6.tar.gz
sudo mv hadoop-3.3.6 /usr/local/hadoop
```

**2. Set Environment Variables**  
Link Java and Hadoop to system path:
```bash
echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> ~/.bashrc
echo 'export HADOOP_HOME=/usr/local/hadoop' >> ~/.bashrc
echo 'export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin' >> ~/.bashrc
source ~/.bashrc
```

**3. Configure Hadoop XML Files**  
Link Java to Hadoop's internal settings:
```bash
echo "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> $HADOOP_HOME/etc/hadoop/hadoop-env.sh
```

Configure core-site.xml:
```bash
nano $HADOOP_HOME/etc/hadoop/core-site.xml
```

Paste this inside the <configuration> tags:
```xml
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://localhost:9000</value>
</property>
```

Configure hdfs-site.xml:
```bash
nano $HADOOP_HOME/etc/hadoop/hdfs-site.xml
```

Paste this inside the <configuration> tags:
```xml
<property>
    <name>dfs.replication</name>
    <value>1</value>
</property>
```

## Phase 3: Cluster Initialization  
Format the file system and start the Hadoop daemons.

**1. Format and Boot**
```bash
hdfs namenode -format
start-dfs.sh
start-yarn.sh
```

**2. System Health Check**
```bash
jps
```

## Phase 4: Application Development  
Create and compile the MapReduce Java application.

**1. Create the Workspace**
```bash
mkdir -p ~/hadoop-lab
cd ~/hadoop-lab
```

**2. The Java Source Code**  
WordCount.java file (included in this repository) contains the complete MapReduce logic. The Mapper class is specifically configured to convert all incoming text to lowercase and utilize Regular Expressions to strip all punctuation before the Shuffle and Sort phase.

**3. Compile and Package the Application**
```bash
hadoop com.sun.tools.javac.Main WordCount.java
jar cf wc.jar WordCount*.class
```

## Phase 5: Data Preparation & HDFS Deployment  
Download the real-world dataset and stage it in the Hadoop Distributed File System.  
**1. Download the Dataset**
```bash
wget [https://www.gutenberg.org/cache/epub/11/pg11.txt](https://www.gutenberg.org/cache/epub/11/pg11.txt) -O input.txt
```

**2. Upload to HDFS**
```bash
hdfs dfs -mkdir -p /user/student/input
hdfs dfs -put input.txt /user/student/input/
```

## Phase 6: Execution & Results  
Run the MapReduce program and analyze the output.

**1. Execute the Job**
```bash
hadoop jar wc.jar WordCount /user/student/input /user/student/output
```

**2. Verify the Results (Alphabetical Sort)**  
View the top 20 lines to prove the MapReduce framework successfully executed the "Shuffle and Sort" phase and cleaned punctuation:
```bash
hdfs dfs -cat /user/student/output/part-r-00000 | head -n 20
```

**3. Advanced Analytics (Frequency Sort)**  
Sort the output numerically to reveal the most frequently used words in the entire novel:
```bash
hdfs dfs -cat /user/student/output/part-r-00000 | sort -k2 -n -r | head -n 15
```
