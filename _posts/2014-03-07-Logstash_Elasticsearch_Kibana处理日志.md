---
layout: page
title: Logstash, Elasticsearch, Kibana处理日志
tag: [ Logstash, Elasticsearch, Kibana, 日志, 数据, 可视化, 安装, 配置, 坑 ]
---
{% include JB/setup %}

**使用环境:**

- **Mac OS X(10.9.2)**
- **Ruby 2.1.0**
- **jdk 1.7.0_51**


>具体的安装和配置过程，已经有很多文章介绍，可以参考结尾的链接，这里不再赘述。主要分享一下，我个人的总结和经历，希望可以给你带来一些帮助，当然如果你没有遇到像我这样的蠢问题那是最好不过。

## 两个主要的使用场景:
---
#### 1. Standalone server:
使用**Logstash**和内部集成的**Elasticsearch**和**Kibana**，足以胜任在一台单独的Server上分析和实时统计各种格式的日志文件。

#### 2. Centralized server: 
![image](http://michael.bouvy.net/blog/wp-content/uploads/2013/11/logstach-archi1.png)  
  

如图示，需要在分布式的Servers上通过**Logstash**的agent进行收集数据（使用**Redis**进行存储），然后由一台Server集中地从**Redis**中pull下来，建立索引，并结合**Elasticsearch**进行查询和统计。

#### Logstash内部已经集成了Elasticsearch和Kibana

**Elasticsearch**和**Kibana**可以选择使用已经集成在**logstash**中的，也可以根据自己的实际情况更加灵活的配置。  
对于使用集成的，只需要在**logstash**的配置文件中加入  
{% highlight text %}
output {
	elasticsearch {
		embedded => true
	}
}
{% endhighlight %}  
这样每次在运行**logstash**的时候，就会自动在localhost:9200端口启动**Elasticsearch**服务，在localhost:9292端口启动**Kibana**服务。  


## 我的需求
---
在我本地处理一堆日志文件：应用日志，Web前端的请求日志，代理服务器的日志，系统捕获的错误日志，blablabla...(部分日志文件格式是csv)  
所以我只要选择**Standalone**的方式，就可以满足我的需求，还要借助filter中的csv插件来解析csv日志。  
我的需求简单明了，似乎官方最简单的教程都可以满足我的需求。。  
But, 我还是遇到坑了，peut-être是我的智商问题＝ ＝。


### 出门就遇坑
安装的过程都很简单，实际上几乎不需要什么安装，都是下载完，解压，添加或修改配置文件，然后run。  
But，如果一切都太简单的时候，最容易让人感到不安。Ruby的魔法虽然神奇，但是总会让人没有安全感。  
于是乎，坑来了。  

#### 坑1: *为何刚配置好，就报错！！(没有认真读文档，不知道Logstash已经内置了Elasticsearch)*
启动了**Logstash**以后(**.conf中已经添加elasticsearch { embedded => true }**)，再尝试从外部单独启动**Elasticsearch**，结果shell一直跑警告刷屏，大概错误信息如下：
```
org.elasticsearch.transport.RemoteTransportException: Failed to deserialize exception response from stream
```
到处Google，无解，别人提到的所有解决方法几乎都有尝试。  
还以为是版本不兼容，又换了不同版本。还是不行。
后来又重新找了几篇比较详细，结合示例的文章。
最后终于发现了原因，大概是因为内置的**Elasticsearch**启动后和外部的占用了同一个端口，导致了一些异常(我不能准确地定位，但是问题大概是出现在这个方向上)，我改成只使用内置的**Elasticsearch**，或者：
{% highlight text %}
	elasticsearch {
		embedded => false
	}
{% endhighlight %} 
再自己指定外部的**Elasticsearch**的host和port。


#### 坑2:  *为何不读取我的文件！！*
我的.conf文件如下：
{% highlight text %}
input {
  stdin {
    type => "stdin-type"
  }

  file {
    type => "syslog"

    # Wildcards work, here :)
    path => [ "mypath/LOGS/*.log" ]
  }
}

output {
  stdout { }
  elasticsearch { embedded => true }
}
{% endhighlight %}  

我获得的日志文件是静态的，我提前把它们文件到某个目录下。  
我准备好.conf文件以后，激动地执行了：
```bash
$ java -jar logstash-1.3.3-flatjar.jar agent -f test.conf -- web
```
然后检查这些文件的信息有没有输出到Elasticsearch中，  
```bash
$ curl -s http://127.0.0.1:9200/_status\?pretty\=true
```
什么都没有！？
我试了无数次，重启了**Elasticsearch**，重启了iTerm，甚至重启了电脑，ça change rien...

后来我试了官方教程中，给的示例配置，把文件路径修改成
```text
path => [ "/var/log/*.log", "/var/log/messages", "/var/log/syslog" ]
```
我观察输出的情况，发现只有第一次启动时会读取整个文件，然后都是增量式的，当文件有变动时，**Logstash**才会实时的读取，并输出到**Elasticsearch**。

然后针对我的需求，我就将所有文件暂时从目录下移除，开启了Logstash以后，再将它们移会，果然，可以正常的输出到**Elasticsearch**了。风扇开始嗡嗡嗡得转。



>参考文章:

- **(Standalone server)**这里有两片文章，结合实例来说的，但是是法语的，结合代码和作者的图示，即使不懂法语还是勉强可以看明白的
[Logstash, ElasticSearch, Kibana – S01E01 – Analyse d’e-réputation sur Twitter](http://blog.xebia.fr/2013/12/06/logstash-elasticsearch-kibana-s01e01-analyse-de-reputation-sur-twitter/)  
[Logstash, ElasticSearch, Kibana – S01E02 – Analyse orientée business de vos logs applicatifs](http://blog.xebia.fr/2013/12/12/logstash-elasticsearch-kibana-s01e02-analyse-orientee-business-de-vos-logs-applicatifs/)  
- **(Centralized server)**这篇文章说的也很详细，作者在需要收集日志的**servers**(**shippers**)上使用了**Redis**和**Logstash**来存储日志数据；然后使用一台专门的**server**(**indexer**)上使用**Logstash**从**shippers**的**Redis**中读取数据，然后用**elasticsearch**进行查询和搜索，最后用**Kibana**进行可视化。  
[Collect & visualize your logs with Logstash, Elasticsearch & Redis](http://michael.bouvy.net/blog/en/2013/11/19/collect-visualize-your-logs-logstash-elasticsearch-redis-kibana/)  
- [Shipping nginx access logs to LogStash](https://medium.com/devops-programming/b01bd0876e82)
- 使用**Logstash**处理csv文件  
[Using Logstash, Elasticsearch and Kibana to get insight in my spending](http://www.trafex.nl/2014/03/02/using-logstash-elasticsearch-and-kibana-to-get-insight-in-my-spending/)
