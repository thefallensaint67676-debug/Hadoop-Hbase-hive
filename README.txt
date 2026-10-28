# Apache Hadoop 3.3.6 on Windows 10/11 -- Setup Guide

This guide collects the configuration shown in the screenshots and adds
the commands needed to start, test, and stop a local Hadoop
pseudo-distributed setup.

> **Paths used in this guide**
>
> - Hadoop folder: `C:\hadoop-3.3.6`
> - Hadoop configuration folder: `C:\hadoop-3.3.6\etc\hadoop`
> - Hadoop data folders: `C:\hadoop-3.3.6\data\namenode` and
> `C:\hadoop-3.3.6\data\datanode`
>
> If your Hadoop folder is somewhere else, replace these paths
> everywhere. Run commands in **Command Prompt (cmd)** unless a step
> says otherwise.

## 1. Prerequisites

1. Install a supported 64-bit JDK. Hadoop 3.3.6 is commonly used with
 JDK 8 or 11 on Windows; use the JDK version prescribed by your
 lab/tutorial if it differs.
2. Set `JAVA_HOME` to the JDK installation directory, for example:
 `C:\Program Files\Java\jdk-11`
3. Download/extract Hadoop 3.3.6 so that the folder is
 `C:\hadoop-3.3.6`.
4. Obtain Windows Hadoop native helper binaries (`winutils.exe`, and
 matching Hadoop DLLs if required) built for the same Hadoop version.
 The screenshots use the `cdarlint/winutils` GitHub repository,
 folder `hadoop-3.3.6/bin`. Use trusted sources and avoid mixing
 binaries from different Hadoop versions.

Check Java and Hadoop from a **new** Command Prompt after setting
environment variables:

``` bat
java -version
echo %JAVA_HOME%
```

## 2. Configure environment variables

Open **Start → Edit the system environment variables → Environment
Variables**.

Create or update the following variables (use the actual JDK folder on
your PC):

 -------------------------------------------------------------------------
 Variable Value
 ----------------------------------- -------------------------------------
 `JAVA_HOME` Your JDK installation directory,
 e.g. `C:\Program Files\Java\jdk-11`

 `HADOOP_HOME` `C:\hadoop-3.3.6`
 -------------------------------------------------------------------------

Edit the **Path** variable and add these separate entries:

``` text
%JAVA_HOME%\bin
%HADOOP_HOME%\bin
%HADOOP_HOME%\sbin
```

The screenshot shows the two Hadoop entries as:

``` text
C:\hadoop-3.3.6\bin
C:\hadoop-3.3.6\sbin
```

Click **OK** on all dialogs. Close and reopen Command Prompt so it
receives the updated environment.

Verify:

``` bat
echo %HADOOP_HOME%
where java
java -version
hadoop version
```

If `hadoop` is not recognized, check that `%HADOOP_HOME%\bin` exists and
is in `Path`.

## 3. Add Windows helper binaries

The screenshot points to the GitHub repository
`https://github.com/cdarlint/winutils` and its `hadoop-3.3.6/bin`
folder.

1. Download the matching `hadoop-3.3.6/bin` files from a source you
 trust.
2. Copy the required files (typically `winutils.exe` and any matching
 Hadoop DLLs supplied with that build) into: `C:\hadoop-3.3.6\bin`
3. Do not replace Hadoop's Java JARs with files from another version.

Verify that the files are present:

``` bat
dir C:\hadoop-3.3.6\bin\winutils.exe
```

If this file is absent, some Windows Hadoop commands may fail with a
native-helper error.

## 4. Create the data directories

Create the two directories shown in the screenshots. In Command Prompt:

``` bat
mkdir C:\hadoop-3.3.6\data
mkdir C:\hadoop-3.3.6\data\namenode
mkdir C:\hadoop-3.3.6\data\datanode
```

If the folders already exist, `mkdir` may report that they already
exist; that is fine.

## 5. Configure Hadoop XML files

