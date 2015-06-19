# 介绍

Hadoop 作为大数据的平台代表，是每一个从事大数据开发者都值得学习的，刚好我入职的后要做的项目是一个
大数据平台，所以要提前学习下 Hadoop 的使用，包括 hive 和 MapReduce 等的使用。

可以在我的博客[Hadoop 入门教程](http://hustlijian.github.io/tutorial/2015/06/19/Hadoop%E5%85%A5%E9%97%A8%E4%BD%BF%E7%94%A8.html)中看到本文。
github 的渲染效果比我的好呀。

## 目标

Hadoop 自己找资料， 搭建环境，用 [streaming](https://hadoop.apache.org/docs/r1.2.1/streaming.html)， python 写一个 wordcount

# Hadoop 介绍

Apache Hadoop is an open-source software framework written in Java for distributed
storage and distributed processing of very large data sets on computer clusters
built from commodity hardware. All the modules in Hadoop are designed with a
fundamental assumption that hardware failures (of individual machines, or racks
of machines) are commonplace and thus should be automatically handled in software
by the framework.

The term "Hadoop" has come to refer not just to the base modules above, but also
to the "ecosystem", or collection of additional software packages that can be
installed on top of or alongside Hadoop, such as Apache Pig, Apache Hive, Apache
HBase, Apache Spark, and others.

## HDFS(Hadoop Distributed File System)

The **Hadoop distributed file system (HDFS)** is a distributed, scalable, and portable
 file-system written in Java for the Hadoop framework. A Hadoop cluster has
 nominally a single namenode plus a cluster of datanodes, although redundancy
 options are available for the namenode due to its criticality. Each datanode
 serves up blocks of data over the network using a block protocol specific to
 HDFS. The file system uses TCP/IP sockets for communication. Clients use remote
procedure call (RPC) to communicate between each other.

## MapReduce

Above the file systems comes the MapReduce Engine, which consists of one JobTracker,
to which client applications submit MapReduce jobs. The JobTracker pushes work out
to available TaskTracker nodes in the cluster, striving to keep the work as close
to the data as possible.

过程如下：

    Map(k1,v1) → list(k2,v2)

    Reduce(k2, list (v2)) → list(v3)

参考论文：[MapReduce: Simplified Data Processing on Large Clusters](http://research.google.com/archive/mapreduce.html)

## Hive

Apache Hive is a data warehouse infrastructure built on top of Hadoop for
providing data summarization, query, and analysis. While initially developed by
Facebook, Apache Hive is now used and developed by other companies such as
Netflix. Amazon maintains a software fork of Apache Hive that is included in
Amazon Elastic MapReduce on Amazon Web Services.

It provides an SQL-like language called HiveQL with schema on read and transparently
converts queries to map/reduce, Apache Tez and Spark jobs.

# Hadoop 安装

使用mac Yosemite(10.10.3)

    brew insall hadoop
    $ hadoop version

    Hadoop 2.7.0
    Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r d4c8d4d4d203c934e8074b31289a28724c0842cf
    Compiled by jenkins on 2015-04-10T18:40Z
    Compiled with protoc 2.5.0
    From source with checksum a9e90912c37a35c3195d23951fd18f
    This command was run using /usr/local/Cellar/hadoop/2.7.0/libexec/share/hadoop/common/hadoop-common-2.7.0.jar


# Hadoop 配置

## 配置 JAVA_HOME

在`.bashrc` 或`.zshrc` 中加入 `JAVA_HOME` 设置：

    # set java home
    [ -f /usr/libexec/java_home ] && export JAVA_HOME=$(/usr/libexec/java_home)

使设置生效：

    source ~/.bashrc  # source ~/.zshrc

## 配置 ssh

1.生成公钥（如果已经生成，就不用了）

    ssh-keygen -t rsa

2.设置 Mac 允许远程登录

“System Preferences” -> “Sharing”. Check “Remote Login”

3.设置免密码登录

    cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys

4.测试本地登录

    $ ssh localhost

      Last login: Fri Jun  19 16:30:53 2015

    $ exit

## Hadoop 配置单节点使用

这里使用单节点做学习使用，配置文件目录  `/usr/local/Cellar/hadoop/2.7.0/libexec/etc/hadoop` 下

### 配置 hdfs-site.xml

设置副本数为 **1**:

    <configuration>
      <property>
        <name>dfs.replication</name>
        <value>1</value>
      </property>
    </configuration>

### 配置 core-site.xml

设置文件系统访问的端口：

    <configuration>
      <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
      </property>
    </configuration>

### 配置 mapred-site.xml

设置 MapReduce 使用的框架：

    <configuration>
      <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
      </property>
    </configuration>

### 配置 yarn-site.xml

    <configuration>
        <property>
            <name>yarn.nodemanager.aux-services</name>
            <value>mapreduce_shuffle</value>
        </property>
    </configuration>

# Hadoop 运行

## 加入启动和停止 Hadoop 的 alias

    alias hstart="start-dfs.sh ; start-yarn.sh"
    alias hstop="stop-yarn.sh ; stop-dfs.sh"

## 格式化文件系统

    $ hdfs namenode -format

## 启动 Hadoop

    hstart

## 建立用户空间

    hdfs dfs -mkdir /user
    hdfs dfs -mkdir /user/$(whoami) # 这里是用户

## 查看 Hadoop 启动的进程情况

    jps

正常情况如下：

    $ jps
    24610 NameNode
    24806 SecondaryNameNode
    24696 DataNode
    25018 NodeManager
    24927 ResourceManager
    25071 Jps

前面是进程号，后面是进程名

## 关闭 Hadoop

    hstop

# Hadoop 实例

运行实例时，当前目录设置为 `/usr/local/Cellar/hadoop/2.7.0/libexec`

1.上传测试文件到 HDFS 中

    hdfs dfs -put etc/hadoop input

把本地 `etc/hadoop` 下的一些文件上传到 HDFS的 `input` 中。

可以在刚才建立的用户下查看上传的文件： [/user/$(whoami)/input](http://localhost:50070/explorer.html#/user/)

2.在上传的数据中运行 Hadoop 提供的例子

    hadoop jar share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.0.jar grep input output 'dfs[a-z.]+'

在上传的数据中使用 MapReduce 运行 `grep`， 计算以`dfs`开头的单词出现的次数，结果保存到 `output` 中。

3.查看运行结果

    hdfs dfs -cat output/part-r-00000   # 文件名可以从[Browse Directory](http://localhost:50070/explorer.html#/)中看到:

    4	dfs.class
    4	dfs.audit.logger
    3	dfs.server.namenode.
    2	dfs.period
    2	dfs.audit.log.maxfilesize
    2	dfs.audit.log.maxbackupindex
    1	dfsmetrics.log
    1	dfsadmin
    1	dfs.servers
    1	dfs.replication
    1	dfs.file

4.删除刚才生成的文件

    hdfs dfs -rm -r /user/$(whoami)/input
    hdfs dfs -rm -r /user/$(whoami)/output

# 使用 python 通过 streaming 完成 wordcount

虽然 Hadoop 是使用 Java 开发的，不过支持其它语言开发 MapReduce 程序：

* [Hadoop Streaming](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/streaming/package-summary.html) is a utility which allows users to create and run jobs with any executables (e.g. shell utilities) as the mapper and/or the reducer.
* [Hadoop Pipes](http://hadoop.apache.org/docs/current/api/org/apache/hadoop/mapred/pipes/package-summary.html) is a SWIG-compatible C++ API to implement MapReduce applications (non JNI™ based).

## 设置 Streaming 变量(方便后面使用)

streaming 在 brew 中的目录是：`/usr/local/Cellar/hadoop/2.7.0/libexec/share/hadoop/tools/lib/hadoop-streaming-2.7.0.jar`,
通过命令查找：

    find ./ -type f -name "*streaming*"

设置为一个变量，方便后面使用:

    export STREAM="/usr/local/Cellar/hadoop/2.7.0/libexec/share/hadoop/tools/lib/hadoop-streaming-2.7.0.jar"

## 编写 Map 和 Reduce 程序

默认是从标准输入中读取数据，输出到标准输出中，调用时可以使用输入输出重定向就可以实现和 Hadoop 交互了，
这应该也就是 Streaming 的含义了，自己写的程序也可以通过管道自己调试。

### mapper.py

    #!/usr/bin/env python
    # filename: mapper.py

    import sys

    for line in sys.stdin:
        line = line.strip()
        words = line.split()
        for word in words:
            print '%s\t%s' % (word, 1)

给程序加可执行权限：

    chmod +x mapper.py

测试下：

    $ echo "this is a  test " | ./mapper.py
    this	1
    is	1
    a	1
    test	1

### reducer.py

    #!/usr/bin/env python
    # filename:reducer.py

    import sys

    current_word = None
    current_count = 0
    word = None

    for line in sys.stdin:
        line = line.strip()
        word, count = line.split('\t', 1)

        try:
            count = int(count)
        except ValueError:
            continue

        if current_word == word:
            current_count += count
        else:
            if current_word:
                print '%s\t%s' % (current_word, current_count)
            current_count = count
            current_word = word

    if current_word == word:
        print '%s\t%s' % (current_word, current_count)

给程序加可执行权限：

      chmod +x reducer.py

测试下：

    $ echo "this is a a a  test test " | ./mapper.py | sort -k1,1 | ./reducer.py
    a	3
    is	1
    test	2
    this	1  

### 使用 Hadoop 调用

1.准备数据

* [The Outline of Science, Vol. 1 (of 4) by J. Arthur Thomson](http://www.gutenberg.org/ebooks/20417)
* [The Notebooks of Leonardo Da Vinci](http://www.gutenberg.org/etext/5000)
* [Ulysses by James Joyce](http://www.gutenberg.org/etext/4300)

        $ ls -l
        total 7200
        -rwxr-xr-x  1 user  staff      165 Jun 19 20:43 mapper.py
        -rw-r-----@ 1 user  staff   674570 Jun 19 21:14 pg20417.txt
        -rw-r-----@ 1 user  staff  1573151 Jun 19 21:14 pg4300.txt
        -rw-r-----@ 1 user  staff  1423803 Jun 19 21:16 pg5000.txt
        -rwxr-xr-x  1 user  staff      539 Jun 19 20:51 reducer.py

2.上传文件到 HDFS

文件要上传到 HDFS 中才能使用 Hadoop 的 MapReduce:

    $ hdfs dfs -mkdir /user/$(whoami)/input
    $ hdfs dfs -put ./*.txt /user/$(whoami)/input #*

3.运行 MapReduce

    $ hadoop jar $STREAM  \
    -files ./mapper.py,./reducer.py \
    -mapper ./mapper.py \
    -reducer ./reducer.py \
    -input /user/$(whoami)/input/pg5000.txt,/user/$(whoami)/input/pg4300.txt,/user/$(whoami)/input/pg20417.txt\
     -output /user/$(whoami)/output

4.查看结果

    $ hdfs dfs -cat /user/$(whoami)/output/part-00000 | sort -nk 2 | tail
    with	4686
    it	4981
    that	6109
    is	7401
    in	11867
    to	12017
    a	12064
    and	16904
    of	23935
    the	42074

说明在正常的书中，介词用得真是相当多的，这些词在很多时候就要去除。

5.改进（使用迭代器和生成器）

使用 `yield` 可以在需要时再提供数据，在占用内存的工作时很有效。

改进的 mapper.py

    #!/usr/bin/env python
    """A more advanced Mapper, using Python iterators and generators."""

    import sys

    def read_input(file):
        for line in file:
            # split the line into words
            yield line.split()

    def main(separator='\t'):
        # input comes from STDIN (standard input)
        data = read_input(sys.stdin)
        for words in data:
            # write the results to STDOUT (standard output);
            # what we output here will be the input for the
            # Reduce step, i.e. the input for reducer.py
            #
            # tab-delimited; the trivial word count is 1
            for word in words:
                print '%s%s%d' % (word, separator, 1)

    if __name__ == "__main__":
        main()

改进的 reducer.py

    #!/usr/bin/env python
    """A more advanced Reducer, using Python iterators and generators."""

    from itertools import groupby
    from operator import itemgetter
    import sys

    def read_mapper_output(file, separator='\t'):
        for line in file:
            yield line.rstrip().split(separator, 1)

    def main(separator='\t'):
        # input comes from STDIN (standard input)
        data = read_mapper_output(sys.stdin, separator=separator)
        # groupby groups multiple word-count pairs by word,
        # and creates an iterator that returns consecutive keys and their group:
        #   current_word - string containing a word (the key)
        #   group - iterator yielding all ["&lt;current_word&gt;", "&lt;count&gt;"] items
        for current_word, group in groupby(data, itemgetter(0)):
            try:
                total_count = sum(int(count) for current_word, count in group)
                print "%s%s%d" % (current_word, separator, total_count)
            except ValueError:
                # count was not a number, so silently discard this item
                pass

    if __name__ == "__main__":
        main()
        
# 查看系统状态的 UI

* Resource Manager: [http://localhost:50070](http://localhost:50070)
* JobTracker: [http://localhost:8088](http://localhost:8088)
* Specific Node Information: [http://localhost:8042](http://localhost:8042)

# 参考

1. [Apache Hadoop](https://en.wikipedia.org/wiki/Apache_Hadoop)
1. [Welcome to Apache™ Hadoop®!](https://hadoop.apache.org/)
2. [Apache Hive](https://en.wikipedia.org/wiki/Apache_Hive)
3. [APACHE HIVE TM](http://hive.apache.org/)
4. [Setting up Hadoop 2.4 and Pig 0.12 on OSX locally](http://www.getblueshift.com/setting-up-hadoop-2-4-and-pig-0-12-on-osx-locally/)
5. [INSTALLING HADOOP ON MAC OSX YOSEMITE TUTORIAL PART 1.](http://amodernstory.com/2014/09/23/installing-hadoop-on-mac-osx-yosemite/)
6. [MapReduce Tutorial](http://hadoop.apache.org/docs/current/hadoop-mapreduce-client/hadoop-mapreduce-client-core/MapReduceTutorial.html)
7. [Hadoop Streaming](https://hadoop.apache.org/docs/r1.2.1/streaming.html)
8. [Writing an Hadoop MapReduce Program in Python](http://www.michael-noll.com/tutorials/writing-an-hadoop-mapreduce-program-in-python/)
