
今天我们来聊一聊大数据，作为一个Hadoop的新手，我也并不敢深入探讨复杂的底层原理。因此，这篇文章的重点更多是从实际操作和入门实践的角度出发，带领大家一起了解大数据应用的基本过程。我们将通过一个经典的案例——WordCounter，来帮助大家入门。简单来说，这个案例的目标是从一个文本文件中读取每一行，统计其中单词出现的频率，最后生成一个统计结果。表面上看，这个任务似乎不难，毕竟我们在本地用Java程序就可以很轻松地实现。


然而，实际情况并非如此简单。虽然我们能够在一台计算机上通过简单的Java程序完成类似的任务，但在大数据的场景下，数据量远远超过一台机器能够处理的能力。此时，单纯依赖一台机器的计算资源就无法应对庞大的数据量，这正是分布式计算和存储技术的重要性所在。分布式计算将任务拆分为多个子任务，并利用多台机器协同工作，从而实现高效处理海量数据，而分布式存储则可以将数据切分并存储在多个节点上，解决数据存储和访问的瓶颈。


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241115092036818-1291708807.png)


因此，通过今天的介绍，我希望能够带大家从一个简单的例子出发，逐步理解大数据处理中如何借助Hadoop这样的分布式框架，来高效地进行数据计算和存储。


# 环境准备


## Hadoop安装


这里我不太喜欢在本地 Windows 系统上进行安装，因为本地环境中通常会积累很多不必要的文件和配置，可能会影响系统的干净与流畅度。因此，演示的重点将放在以 Linux 服务器为主的环境上，通过 Docker 实现快速部署。


我们将利用宝塔面板进行一键式安装，只需通过简单的操作即可完成整个部署过程，免去手动敲命令的麻烦，让安装变得更加便捷和快速。


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241115092042218-1428286348.png)


### 开放端口


这里，系统本身已经对外开放了部分端口，例如 9870 用于访问 Web UI 界面，但有一个重要的端口 8020 并没有开放。这个端口是我们需要通过本地的 IntelliJ IDEA 进行连接和使用的，因此必须手动进行额外的配置，确保该端口能够正常访问。具体操作可以参考以下示意图进行设置，以便顺利完成连接。


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241115092047379-1410843469.png)


如果你已经成功启动并完成配置，那么此时你应该能够顺利访问并查看 Web 页面。如图所示：


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241115092051987-663398474.png)


# 项目开发


## 创建项目


我们可以直接创建一个新的项目，并根据项目需求手动配置相关的项目信息，例如 `groupId`、`artifactId`、`version` 等基本配置。为了确保兼容性和稳定性，我们选择使用 JDK 8 作为开发环境版本。


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241115092057004-748416758.png)


首先，让我们先来查看一下项目的文件目录结构，以便对整个项目的组织形式和文件分布有一个清晰的了解。



> tree /f 可以直接生成



```
├─input
│      test.txt
├─output
├─src
│  ├─main
│  │  ├─java
│  │  │  └─org
│  │  │      └─xiaoyu
│  │  │              InputCountMapper.java
│  │  │              Main.java
│  │  │              WordsCounterReducer.java
│  │  │
│  │  └─resources
│  │          core-site.xml
│  │          log4j.xml

```

接下来，我们将实现大数据中的经典示例——"Hello, World!" 程序，也就是我们通常所说的 WordCounter。为了实现这个功能，首先，我们需要编写 MapReduce 程序。在 Map 阶段，主要的任务是将输入的文件进行解析，将数据分解并转化成有规律的格式（例如，单词和其出现次数的键值对）。接着，在 Reduce 阶段，我们会对 Map 阶段输出的数据进行汇总和统计，最终得到我们想要的统计结果，比如每个单词的出现次数。


此外，我们还需要编写一个启动类——Job 类，用来配置和启动 MapReduce 任务，确保 Map 和 Reduce 阶段的流程能够顺利进行。通过这整套流程的实现，我们就完成了一个基本的 WordCounter 程序，从而理解了 MapReduce 的核心思想与应用。


### pom依赖


这里没有什么好说的，直接添加相关依赖即可：



```
<dependency>
    <groupId>org.apache.hadoopgroupId>
    <artifactId>hadoop-commonartifactId>
    <version>3.2.0version>
dependency>
<dependency>
    <groupId>org.apache.hadoopgroupId>
    <artifactId>hadoop-hdfsartifactId>
    <version>3.2.0version>
dependency>
<dependency>
    <groupId>log4jgroupId>
    <artifactId>log4jartifactId>
    <version>1.2.17version>
dependency>
<dependency>
    <groupId>org.apache.hadoopgroupId>
    <artifactId>hadoop-clientartifactId>
    <version>3.2.0version>
dependency>


<dependency>
    <groupId>org.apache.hadoopgroupId>
    <artifactId>hadoop-mapreduce-client-coreartifactId>
    <version>3.2.0version>
dependency>
<dependency>
    <groupId>org.apache.hadoopgroupId>
    <artifactId>hadoop-mapreduce-client-commonartifactId>
    <version>3.2.0version>
dependency>

```