Edit the files inside `C:\hadoop-3.3.6\etc\hadoop`. Preserve the
existing XML header/comments and ensure each file has one outer
`<configuration> ... </configuration>` element.

### 5.1 `core-site.xml`

File: `C:\hadoop-3.3.6\etc\hadoop\core-site.xml`

Set the default filesystem to the local NameNode:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <property>
 <name>fs.defaultFS</name>
 <value>hdfs://localhost:9000</value>
 </property>
</configuration>
```

### 5.2 `hdfs-site.xml`

File: `C:\hadoop-3.3.6\etc\hadoop\hdfs-site.xml`

The screenshots use replication factor `1` and the following local data
paths:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <property>
 <name>dfs.replication</name>
 <value>1</value>
 </property>

 <property>
 <name>dfs.namenode.name.dir</name>
 <value>file:///C:/hadoop-3.3.6/data/namenode</value>
 </property>

 <property>
 <name>dfs.datanode.data.dir</name>
 <value>file:///C:/hadoop-3.3.6/data/datanode</value>
 </property>
</configuration>
```

Use forward slashes in the `file:///` URIs. Ensure these folders match
the directories created in Step 4.

### 5.3 `mapred-site.xml`

File: `C:\hadoop-3.3.6\etc\hadoop\mapred-site.xml`

The screenshot sets MapReduce to use YARN:

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <property>
 <name>mapreduce.framework.name</name>
 <value>yarn</value>
 </property>
</configuration>
```

If the file is named `mapred-site.xml.template`, copy it to
`mapred-site.xml` first, then edit `mapred-site.xml`.

### 5.4 `yarn-site.xml`

File: `C:\hadoop-3.3.6\etc\hadoop\yarn-site.xml`

The screenshots configure the MapReduce shuffle service:

``` xml
<?xml version="1.0"?>
<configuration>
 <property>
 <name>yarn.nodemanager.aux-services</name>
 <value>mapreduce_shuffle</value>
 </property>

 <property>
 <name>yarn.nodemanager.auxservices.mapreduce.shuffle.class</name>
 <value>org.apache.hadoop.mapred.ShuffleHandler</value>
 </property>
</configuration>
```

### XML checks

- Each opening `<property>` must have a closing `</property>`.
- Each `<name>` must have a matching `</name>`, and each `<value>` a
 matching `</value>`.
- Do not put properties outside the `<configuration>` element.
- Save every file after editing.
- The `mapreduce.framework.name` value is `yarn` (not `yarn/`).

## 6. Configure `hadoop-env.cmd`

Open:

`C:\hadoop-3.3.6\etc\hadoop\hadoop-env.cmd`

Find or add the `JAVA_HOME` setting. Use the actual installed JDK path,
with no trailing `\bin`, for example:

``` bat
set JAVA_HOME=C:\Program Files\Java\jdk-11
```

If your JDK path contains spaces and the script has trouble, use the
exact syntax recommended by your JDK/Hadoop distribution; do not point
`JAVA_HOME` to the `bin` folder.

## 7. Format the NameNode --- first-time setup only

Open **Command Prompt as Administrator** if your lab setup requires it,
then run:

``` bat
cd /d C:\hadoop-3.3.6
hdfs namenode -format
```

Read the output and confirm the format completes successfully before
continuing.

**Important:** Format the NameNode only when initializing a fresh HDFS
setup. Re-formatting an existing NameNode can destroy its HDFS namespace
metadata and make existing HDFS files inaccessible.

## 8. Start Hadoop services

Open a Command Prompt and run:

``` bat
cd /d C:\hadoop-3.3.6\sbin
start-dfs.cmd
```

Wait for the NameNode and DataNode windows/services to start. Then open
another Command Prompt and run:

``` bat
cd /d C:\hadoop-3.3.6\sbin
start-yarn.cmd
```

The screenshot shows `start-all.cmd` printing a deprecation message and
recommending `start-dfs.cmd` and `start-yarn.cmd`; use those two
commands instead.

To inspect Java processes:

``` bat
jps
```

A working pseudo-distributed setup commonly shows processes such as
`NameNode`, `DataNode`, `SecondaryNameNode`, `ResourceManager`, and
`NodeManager`. Which processes appear depends on which services were
started and whether they started successfully.

## 9. Verify the web interfaces

Open these addresses in a browser:

 Service URL
 ------------------------- -------------------------
 HDFS NameNode UI `http://localhost:9870`
 YARN ResourceManager UI `http://localhost:8088`

