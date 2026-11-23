# Hadoop 3.3.6 on Windows – Troubleshooting Guide

This guide covers common problems encountered while installing and running Apache Hadoop 3.3.6 on Windows 10/11.

Before troubleshooting, check that Hadoop and Java are installed correctly.

## 1. `java` is not recognized

**Error:**

```text
'java' is not recognized as an internal or external command
```

**Reason:**

Java is not installed correctly or its `bin` folder is missing from the system `Path`.

**Solution:**

1. Check that Java is installed in `C:\Java`.
2. Open Environment Variables.
3. Set:

```text
JAVA_HOME = C:\Java
```

4. Add this to `Path`:

```text
%JAVA_HOME%\bin
```

5. Open a new Command Prompt and execute:

```cmd
java -version
javac -version
```

## 2. `hadoop` is not recognized

**Error:**

```text
'hadoop' is not recognized as an internal or external command
```

**Reason:**

Hadoop environment variables are missing or incorrect.

**Solution:**

Set the following environment variables:

```text
HADOOP_HOME = C:\Hadoop\hadoop-3.3.6
HADOOP_CONF_DIR = C:\Hadoop\hadoop-3.3.6\etc\hadoop
```

Add these entries to `Path`:

```text
%HADOOP_HOME%\bin
%HADOOP_HOME%\sbin
```

Open a new Command Prompt and run:

```cmd
hadoop version
```

## 3. `winutils.exe` not found

**Error:**

```text
Could not locate executable null\bin\winutils.exe
```

**Reason:**

The Windows Hadoop binary is missing or stored in the wrong folder.

**Solution:**