### core\-site.xml


这里配置的我们远程Hadoop连接配置信息：



```
xml version="1.0" encoding="UTF-8"?
xml-stylesheet type="text/xsl" href="configuration.xsl"?
<configuration>
    <property>
        <name>fs.defaultFSname>
        <value>hdfs://你自己的ip:8020value>
    property> 
configuration>

```

### test.txt


我们此次主要以演示为主，因此并不需要处理非常大的文件。为了简化演示过程，我在此仅提供了一部分数据。



```
xiaoyu xiaoyu
cuicui ntfgh
hanhan dfb
yy yy
asd dfg
123 43g
nmao awriojd

```

### InputCountMapper


先来构建一下InputCountMapper类。代码如下:



```
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Mapper;

import java.io.IOException;

public class InputCountMapper extends Mapper {
    private final static IntWritable one = new IntWritable(1);
    private Text word = new Text();

    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString().trim();
        for (int i = 0; i < line.split(" ").length; i++) {
            word.set(line.split(" ")[i]);
            context.write(word, one);
        }
    }
}

```

在Hadoop的MapReduce编程中，写法其实是相对简单的，关键在于正确理解和定义泛型。你需要集成一个`Mapper`类，并根据任务的需求为其定义四个泛型类型。在这个过程中，每两个泛型组成一对，形成K\-V（键值对）结构。以上面的例子来说，输入数据的K\-V类型是`LongWritable-Text`，输出数据的K\-V类型定义为`Text-IntWritable`。这里的`LongWritable`、`Text`、`IntWritable`等都是Hadoop自定义的数据类型，它们代表了不同的数据格式和类型。除了`String`在Hadoop中被替换成`Text`，其他的数据类型通常是在后面加上`Writable`后缀。


接下来，对于`Mapper`类的输出格式，我们已经在代码中定义了格式类型。然而，需要注意的是，我们重写的`map`方法并没有直接返回值。相反，`Mapper`类会通过`Context`上下文对象来传递最终结果。


因此，我们只需要确保在`map`方法中将格式化后的数据存入`Context`，然后交给`Reducer`处理即可。


### WordsCounterReducer


这一步的代码如下：



```
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Reducer;

import java.io.IOException;

public class WordsCounterReducer extends Reducer {
    @Override
    protected void reduce(Text key, Iterable values, Context context) throws IOException, InterruptedException {
        int sum = 0;
        for (IntWritable val : values) {
            sum += val.get();
        }
        context.write(key, new IntWritable(sum));
    }
}

```

在Hadoop的MapReduce编程中，`Reduce`阶段的写法也遵循固定模式。首先，我们需要集成`Reducer`类，并定义好四个泛型参数，类似于`Mapper`阶段。这四个泛型包括输入键值对类型、输入值类型、输出键值对类型、以及输出值类型。


在`Reduce`阶段，输入数据的格式会有所变化，尤其是在值的部分，通常会变成`Iterable`类型的集合。这个变化的原因是，`Mapper`阶段处理时，我们通常将每个单词的出现次数（或其他统计信息）作为1存入`Context`。比如，假设在`Mapper`阶段遇到单词“xiaoyu”时，我们每次都会输出一个`(xiaoyu, 1)`的键值对。结果，如果单词“xiaoyu”在输入数据中出现多次，`Context`会把这些键值对合并成一个`Iterable`集合，像是`(xiaoyu, [1, 1])`，表示该单词出现了两次。


在这个例子中，`Reduce`阶段的操作非常简单，只需要对每个`Iterable`集合中的值进行累加即可。比如，对于`xiaoyu`的输入集合`(xiaoyu, [1, 1])`，我们只需要将其所有的`1`值累加起来，得出最终的结果2。


### Main


最后我们需要生成一个Job，代码如下：



```
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class Main {
    static {
        try {
            System.load("E:\\hadoop.dll");//建议采用绝对地址，bin目录下的hadoop.dll文件路径
        } catch (UnsatisfiedLinkError e) {
            System.err.println("Native code library failed to load.\n" + e);
            System.exit(1);
        }
    }


    public static void main(String[] args) throws Exception{
        Configuration conf = new Configuration(); 
        Job job = Job.getInstance(conf, "wordCounter"); 
        job.setJarByClass(Main.class);
        job.setMapperClass(InputCountMapper.class);
        job.setReducerClass(WordsCounterReducer.class);

        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);

        FileInputFormat.addInputPath(job, new Path("file:///E:/hadoop/test/input"));
        FileOutputFormat.setOutputPath(job, new Path("file:///E:/hadoop/test/output"));

        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}

```

好的，这里所展示的是一种完全固定的写法，但在实际操作过程中，需要特别注意的是，我们必须通过 Windows 环境来连接远程的 Hadoop 集群进行相关操作。这个过程中会遇到很多潜在的问题和坑，尤其是在配置、连接、权限等方面。