If a page does not load, check the terminal output and logs under
`C:\hadoop-3.3.6\logs`.

## 10. Basic HDFS commands for the setup guide

Run from any Command Prompt where Hadoop is configured in `Path`.

``` bat
hdfs dfs -mkdir -p /user/%USERNAME%
hdfs dfs -ls /
hdfs dfs -ls /user
```

Create a local text file:

``` bat
echo Hello Hadoop > input.txt
```

Upload it to HDFS:

``` bat
hdfs dfs -mkdir -p /input
hdfs dfs -put -f input.txt /input/
hdfs dfs -ls /input
hdfs dfs -cat /input/input.txt
```

Download a file from HDFS:

``` bat
hdfs dfs -get /input/input.txt downloaded.txt
```

Remove the example HDFS input folder when finished:

``` bat
hdfs dfs -rm -r /input
```

## 11. Stop Hadoop services

Use a Command Prompt:

``` bat
cd /d C:\hadoop-3.3.6\sbin
stop-yarn.cmd
stop-dfs.cmd
```

Wait for the service windows to close. If a process remains, inspect it
with `jps` and review the Hadoop logs before terminating anything
manually.

## 12. Troubleshooting checklist

### `hadoop` or `hdfs` is not recognized

- Confirm `C:\hadoop-3.3.6\bin` and `C:\hadoop-3.3.6\sbin` exist.
- Confirm both are in `Path`.
- Open a new Command Prompt after changing environment variables.
- Run `echo %HADOOP_HOME%` and `hadoop version`.

### `JAVA_HOME` is not set or Java cannot be found

- Check `java -version` and `echo %JAVA_HOME%`.
- Set `JAVA_HOME` to the JDK root folder, not its `bin` subfolder.
- Check `C:\hadoop-3.3.6\etc\hadoop\hadoop-env.cmd`.
- Use a JDK version compatible with the Hadoop build used by your lab.

### `winutils.exe` or native Hadoop binary error

- Confirm the Windows helper binaries are in `C:\hadoop-3.3.6\bin`.
- Make sure the binaries match Hadoop 3.3.6 and your Windows
 architecture.
- Do not download arbitrary DLLs from unknown sites.

### NameNode format fails or Hadoop cannot write to data folders

- Check the XML file paths and ensure `C:\hadoop-3.3.6\data\namenode`
 and `...\datanode` exist.
- Check folder permissions.
- Read the exact error and logs before trying to format again.
- Do not delete the NameNode data directory or re-format an existing
 setup unless you intend to reset HDFS.

### Port already in use

- Another Hadoop instance may still be running. Check `jps` and close
 it cleanly with the stop scripts.
- Check whether another program is using ports `9000`, `9870`, or
 `8088`.

### NameNode UI opens but HDFS commands fail

- Confirm `start-dfs.cmd` started successfully.
- Confirm `fs.defaultFS` is `hdfs://localhost:9000`.
- Check the NameNode/DataNode terminal output and logs.

### YARN services do not start

- Confirm `mapred-site.xml` has `mapreduce.framework.name` set to
 `yarn`.
- Confirm `yarn-site.xml` contains the two properties shown in Step
 5.4.
- Start HDFS first, then run `start-yarn.cmd`.

## 13. Quick practical-exam sequence

For a machine already configured and formatted:

``` bat
start-dfs.cmd
start-yarn.cmd
jps
hdfs dfs -mkdir -p /input
echo Hello Hadoop > input.txt
hdfs dfs -put -f input.txt /input/
hdfs dfs -ls /input
hdfs dfs -cat /input/input.txt
```

