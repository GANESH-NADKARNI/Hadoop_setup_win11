# Hadoop 3.3.6 Setup on Windows 11

A beginner-friendly guide to installing and configuring Apache Hadoop 3.3.6 (single node, pseudo-distributed) on Windows 11 without Linux, WSL, or Docker.

This guide is the base for the other repositories in this series: **Hive**, **Spark**, and the **MapReduce student-marks** project all assume this exact Hadoop setup.

## 1. Requirements

- Windows 10 or Windows 11
- Java JDK 8
- Apache Hadoop 3.3.6
- Hadoop Windows binaries (`winutils.exe` and `hadoop.dll`, version 3.3.x)

## 2. Download the Required Software

| Software | Download |
|---|---|
| Java JDK 8 (Temurin) | [Download Java 8](https://adoptium.net/temurin/releases/?version=8) |
| Hadoop 3.3.6 | [Download Hadoop 3.3.6](https://archive.apache.org/dist/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz) |
| Hadoop Windows binaries | [GitHub - winutils](https://github.com/cdarlint/winutils) |

## 3. Install Java

1. Install Java JDK 8.
2. Use an installation path **without spaces**:

   `C:\Java`

3. Open Command Prompt and check:

```cmd
java -version
javac -version
```

## 4. Install Hadoop

1. Create the folder `C:\Hadoop`.
2. Extract the Hadoop archive into it.
3. Make sure the final path is:

```text
C:\Hadoop\hadoop-3.3.6
```

## 5. Add Windows Hadoop Binaries

Copy `winutils.exe` and `hadoop.dll` from the **hadoop-3.3.x** folder of the winutils repository into:

```text
C:\Hadoop\hadoop-3.3.6\bin\winutils.exe
C:\Hadoop\hadoop-3.3.6\bin\hadoop.dll
```

Missing or mismatched files here cause most of the strange start-up errors on Windows.

## 6. Configure Environment Variables

Open **Windows Search → Edit the system environment variables → Environment Variables**.

Under System Variables, create:

| Variable | Value |
|---|---|
| `JAVA_HOME` | `C:\Java` |
| `HADOOP_HOME` | `C:\Hadoop\hadoop-3.3.6` |
| `HADOOP_CONF_DIR` | `C:\Hadoop\hadoop-3.3.6\etc\hadoop` |

Edit `Path` and add:

```text
%JAVA_HOME%\bin
%HADOOP_HOME%\bin
%HADOOP_HOME%\sbin
```

Open a **new** Command Prompt and verify:

```cmd
echo %JAVA_HOME%
echo %HADOOP_HOME%
java -version
hadoop version
```

## 7. Configure Hadoop Files

Go to `C:\Hadoop\hadoop-3.3.6\etc\hadoop`. Replace the contents of the four files below.

### A. core-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- NameNode address. Hive, Spark and clients must use this exact URI. -->
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:19000</value>
    </property>

    <!-- Keep Hadoop temp files inside the install folder -->
    <property>
        <name>hadoop.tmp.dir</name>
        <value>C:/Hadoop/hadoop-3.3.6/data/tmp</value>
    </property>
</configuration>
```

Port `19000` is used here instead of the common `9000`. Any free port works, as long as every other tool uses the same URI. Use `localhost`, not `0.0.0.0`, as the address.

### B. hdfs-site.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- Single node: the default of 3 leaves every block under-replicated -->
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>

    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:///C:/Hadoop/hadoop-3.3.6/data/namenode</value>
    </property>

    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:///C:/Hadoop/hadoop-3.3.6/data/datanode</value>
    </property>
</configuration>
```

### C. mapred-site.xml

```xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>

    <!-- Required on Hadoop 3.x so YARN containers can find the MapReduce classes -->
    <property>
        <name>yarn.app.mapreduce.am.env</name>
        <value>HADOOP_MAPRED_HOME=C:/Hadoop/hadoop-3.3.6</value>
    </property>
    <property>
        <name>mapreduce.map.env</name>
        <value>HADOOP_MAPRED_HOME=C:/Hadoop/hadoop-3.3.6</value>
    </property>
    <property>
        <name>mapreduce.reduce.env</name>
        <value>HADOOP_MAPRED_HOME=C:/Hadoop/hadoop-3.3.6</value>
    </property>

    <!-- Absolute paths: %HADOOP_HOME% is not reliably expanded inside containers -->
    <property>
        <name>mapreduce.application.classpath</name>
        <value>C:/Hadoop/hadoop-3.3.6/share/hadoop/mapreduce/*,C:/Hadoop/hadoop-3.3.6/share/hadoop/mapreduce/lib/*,C:/Hadoop/hadoop-3.3.6/share/hadoop/common/*,C:/Hadoop/hadoop-3.3.6/share/hadoop/common/lib/*,C:/Hadoop/hadoop-3.3.6/share/hadoop/yarn/*,C:/Hadoop/hadoop-3.3.6/share/hadoop/yarn/lib/*,C:/Hadoop/hadoop-3.3.6/share/hadoop/hdfs/*,C:/Hadoop/hadoop-3.3.6/share/hadoop/hdfs/lib/*</value>
    </property>

    <!-- Small containers so jobs fit on a laptop -->
    <property><name>yarn.app.mapreduce.am.resource.mb</name><value>512</value></property>
    <property><name>mapreduce.map.memory.mb</name><value>512</value></property>
    <property><name>mapreduce.reduce.memory.mb</name><value>512</value></property>
</configuration>
```

### D. yarn-site.xml

```xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>

    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce_shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>

    <property>
        <name>yarn.nodemanager.local-dirs</name>
        <value>C:/Hadoop/hadoop-3.3.6/data/nm-local</value>
    </property>
    <property>
        <name>yarn.nodemanager.log-dirs</name>
        <value>C:/Hadoop/hadoop-3.3.6/data/nm-logs</value>
    </property>

    <property>
        <name>yarn.nodemanager.env-whitelist</name>
        <value>JAVA_HOME,HADOOP_COMMON_HOME,HADOOP_HDFS_HOME,HADOOP_CONF_DIR,CLASSPATH_PREPEND_DISTCACHE,HADOOP_YARN_HOME,HADOOP_MAPRED_HOME,HADOOP_HOME,PATH,LANG,TZ</value>
    </property>

    <!-- Resources. Lower memory-mb if your laptop has 8 GB RAM or less free. -->
    <property><name>yarn.nodemanager.resource.memory-mb</name><value>4096</value></property>
    <property><name>yarn.nodemanager.resource.cpu-vcores</name><value>4</value></property>
    <property><name>yarn.scheduler.minimum-allocation-mb</name><value>256</value></property>
    <property><name>yarn.scheduler.maximum-allocation-mb</name><value>4096</value></property>

    <!-- Disable YARN memory checks (simplifies running on a laptop) -->
    <property><name>yarn.nodemanager.vmem-check-enabled</name><value>false</value></property>
    <property><name>yarn.nodemanager.pmem-check-enabled</name><value>false</value></property>
</configuration>
```

## 8. Configure Java in Hadoop

Open `C:\Hadoop\hadoop-3.3.6\etc\hadoop\hadoop-env.cmd`, find the `JAVA_HOME` line and set:

```cmd
set JAVA_HOME=C:\Java
```

## 9. Create Hadoop Data Folders

```cmd
mkdir C:\Hadoop\hadoop-3.3.6\data
mkdir C:\Hadoop\hadoop-3.3.6\data\namenode
mkdir C:\Hadoop\hadoop-3.3.6\data\datanode
mkdir C:\Hadoop\hadoop-3.3.6\data\tmp
mkdir C:\Hadoop\hadoop-3.3.6\data\nm-local
mkdir C:\Hadoop\hadoop-3.3.6\data\nm-logs
```

## 10. Format the NameNode

```cmd
hdfs namenode -format -force
```

Wait until the output ends with `has been successfully formatted`. Do not type anything while it runs.

**Important:** Format only on a brand-new installation. Formatting again creates a new cluster ID, and the old DataNode data no longer matches it. See `troubleshoot/README.md` before formatting again.

## 11. Start Hadoop Services

```cmd
cd /d C:\Hadoop\hadoop-3.3.6\sbin
start-dfs.cmd
start-yarn.cmd
```

## 12. Verify Hadoop Services

```cmd
jps
```

Expected processes:

```text
NameNode
DataNode
ResourceManager
NodeManager
Jps
```

Check that the DataNode registered:

```cmd
hdfs dfsadmin -report
```

It should show `Live datanodes (1)`. Safe mode turns off by itself about 30 seconds after the DataNode registers.

Web interfaces:

| Service | URL |
|---|---|
| NameNode | http://localhost:9870 |
| ResourceManager | http://localhost:8088 |

Quick HDFS test:

```cmd
hdfs dfs -mkdir -p /test
hdfs dfs -ls /
```

## 13. Stop Hadoop Services

```cmd
cd /d C:\Hadoop\hadoop-3.3.6\sbin
stop-yarn.cmd
stop-dfs.cmd
jps
```

## 14. Troubleshooting

If a service fails to start or a command returns an error, see [troubleshoot/README.md](troubleshoot/README.md).

## Repository Structure

```text
Hadoop-Windows-Setup/
│
├── README.md
│
└── troubleshoot/
    └── README.md
```

## Author

**Ganesh Nadkarni**

GitHub: [GANESH-NADKARNI](https://github.com/GANESH-NADKARNI)

## Support

If this repository helped you set up Hadoop on Windows, please consider giving it a ⭐ star.