接下来，我将逐一解析并解决这些常见的难题，希望能为大家提供一些实际的参考和指导，帮助大家更顺利地完成操作。


# 疑难解答


## 目录不存在


如果你并不是以本地 Windows 目录为主，而是以远程服务器上的目录为主进行操作，那么你可能会采用类似以下的写法：



```
FileInputFormat.addInputPath(job, new Path("/input"));
FileOutputFormat.setOutputPath(job, new Path("/output"));

```

那么，在这种情况下，我们必须先创建与操作相关的输入目录（input），但需要特别注意的是，切勿提前创建输出目录（output），因为 Hadoop 在运行作业时会自动创建该目录，如果该目录已存在，会导致作业执行失败。因此，只需要进入 Docker 环境并直接执行以下命令即可顺利开始操作。



> hdfs dfs \-mkdir /input


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241115092112915-1204763710.png)


当然，还有一种更简单的方式，就是直接通过图形界面在页面上创建相关目录或资源。具体操作可以参考以下步骤，如图所示：


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241115092117633-1297167029.png)


## Permission denied


接下来，当你在运行 Job 任务时，系统会在最后一步尝试创建输出目录（output）。然而，由于当前用户并没有足够的权限来进行此操作，因此会出现类似于以下的权限错误提示：`Permission denied: user=yu, access=WRITE, inode="/":root:supergroup:drwxr-xr-x`。该错误意味着当前用户（yu）试图在根目录下创建目录或文件，但由于该目录的权限设置为只有管理员（root）才能写入，普通用户无法进行写操作，从而导致作业执行失败。


所以你仍需要进入docker容器，执行以下命令：



> hadoop fs \-chmod 777 /


这样基本上就可以顺利完成任务了。接下来，你可以直接点击进入查看 output 目录下的文件内容。不过需要注意的是，由于我们没有配置具体的 IP 地址，因此在进行文件下载时，你需要手动将文件中的 IP 地址替换为你自己真实的 IP 地址，才能确保下载过程能够顺利进行并成功获取所需的文件。


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241115092122965-825725416.png)


## 报错：org.apache.hadoop.io.nativeio.NativeIO$Windows


这种问题通常是由于缺少 `hadoop.dll` 文件导致的。在 Windows 系统上运行 Hadoop 时，`hadoop.dll` 或者 `winutils.exe` 是必需的依赖文件，因为它们提供了 Hadoop 在 Windows 上所需的本地代码支持和执行环境。


为了确保顺利运行，你需要下载对应版本的 `hadoop.dll` 或者 `winutils.exe` 文件。已经为你准备好了多个 Hadoop 版本对应的这些文件，所有的文件都可以从以下链接下载：[https://github.com/cdarlint/winutils](https://github.com)


我们这里只下载一个hadoop.dll，为了不重启电脑，直接在代码里面写死：



```
static {
  try {
      System.load("E:\\hadoop.dll");//建议采用绝对地址，bin目录下的hadoop.dll文件路径
  } catch (UnsatisfiedLinkError e) {
      System.err.println("Native code library failed to load.\n" + e);
      System.exit(1);
  }
}

```

如果仍然有问题，那就配置下windows下的wsl子系统：


使用Windows \+ R快捷键打开「运行」对话框，执行OptionalFeatures打开「Windows 功能」。


勾选「适用于 Linux 的 Windows 子系统」和「虚拟机平台」，然后点击「确定」。


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241115092130362-1322553952.png)


# 最终效果


终于成功跑出结果了！在这个过程中，输出的结果是按照默认的顺序进行排序的，当然这个排序方式是可以根据需要进行自定义的。如果你对如何控制排序有兴趣，实际上可以深入了解并调整排序机制。


![image](https://img2024.cnblogs.com/blog/1423484/202411/1423484-20241115092135759-1897597497.png)


# 总结


通过今天的分享，我们简单地了解了大数据处理中一个经典的应用——WordCounter，并通过Hadoop框架的实践，展示了如何使用MapReduce进行分布式计算。虽然表面上看，WordCounter是一个相对简单的程序，但它却揭示了大数据处理中的核心思想。


从安装配置到编写代码，我们一步步走过了Hadoop集群的搭建过程，希望通过这篇文章，你能对大数据应用开发，特别是Hadoop框架下的MapReduce编程，获得一些启发和帮助。大数据的世界庞大而复杂，但每一次小小的实践，都会带你离真正掌握这门技术更近一步。




---


我是努力的小雨，一名 Java 服务端码农，潜心研究着 AI 技术的奥秘。我热爱技术交流与分享，对开源社区充满热情。同时也是一位腾讯云创作之星、阿里云专家博主、华为云云享专家、掘金优秀作者。


💡 我将不吝分享我在技术道路上的个人探索与经验，希望能为你的学习与成长带来一些启发与帮助。


🌟 欢迎关注努力的小雨！🌟


 本博客参考[豆荚加速器](https://yirou.org)。转载请注明出处！