When finished:

``` bat
stop-yarn.cmd
stop-dfs.cmd
```

For a **new** installation, complete the prerequisites, environment
variables, Windows helper binaries, data folders, XML configuration, and
one-time NameNode format before starting services.

## Reference from the screenshots

- Hadoop installation path shown: `C:\hadoop-3.3.6`
- Configuration path shown: `C:\hadoop-3.3.6\etc\hadoop`
- Windows helper binaries repository shown:
 `https://github.com/cdarlint/winutils`
- HDFS NameNode format command shown: `hdfs namenode -format`
- Start script warning shown: `start-all.cmd` is deprecated; use
 `start-dfs.cmd` and `start-yarn.cmd`.


---

# 14. Apache HBase 1.4.9 on Windows Setup

This section adds the HBase setup shown in the additional screenshots. It assumes Hadoop is installed at `C:\hadoop-3.3.6` and HBase will be extracted to `C:\hbase`.

> **Compatibility note:** HBase 1.4.9 is an old release. It may not work reliably with Hadoop 3.3.6. Use the exact HBase/Hadoop versions prescribed by your lab. The steps below document the screenshot-based setup, but if startup fails, version compatibility is one of the first things to check.

## 14.1 Download and extract HBase

The screenshots use HBase 1.4.9:

`https://archive.apache.org/dist/hbase/1.4.9/`

1. Download the appropriate HBase 1.4.9 binary archive from the Apache archive.
2. Extract it so the installation directory is:
 `C:\hbase`
3. Confirm that these paths exist:
 - `C:\hbase\bin`
 - `C:\hbase\conf`
 - `C:\hbase\conf\hbase-site.xml`
 - `C:\hbase\conf\hbase-env.cmd`

## 14.2 Set HBase environment variables

Open **Environment Variables** in Windows.

Create:

| Variable | Value |
|---|---|
| `HBASE_HOME` | `C:\hbase` |

Add this entry to `Path`:

```text
%HBASE_HOME%\bin
```

Keep the Hadoop entries from the earlier section in `Path` as well:

```text
%HADOOP_HOME%\bin
%HADOOP_HOME%\sbin
```

Open a new Command Prompt and verify:

```bat
echo %HBASE_HOME%
hbase version
```

## 14.3 Configure `hbase-site.xml`

Open:

`C:\hbase\conf\hbase-site.xml`

Keep the existing XML header and `<configuration>` wrapper, then add the properties below inside `<configuration>`. Do not add a second `<configuration>` element if the file already has one.

```xml
<property>
 <name>hbase.rootdir</name>
 <value>hdfs://localhost:9000/hbase</value>
</property>

<property>
 <name>hbase.zookeeper.property.dataDir</name>
 <value>C:\zookeeper</value>
</property>

<property>
 <name>hbase.cluster.distributed</name>
 <value>false</value>
</property>
```

**Important corrections and notes:**

- The screenshot shows `hbase.rootdir` as `C:\Hfile` in one place and `hdfs://localhost:9820/hbase` in another. These are alternative storage choices, not settings to use simultaneously. Because the Hadoop guide above configures `fs.defaultFS` as `hdfs://localhost:9000`, this guide uses `hdfs://localhost:9000/hbase` so the ports match.
- If your Hadoop NameNode is intentionally configured to use port `9820`, use that port consistently in both Hadoop's `core-site.xml` and HBase's `hbase.rootdir`. Do not mix `9000` and `9820`.
- `hbase.cluster.distributed=false` configures HBase standalone mode. In that mode HBase manages its own local services; the `hbase.rootdir` should normally be a local path such as `file:///C:/Hfile` if your lab specifically expects local storage. If your lab expects HBase data in HDFS, follow its prescribed distributed-mode configuration instead.
- The screenshot's HBase setup uses `hbase.cluster.distributed=false` while also showing an HDFS root directory. That combination is potentially inconsistent depending on the exact HBase release and lab setup. Do not switch to distributed mode unless your instructor's configuration calls for it.

