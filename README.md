# Hadoop 3.3.6 Setup on Windows 11

A beginner-friendly guide to installing and configuring Apache Hadoop 3.3.6 on Windows 11 without Linux, WSL, or Docker.

## 1. Requirements

- Windows 10 or Windows 11
- Java JDK 8
- Apache Hadoop 3.3.6
- Git for Windows
- Hadoop Windows binaries (`winutils.exe`)

## 2. Download the Required Software

| Software | Download |
|---|---|
| Java JDK 8 (Temurin) | [Download Java 8](https://adoptium.net/temurin/releases/?version=8) |
| Hadoop 3.3.6 | [Download Hadoop 3.3.6](https://archive.apache.org/dist/hadoop/common/hadoop-3.3.6/hadoop-3.3.6.tar.gz) |
| Git for Windows | [Download Git](https://git-scm.com/downloads/win) |
| Hadoop Windows binaries | [GitHub - winutils](https://github.com/cdarlint/winutils) |

Install Git for Windows. Use Git Bash or Command Prompt for Git commands.



Open Command Prompt and run:

```cmd
git clone https://github.com/GANESH-NADKARNI/Hadoop-Windows-Setup.git
```

Move into the repository:

```cmd
cd Hadoop-Windows-Setup
```

To download the repository again later, use the same `git clone` command from a different folder.

## 3. Install Java

1. Install Java JDK 8.
2. Set the installation location to:

   `C:\Java`

3. Open Command Prompt and check:

```cmd
java -version
javac -version
```

Both commands should display Java version information.

## 4. Install Hadoop

1. Create this folder:

   `C:\Hadoop`

2. Extract the Hadoop archive.
3. Rename the extracted folder to:

   `hadoop-3.3.6`

Your Hadoop installation should be:

```text
C:\Hadoop\hadoop-3.3.6
```

## 5. Add Windows Hadoop Binaries

1. Open the downloaded Windows binaries repository.
2. Select the Hadoop 3.3.6 binaries.
3. Copy `winutils.exe` and `hadoop.dll`.
4. Place both files inside:

   `C:\Hadoop\hadoop-3.3.6\bin`

Expected locations:

```text
C:\Hadoop\hadoop-3.3.6\bin\winutils.exe
C:\Hadoop\hadoop-3.3.6\bin\hadoop.dll
```

## 6. Configure Environment Variables

Open:

**Windows Search → Edit the system environment variables → Environment Variables**

Under System Variables, create or edit:

| Variable | Value |
|---|---|
| `JAVA_HOME` | `C:\Java` |
| `HADOOP_HOME` | `C:\Hadoop\hadoop-3.3.6` |
| `HADOOP_CONF_DIR` | `C:\Hadoop\hadoop-3.3.6\etc\hadoop` |

Edit the `Path` variable and add:

```text
%JAVA_HOME%\bin
%HADOOP_HOME%\bin
%HADOOP_HOME%\sbin
```

Click OK to save the changes. Open a new Command Prompt.

Verify:

```cmd
echo %JAVA_HOME%
echo %HADOOP_HOME%
java -version
hadoop version
```

## 7. Configure Hadoop Files

Go to:

`C:\Hadoop\hadoop-3.3.6\etc\hadoop`

You will configure four XML files in this folder.

### A. core-site.xml

Open `core-site.xml` and replace its contents with:

```xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://0.0.0.0:19000</value>
    </property>
</configuration>
```

### B. hdfs-site.xml

Open `hdfs-site.xml` and replace its contents with:

```xml
<?xml version="1.0"?>
<configuration>
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

Open `mapred-site.xml`.

If it does not exist, copy `mapred-site.xml.template` in the same folder and rename the copy to `mapred-site.xml`.

Use:

```xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>
```

### D. yarn-site.xml

Open `yarn-site.xml` and replace its contents with:

```xml
<?xml version="1.0"?>
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

## 8. Configure Java in Hadoop

Open:

`C:\Hadoop\hadoop-3.3.6\etc\hadoop\hadoop-env.cmd`

Find the `JAVA_HOME` setting and set it to:

```cmd
set JAVA_HOME=C:\Java
```

Save the file.

## 9. Create Hadoop Data Folders

Open Command Prompt and execute:

```cmd
mkdir C:\Hadoop\hadoop-3.3.6\data
mkdir C:\Hadoop\hadoop-3.3.6\data\namenode
mkdir C:\Hadoop\hadoop-3.3.6\data\datanode
```

## 10. Format the NameNode

Execute:

```cmd
hdfs namenode -format
```

Wait for formatting to complete successfully.

**Important:** Run this only when initializing a new Hadoop installation. Do not format an existing cluster again, as existing HDFS data may become inaccessible.

## 11. Start Hadoop Services

Open Command Prompt:

```cmd
cd /d C:\Hadoop\hadoop-3.3.6\sbin
```

Start HDFS:

```cmd
start-dfs.cmd
```

Start YARN:

```cmd
start-yarn.cmd
```

## 12. Verify Hadoop Services

Open a new Command Prompt:

```cmd
jps
```

Expected Hadoop processes:

```text
NameNode
DataNode
ResourceManager
NodeManager
Jps
```

Open the Hadoop web interfaces:

| Service | URL |
|---|---|
| NameNode | http://localhost:9870 |
| ResourceManager | http://localhost:8088 |


## Author

**Ganesh Nadkarni**

GitHub: [GANESH-NADKARNI](https://github.com/GANESH-NADKARNI)

## Support

If this repository helped you set up Hadoop on Windows, please consider giving it a ⭐ star.

Your support is appreciated!