1. Download the Windows binaries matching Hadoop 3.3.6 from [cdarlint/winutils](https://github.com/cdarlint/winutils).
2. Copy `winutils.exe` and `hadoop.dll`.
3. Place them inside:

```text
C:\Hadoop\hadoop-3.3.6\bin
```

Verify:

```cmd
dir C:\Hadoop\hadoop-3.3.6\bin\winutils.exe
```

## 4. `HADOOP_HOME` or `hadoop.home.dir` is unset

**Error:**

```text
HADOOP_HOME and hadoop.home.dir are unset
```

**Reason:**

The Hadoop environment variables are not available to the process starting Hadoop.

**Solution:**

1. Set `HADOOP_HOME` to:

```text
C:\Hadoop\hadoop-3.3.6
```

2. Check that `Path` contains:

```text
%HADOOP_HOME%\bin
```

3. Close and reopen Command Prompt.
4. Verify:

```cmd
echo %HADOOP_HOME%
hadoop version
```

## 5. NameNode fails to start

**Possible errors:**

```text
NameNode is not starting
```

or

```text
Incompatible clusterIDs
```

**Possible reasons:**

- Incorrect NameNode directory configuration.
- NameNode and DataNode metadata have mismatched cluster IDs.
- Another NameNode process is already running.

**Solution:**

1. Check `hdfs-site.xml`.
2. Verify the NameNode and DataNode storage directory paths.
3. Check whether Hadoop processes are already running:

```cmd
jps
```

4. Check the Hadoop logs for the exact error.

**Important:** Do not delete the NameNode or DataNode directories or format the NameNode on an installation that holds HDFS data you need. On a fresh setup with nothing to keep, see section 24.

## 6. DataNode is not starting

**Possible reasons:**

- DataNode storage directory is incorrect.
- Directory permissions are insufficient.
- NameNode and DataNode cluster IDs do not match.

**Solution:**

1. Check the `dfs.datanode.data.dir` property in `hdfs-site.xml`.
2. Confirm the configured directory exists.
3. Check the DataNode logs for errors.
4. Confirm the NameNode is running.
5. If the log says `current directory is not empty` or `Storage directory ... does not exist`, see section 17.

Check processes:

```cmd
jps
```

## 7. HDFS connection refused

**Error:**

```text
Connection refused
```

**Reason:**

The NameNode is not running, or the configured HDFS address and port are incorrect.

**Solution:**

1. Start HDFS:

```cmd
start-dfs.cmd
```

2. Check `fs.defaultFS` in `core-site.xml`.
3. Verify that the NameNode is running:

```cmd
jps
```

4. Try:

```cmd
hdfs dfs -ls /
```

## 8. Incorrect HDFS endpoint configuration

**Error:**

```text
The endpoint configuration is wrong
```

**Possible reasons:**

- Hadoop configuration files are incorrect.
- The NameNode is not running.
- The client is using a different Hadoop configuration directory.

**Solution:**

Check the following:

```cmd
echo %HADOOP_CONF_DIR%
```

Expected:

```text
C:\Hadoop\hadoop-3.3.6\etc\hadoop
```

Check `core-site.xml` and confirm the `fs.defaultFS` value matches your NameNode configuration.

## 9. NameNode format fails

**Possible reasons:**

- Incorrect Java configuration.
- Invalid storage directory.
- Permission problems.

**Solution:**

1. Check Java:

```cmd
java -version
```

2. Check the NameNode directory configured in `hdfs-site.xml`.
3. Confirm the directory exists and is writable.
4. Review the NameNode logs.

**Important:** Only format a new Hadoop installation. Never use formatting as a general troubleshooting step on an existing cluster.

## 10. Cannot create an HDFS directory

**Error:**

```text
Permission denied
```

**Reason:**

The current HDFS user does not have permission to create files or directories at the requested location.

**Solution:**

Check the HDFS directory permissions:

```cmd
hdfs dfs -ls /
```

Create a directory in a location where your HDFS user has write permission:

```cmd
hdfs dfs -mkdir -p /student_data
```

## 11. ResourceManager or NodeManager is not starting

**Possible reasons:**

- YARN configuration is incorrect.
- Java configuration is incorrect.
- Another process is using the required port.

**Solution:**

Check `yarn-site.xml` and confirm the following property is configured:

```xml
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
```

Start YARN:

```cmd
start-yarn.cmd
```

Verify:

```cmd
jps
```

Look for:

```text
ResourceManager
NodeManager
```

## 12. `jps` does not show Hadoop processes

**Reason:**

Hadoop services may not have started successfully.

**Solution:**

Start HDFS:

```cmd
start-dfs.cmd
```

Start YARN:

```cmd
start-yarn.cmd
```

Check again:

```cmd
jps
```

If processes are still missing, inspect the Hadoop logs for the actual startup error.

## 13. Hadoop Web UI is not opening

**NameNode UI:**

http://localhost:9870

**ResourceManager UI:**

http://localhost:8088

**Possible reasons:**

- Hadoop services are not running.
- The port is already in use.
- A firewall or network setting is blocking access.

**Solution:**

1. Verify the relevant Java process using `jps`.
2. Check the Hadoop logs.
3. Confirm that the web interface port is available.

## 14. Output directory already exists

**Error:**

```text
FileAlreadyExistsException
```

**Reason:**

Hadoop MapReduce does not normally overwrite an existing output directory.

**Solution:**

Check the output directory:

```cmd
hdfs dfs -ls /
```

If the old output is no longer required, remove it:

```cmd
hdfs dfs -rm -r /output
```

Run the MapReduce job again.

**Warning:** This command permanently deletes the specified HDFS output directory.

## 15. `python` is not recognized

**Error:**

```text
'python' is not recognized as an internal or external command
```

**Reason:**

Python is not installed or its installation directory is missing from `Path`.

**Solution:**

1. Install Python from [python.org](https://www.python.org/downloads/).
2. Enable the option to add Python to `Path` during installation.
3. Open a new Command Prompt.
4. Verify:

```cmd
python --version
```

## 16. Hadoop Streaming cannot find mapper or reducer

**Possible error:**

```text
No such file or directory
```

**Reason:**

The mapper or reducer script path is incorrect, or the files were not distributed to the worker processes.

**Solution:**

- Check that the mapper and reducer files exist.
- Verify the paths passed to the Hadoop Streaming command.
- Use the `-files` option to distribute scripts.

Example:

```cmd
-files "file:///C:/data/mapper.py#mapper.py,file:///C:/data/reducer.py#reducer.py"
```

Use the correct local paths for your scripts.

## 17. DataNode exits: "current directory is not empty" or "Storage directory does not exist"

**Errors (in the DataNode log):**

```text
Directory C:\Hadoop\hadoop-3.3.6\data\datanode is in an inconsistent state:
Can't format the storage directory because the current directory is not empty.
```

or

```text
Storage directory for location [DISK]file:/C:/Hadoop/hadoop-3.3.6/data/datanode
and block pool id BP-... does not exist
```

**Reason:**

The NameNode was formatted again, which gives it a new cluster ID and block pool. The DataNode folder still holds data from the old format (or was only half deleted), so the two no longer match and the DataNode refuses to start.

**Solution (fresh setup, no HDFS data to keep):**

Follow section 24.

A faster alternative is to point the DataNode at a new empty folder. Any unused folder name works (`datanode1`, `datanode2`, ...):

```cmd
mkdir C:\Hadoop\hadoop-3.3.6\data\datanode1
```

In `hdfs-site.xml`:

```xml
<property>
    <name>dfs.datanode.data.dir</name>
    <value>file:///C:/Hadoop/hadoop-3.3.6/data/datanode1</value>
</property>
```

Then format and start:

```cmd
hdfs namenode -format -force
start-dfs.cmd
```

The new folder must not already exist with old files in it.

## 18. `rmdir` says "Access is denied" or "being used by another process"

**Errors:**

```text
C:\Hadoop\hadoop-3.3.6\data\datanode\current\BP-909~1.169 - Access is denied.
The process cannot access the file because it is being used by another process.
```

**Reason:**

- A NameNode or DataNode is still running and holds the folder.
- The Command Prompt is not running as Administrator.
- Folder permissions were damaged by an earlier crash.

**Solution:**

1. Check for leftover processes and stop them:

```cmd
jps
taskkill /F /PID <pid shown by jps>
```

2. Open Command Prompt **as Administrator** (right-click → Run as administrator).
3. Take ownership and grant access:

```cmd
takeown /f C:\Hadoop\hadoop-3.3.6\data /r /d y
icacls C:\Hadoop\hadoop-3.3.6\data /grant "%USERNAME%":(OI)(CI)F /T
```

4. If deleting still fails, rename the folder instead. Renaming only needs rights on the parent folder:

```cmd
ren C:\Hadoop\hadoop-3.3.6\data\datanode datanode_old
mkdir C:\Hadoop\hadoop-3.3.6\data\datanode
```

5. If it is still locked, restart Windows and run the delete as the first command.

## 19. `hdfs namenode -format` asks "(Y or N)" and ignores your commands

**Error:**

```text
Re-format filesystem in Storage Directory root= C:\Hadoop\hadoop-3.3.6\data\namenode ... ? (Y or N)
Invalid input: start-dfs.cmd
```

or

```text
java.io.IOException: Cannot remove current directory: C:\Hadoop\hadoop-3.3.6\data\namenode\current
```

**Reason:**

- The prompt waits for `Y` or `N`. Anything else you type, such as `start-dfs.cmd`, is rejected and is **not** run.
- The "Cannot remove current directory" error means the old NameNode is still running and has the folder locked.

**Solution:**

1. Stop the NameNode first (`jps`, then `taskkill`, see section 20).
2. Format without the prompt:

```cmd
hdfs namenode -format -force
```

3. Wait for `has been successfully formatted` before typing anything else.

## 20. `stop-dfs.cmd` says "No tasks running" but `jps` still lists processes

**Output:**

```text
INFO: No tasks running with the specified criteria.
```

**Reason:**

The Windows stop scripts look for processes by window title. They do not find a NameNode or DataNode whose window was closed or opened differently.

**Solution:**

```cmd
jps
taskkill /F /PID <pid shown by jps>
jps
```

Repeat until `jps` lists only `Jps`. Avoid `taskkill /F /IM java.exe` unless you are happy to stop every Java application.

## 21. `hdfs dfsadmin -report` shows "Safe mode is ON" and capacity 0

**Output:**

```text
Safe mode is ON
Configured Capacity: 0 (0 B)
```

**Reason:**

No DataNode has registered with the NameNode, so the NameNode stays in safe mode.

**Solution:**

1. Run `jps` and check that a DataNode is running.
2. If it is missing, read its log in `C:\Hadoop\hadoop-3.3.6\logs` and see sections 17 and 18.
3. Once the DataNode registers, safe mode turns off by itself after about 30 seconds. Check:

```cmd
hdfs dfsadmin -report
```

It should show `Live datanodes (1)`.

## 22. Job fails: "Could not find or load main class ...MRAppMaster"

**Error:**

```text
Could not find or load main class org.apache.hadoop.mapreduce.v2.app.MRAppMaster
```

**Reason:**

Hadoop 3.x needs `HADOOP_MAPRED_HOME` passed to the YARN containers.

**Solution:**

Add these properties to `mapred-site.xml`:

```xml
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
```

Make sure `yarn.nodemanager.env-whitelist` in `yarn-site.xml` includes `HADOOP_MAPRED_HOME,HADOOP_HOME,PATH,LANG,TZ`. Then restart YARN:

```cmd
stop-yarn.cmd
start-yarn.cmd
```

## 23. Container killed: "running beyond virtual memory limits"

**Error:**

```text
Container ... is running beyond virtual memory limits
```

**Reason:**

YARN's memory checks are strict for small containers on a laptop.

**Solution:**

Add to `yarn-site.xml`:

```xml
<property>
    <name>yarn.nodemanager.vmem-check-enabled</name>
    <value>false</value>
</property>
<property>
    <name>yarn.nodemanager.pmem-check-enabled</name>
    <value>false</value>
</property>
```

Restart YARN.

## 24. Re-formatting HDFS safely

Use this only when you must run `hdfs namenode -format` again. **This deletes all HDFS data.**

1. Stop everything and confirm `jps` lists only `Jps` (see section 20).
2. Open Command Prompt **as Administrator**.
3. Delete both data folders and recreate them:

```cmd
rmdir /s /q C:\Hadoop\hadoop-3.3.6\data\datanode
rmdir /s /q C:\Hadoop\hadoop-3.3.6\data\namenode
mkdir C:\Hadoop\hadoop-3.3.6\data\datanode
mkdir C:\Hadoop\hadoop-3.3.6\data\namenode
```

If a delete fails, see section 18.

4. Format and start:

```cmd
hdfs namenode -format -force
start-dfs.cmd
jps
hdfs dfsadmin -report
```

Always clear the DataNode folder **before** re-formatting. The NameNode and DataNode must come from the same format.

## 25. Harmless warnings in the logs

These messages can be ignored:

```text
WARN server.AuthenticationFilter: Unable to initialize FileSignerSecretProvider
WARN web.DatanodeHttpServer: Got null for restCsrfPreventionFilter
DEPRECATED: Use of this script to execute hdfs command is deprecated.
```

## 26. Check Hadoop Logs

When Hadoop fails to start or a job fails, check the logs before changing configuration files.

Default log directory:

```text
C:\Hadoop\hadoop-3.3.6\logs
```

List log files:

```cmd
dir C:\Hadoop\hadoop-3.3.6\logs
```

Search for common errors:

```cmd
findstr /S /I "ERROR Exception FAILED" C:\Hadoop\hadoop-3.3.6\logs\*.log
```

Read the error message and its surrounding lines to identify the actual cause.

---

## Useful Verification Commands

| Purpose | Command |
|---|---|
| Check Java | `java -version` |
| Check Hadoop | `hadoop version` |
| Check Java processes | `jps` |
| List HDFS files | `hdfs dfs -ls /` |
| Check HDFS health | `hdfs dfsadmin -report` |
| Check Hadoop home | `echo %HADOOP_HOME%` |
| Check Java home | `echo %JAVA_HOME%` |

## Author

**Ganesh Nadkarni**

GitHub: [GANESH-NADKARNI](https://github.com/GANESH-NADKARNI)

## Support

If this troubleshooting guide helped you solve a Hadoop issue, please consider giving the repository a ⭐ star on GitHub.

If you find a new issue or have a better solution, contributions are welcome!