For a **standalone/local HBase setup** matching `hbase.cluster.distributed=false`, use this alternative root directory instead:

```xml
<property>
 <name>hbase.rootdir</name>
 <value>file:///C:/Hfile</value>
</property>
```

Choose one `hbase.rootdir` value only. Create the corresponding folder if needed:

```bat
mkdir C:\Hfile
mkdir C:\zookeeper
```

## 14.4 Configure `hbase-env.cmd`

Open:

`C:\hbase\conf\hbase-env.cmd`

Find the `JAVA_HOME` setting and point it to the installed **JDK 8** folder if that is what your lab requires. The screenshot uses the Windows short-path form:

```bat
set JAVA_HOME=C:\Progra~1\Java\jdk-1.8
```

That path is only an example; it must match the actual JDK folder on your computer. You can check Java with:

```bat
java -version
echo %JAVA_HOME%
```

Do not set `JAVA_HOME` to the `bin` directory. If your JDK is installed somewhere else, update the path accordingly.

## 14.5 Start Hadoop first

If HBase is configured to store its data in HDFS, Hadoop must be running first. Start HDFS and YARN from the Hadoop `sbin` directory:

```bat
cd /d C:\hadoop-3.3.6\sbin
start-dfs.cmd
start-yarn.cmd
jps
```

For standalone HBase using a local `file:///` root directory, HDFS is not necessarily required for HBase's local storage, but follow the sequence expected by your practical lab.

## 14.6 Start HBase

Open a new Command Prompt:

```bat
cd /d C:\hbase\bin
start-hbase.cmd
```

Wait for the HBase services to initialize. Check the console output and files under `C:\hbase\logs` if startup fails.

## 14.7 Open the HBase shell

In a new Command Prompt, run:

```bat
hbase shell
```

At the HBase prompt, try these basic commands:

```text
status
version
list
```

Create a small table and insert a sample row:

```text
create 'student', 'info'
put 'student', '1', 'info:name', 'Sanjeev'
put 'student', '1', 'info:department', 'CSE'
scan 'student'
get 'student', '1'
```

Delete the example table if you no longer need it:

```text
disable 'student'
drop 'student'
exit
```

## 14.8 Stop HBase

Exit the HBase shell first:

```text
exit
```

Then stop HBase from a Command Prompt:

```bat
cd /d C:\hbase\bin
stop-hbase.cmd
```

If you also started Hadoop and are finished with it, stop YARN and HDFS:

```bat
cd /d C:\hadoop-3.3.6\sbin
stop-yarn.cmd
stop-dfs.cmd
```

## 14.9 HBase quick-revision sequence

For a previously configured installation:

```bat
cd /d C:\hadoop-3.3.6\sbin
start-dfs.cmd
start-yarn.cmd
```

Then:

```bat
cd /d C:\hbase\bin
start-hbase.cmd
hbase shell
```

Inside HBase shell:

```text
status
list
create 'student', 'info'
put 'student', '1', 'info:name', 'Sanjeev'
scan 'student'
exit
```

Stop HBase and Hadoop using the commands in Sections 14.8 and 11.

## 14.10 HBase troubleshooting

- **`hbase` is not recognized:** Check `HBASE_HOME`, `%HBASE_HOME%\bin` in `Path`, and reopen Command Prompt.
- **Java version or `JAVA_HOME` error:** Confirm the actual JDK path in `hbase-env.cmd`. HBase 1.4.9 is old; use the Java version specified by your lab.
- **HBase cannot connect to HDFS:** Check that HDFS is running and the port in `hbase.rootdir` matches `fs.defaultFS` in Hadoop's `core-site.xml`.
- **HBase startup fails with Hadoop class or method errors:** Suspect an HBase/Hadoop version mismatch. HBase 1.4.9 and Hadoop 3.3.6 are from different generations; use a compatible pair recommended by your course.
- **ZooKeeper directory error:** Ensure the configured local directory exists and the Windows user can write to it.
- **Port conflicts or services already running:** Check the console and logs before starting another instance. Stop services cleanly rather than deleting data directories.


