# MapReduce课前预习
## MapReduce的定义    
 MapReduce 是一种用于在大型商用硬件集群中（成千上万的结点）对海量数据（多个兆字节数据集）实施可靠的、高容错的分布式计算的框架，也是一种经典的并行计算模型。    
 ### MapReduce核心特性    
 主要用于大数据计算领域，解决海量数据的计算问题。
MR 本身只是一个编程和计算框架，或者干脆一点就是一堆可调用的 jar 包，和 mysql、hdfs、impala等有运行实例的服务不一样， MR 本身没有运行实例。
MR 有两个阶段组成：Map 和 Reduce，用户只需实现 map() 和 reduce() 两个函数，即可实现分布式计算。
MapReduce 编程模型只包含 Map 和 Reduce 两个过程，map 的主要输入是一对 <key,value> 值，经过 map 计算后输出一对 <key,value> 值；然后将相同 Key 合并，形成 <key,value> 集合;再将这个<key,value 集合>输入 reduce，经过计算输出零个或多个 <key,value> 对。     
## MapReduce的基本原理   
 将一个复杂的问题（数据集）分成若干个简单的子问题（数据块）进行解决（Map函数）；然后对子问题的结果进行合并（Reduce函数）。得到原有问题的解（结果）。MapReduce模型适合于大文件的处理，对很多小文件的处理效率不是很高。   
 ![](https://img-blog.csdnimg.cn/6270a57c22084904ad8912dabf98f5f9.png)    
 ## MapReduce优缺点   
 ### 优点   
 1 ）MapReduce 易于编程     
它简单的实现一些接口， 就可以完成一个分布式程序， 这个分布式程序可以分布到大量廉价的 PC 机器上运行。也就是说你写一个分布式程序，跟写一个简单的串行程序是一模一样的。就是因为这个特点使得 MapReduce 编程变得非常流行。          
2 ） 良好的扩展性     
当你的计算资源不能得到满足的时候， 你可以通过简单的增加机器来扩展它的计算能力。     
3 ） 高容错性     
MapReduce 设计的初衷就是使程序能够部署在廉价的 PC 机器上， 这就要求它具有很高的容错性。比如其中一台机器挂了，它可以把上面的计算任务转移到另外一个节点上运行，不至于这个任务运行失败， 而且这个过程不需要人工参与， 而完全是由Hadoop内部完成的。    
4 ） 适合 PB 级以上海量数据的离线处理     
可以实现上千台服务器集群并发工作，提供数据处理能力。     
### 缺点       
1 ） 不擅长实时计算     
MapReduce 无法像 MySQL 一样，在毫秒或者秒级内返回结果。     
2 ） 不擅长流式计算     
流式计算的输入数据是动态的， 而 MapReduce 的输入数据集是静态的， 不能动态变化。
这是因为 MapReduce 自身的设计特点决定了数据源必须是静态的。    
3 ） 不擅长 DAG （有向无环图）计算     
多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出。在这种情况下，MapReduce 并不是不能做， 而是使用后， 每个 MapReduce 作业的输出结果都会写入到磁盘，会造成大量的磁盘 IO，导致性能非常的低下。   
## MapReduce编程模型  
### MapReduce编程模型简介    
#### MapReduce简单模型   
对于某些任务来说，可能并不一定需要Reducd过程，如只需要对文本的每一行数据作简单的格式转换即可，那么需要由Mapper处理后就可以了。所以MapReduce也有简单的编程模型，该模型只有Mapper过程，由Mapper产生的数据直接写入HDFS。    
#### MapReduce复杂模型    
对于大部分的任务来说，都是需要Reduce过程，并且由于任务繁重，会启动多个Reduce（默认为1，根据任务量可由用户自己设定合适的Reduce数量）来进行汇总。如果只用一个Reduce计算所有Mapper的结果，会导致单个Reduce负载过于繁重，成为性能的瓶颈，大大增加任务的运行周期。   
## MapReduce编程实例   
参考：[Wordcount 案例分析](https://blog.csdn.net/qq_48268603/article/details/112817133?spm=1001.2101.3001.6650.2&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-112817133-blog-100631399.pc_relevant_multi_platform_whitelistv4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-2-112817133-blog-100631399.pc_relevant_multi_platform_whitelistv4&utm_relevant_index=5)    
## MapReduce数据流    
### 分片、格式化数据源(InputFormat)    
InputFormat主要有两个任务：    
1、对源文件进行分片，并确定Mapper的数量；    
2、对各分片进行格式化，处理成<key,value>形式的数据流并传给Mapper。    
### Map过程   
Mapper接受<key,value>形式的数据，并处理成<key,value>形式的数据，具体的处理过程可由用户定义。   
### Combiner过程    
每一个map()都可能会产生大量的本地输出，Combiner()的作用就是对map()端的输出先做一次合并，以减少在Map和Reduce结点之间的数据传输量，提高网络I/O性能，是MapReduce的一种优化手段之一。    
### Shuffle过程   
Shufe过程是指从Mapper产生的直接输出结果，经过一系列的处理，成为最终的Reducer直接输入数据为止的整个过程，这一过程也是 MapReduce的核心过程。     
整个Shufle过程可以分为两个阶段，Mapper 端的Shufle和Reducer端的Shufile。由Mapper产生的数据并不会直接写入磁盘，而是先存储在内存中，当内存中的数据达到设定阈值时，再把数据写到本地磁盘，并同时进行sort (排序)、combine ( 合并)、partition (分片)等操作。sort 操作是把Mapper产生的结果按key值进行排序; combine 操作是把key值相同的相邻记录进行合并; partition 操作涉及如何把数据均衡地分配给多个Reducer,它直接关系到Reducer的负载均衡。其中combine操作不一-定会有，因为在某些场景不适用，但为了使Mapper的输出结果更加紧凑，大部分情况下都会使用。      
Maper和Reducer是运行在不同的结点上的，或者说，Mapper 和Reducer运行在同一个第点上的情况很少，并且，Reducer 数量总是比Mapper数量少，所以Reducer端总是要从其他街↑结点上下载Mapper的结果数据，这些数据也要进行相应的处理才能更好地被Rchuce理这些处理过程就是Reduer端的Sule过程。      
### Reduce过程   
Reducer接收<key, {value list}>形式的数据流，形成<key, value>形式的数据输出，输出数据直接写入HDFS,具体的处理过程可由用户定义。在WordCount 中，Reducer 会将相同key的value list进行累加，得到这个单词出现的总次数，然后输出。     
## MapReduce任务运行流程   
![](https://img-blog.csdnimg.cn/c322ff0b54fa434b93625d8f6fa2eb2e.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_Q1NETiBAU2hvY2thbmc=,size_103,color_FFFFFF,t_70,g_se,x_16)