---

# 15. Apache Hive 2.1.0 on Windows Setup

This section adds the Hive setup shown in the screenshots. The screenshot output confirms a Hive 2.1.0 shell connected successfully and ran `create database test;`.

> **Compatibility warning:** Hive 2.1.0 is an old release. It may have compatibility issues with newer Java versions and Hadoop 3.3.6. For your setup guide, use the exact versions provided by your lab instructor. The configuration below documents the supplied setup; if it fails, version compatibility is a key thing to check.

## 15.1 Expected installation paths

The screenshots use:

- Hadoop: `C:\hadoop`
- Hive: `C:\hive`
- Hive configuration: `C:\hive\conf`
- Java: `C:\Java\jdk1.8.0_351`

If your existing Hadoop installation is `C:\hadoop-3.3.6`, use that path consistently instead of `C:\hadoop`. Do not set `HADOOP_HOME` to the `bin` folder; it should point to the Hadoop root directory.

## 15.2 Set Hive environment variables

Open **System Properties → Advanced → Environment Variables** and create or update these variables. Use the paths that actually exist on your computer.

| Variable | Example value |
|---|---|
| `HIVE_HOME` | `C:\hive` |
| `HIVE_BIN` | `C:\hive\bin` |
| `HIVE_LIB` | `C:\hive\lib` |
| `HADOOP_HOME` | `C:\hadoop` or `C:\hadoop-3.3.6` |
| `JAVA_HOME` | `C:\Java\jdk1.8.0_351` |

Add these entries to the `Path` variable:

```text
%HIVE_HOME%\bin
%HADOOP_HOME%\bin
%HADOOP_HOME%\sbin
```

The screenshots show `HADOOP_HOME` and `JAVA_HOME` ending in `\bin`. That is not the recommended setting: `HADOOP_HOME` should be the Hadoop installation root, and `JAVA_HOME` should be the JDK root (not its `bin` directory). For example:

```text
HADOOP_HOME=C:\hadoop
JAVA_HOME=C:\Java\jdk1.8.0_351
```

Open a **new** Command Prompt after changing variables, then verify:

```bat
echo %HIVE_HOME%
echo %HADOOP_HOME%
echo %JAVA_HOME%
java -version
hadoop version
hive --version
```

## 15.3 Set `HADOOP_USER_CLASSPATH_FIRST`

The screenshots create this variable with the value `true` in both user and system variables. If your lab specifically requires it, add:

```text
Variable name: HADOOP_USER_CLASSPATH_FIRST
Variable value: true
```

Usually one definition is sufficient. Avoid defining the same variable differently in both user and system scopes.

## 15.4 Configure `hive-site.xml`

Open:

`C:\hive\conf\hive-site.xml`

If the file does not exist, create a new file named exactly `hive-site.xml` in the `conf` folder. In File Explorer, enable **View → Show → File name extensions** so Windows does not accidentally save it as `hive-site.xml.txt`.

Use the following valid XML configuration based on the code supplied:

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<configuration>
 <property>
 <name>javax.jdo.option.ConnectionURL</name>
 <value>jdbc:derby://localhost:1527/metastore_db;create=true</value>
 <description>JDBC connect string for a JDBC metastore</description>
 </property>

 <property>
 <name>javax.jdo.option.ConnectionDriverName</name>
 <value>org.apache.derby.jdbc.ClientDriver</value>
 <description>Driver class name for a JDBC metastore</description>
 </property>

 <property>
 <name>hive.server2.enable.impersonation</name>
 <value>true</value>
 <description>Enable user impersonation for HiveServer2</description>
 </property>

 <property>
 <name>hive.server2.authentication</name>
 <value>NONE</value>
 <description>Client authentication mode</description>
 </property>

 <property>
 <name>datanucleus.autoCreateTables</name>
 <value>true</value>
 </property>
</configuration>
```

**Important:** The supplied JDBC URL uses the Derby **network server** form (`jdbc:derby://localhost:1527/...`) and therefore expects a Derby Network Server to be running on port `1527`. If the network server is not running, Hive may fail to connect to the metastore. A local embedded Derby setup uses a different JDBC URL and driver; use the URL/driver specified by your lab.

The `hive.server2.*` settings apply to HiveServer2. They are not required just to open the legacy Hive CLI, but they are included because they appear in your provided configuration.

## 15.5 Start Hadoop services

If Hadoop is configured for HDFS, start HDFS first:

```bat
cd /d C:\hadoop\sbin
start-dfs.cmd
```

Then start YARN if required by your practical:

```bat
start-yarn.cmd
jps
```

If your Hadoop folder is `C:\hadoop-3.3.6`, replace `C:\hadoop` in the commands with `C:\hadoop-3.3.6`.

## 15.6 Start the Derby metastore database server

Because the configuration above uses `jdbc:derby://localhost:1527/metastore_db;create=true`, it expects a Derby Network Server listening on port `1527`.

The exact command depends on the Derby libraries bundled with your Hive distribution. If your lab's package includes the Derby server classes and dependencies, use the command supplied by your lab instructions. If the server is not running, the Hive client can report connection-refused or metastore connection errors.

Do not assume that `hive` automatically starts a network Derby server merely because the JDBC URL includes `localhost:1527`.

## 15.7 Open the Hive shell

Open a new Command Prompt and run:

```bat
hive
```

A successful legacy Hive CLI session typically displays a `hive>` prompt. The screenshots show Hive 2.1.0 and successfully execute `create database test;`.

Try these practical-exam commands at the `hive>` prompt:

```sql
show databases;
create database test;
show databases;
use test;
```

Create a table and insert sample data:

```sql
create table student (
 id int,
 name string
)
row format delimited
fields terminated by ',';

insert into student values (1, 'Sanjeev'), (2, 'Rahul');
select * from student;
describe student;
```

If the Hive version or execution engine does not support the multi-row `INSERT` syntax, insert rows separately:

```sql
insert into student values (1, 'Sanjeev');
insert into student values (2, 'Rahul');
select * from student;
```

Exit Hive:

```sql
exit;
```

## 15.8 Hive quick-revision sequence

For a previously configured machine:

1. Start HDFS (and YARN if required).
2. Start the Derby Network Server if your configured JDBC URL uses port `1527`.
3. Run `hive`.
4. Execute:

```sql
show databases;
create database test;
show databases;
use test;
create table student (id int, name string);
show tables;
select * from student;
exit;
```

## 15.9 Troubleshooting Hive

- **`hive` is not recognized:** Check `HIVE_HOME`, `%HIVE_HOME%\bin` in `Path`, and reopen Command Prompt.
- **`JAVA_HOME` error:** Set it to the JDK root folder, not `...\bin`.
- **`HADOOP_HOME` error:** Set it to the Hadoop root directory, not `C:\hadoop\bin`.
- **Connection refused on port `1527`:** The configured Derby Network Server may not be running, or the port may differ.
- **Metastore schema/table errors:** Check the Derby metastore connection and whether the metastore schema has been initialized as required by the installed Hive release.
- **SLF4J multiple-binding warning:** The screenshots show multiple SLF4J bindings. This warning alone may not stop Hive, but duplicate logging JARs can cause confusing logs. Do not randomly delete JARs; check the lab's expected library versions.
- **Java/Hadoop class errors:** Hive 2.1.0 is old. Verify compatibility with the Java and Hadoop versions installed in your lab.
- **Hive starts but SQL fails:** Read the full error; verify the metastore is reachable and HDFS is running if Hive's warehouse is configured on HDFS.

## 15.10 Stop services

Exit Hive with `exit;`. Stop any Derby Network Server using the method supplied by your lab. If you started Hadoop and are finished:

```bat
cd /d C:\hadoop\sbin
stop-yarn.cmd
stop-dfs.cmd
```

Replace the Hadoop path as necessary.
