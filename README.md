## Notice  
This repesition has stoppted updating.

## Overview
---
 time:2016.3
* 检索方式：  
  通过cypher中match的语法，对query（已转化为cypher语句）在neo4j上进行检索，对得到的结果进行处理（排序等），和预置选项进行匹配，得到最终结果。关键：自然语言转为cypher查询语句
* 基础：  
 已存在预期的neu4j数据库
* 具体方式：  

  1. 对query进行分词、语义角色标注(依存分析)（借助niuparser等）
  2. 确定query的类别
	* 关键：把query分为两种形式（需要制定标准）：  
	  1. 实体的属性{attribute:key}  
	  2. 关系实体的元组-[:r{attribute:key}]->(e)  
  3. 使用match进行查询，得到结果（集合）
    * 关键：需要能进行同义词的匹配  

## Ver  0.1  
---
time:2016.4  
* change:  
  经过讨论，目前的数据库建库方式采用<e,r,e>,<r,“name”,value>的形式，即目前的数据库中node是不存在attribute的。
* 基础：  
	neo4j Document: http://neo4j.com/docs/2.3.3/
* 问句的分析：  
  1. 分词及词性分析
  2. 命名实体识别（地点，时间，人名等）
  3. 依存关系分析（主谓，动宾，修饰等）
  4. 语句分类（疑问词，限定域，开放域）
  5. 去冗余提取焦点词[focus word]
  6. 最终句子的组织方式
* 查询和相关API:  
  1. 问句表达式到cypher的转化
  2. 查询及返回
  3. 为后续分析编写设置API
	 
#### 前期尝试:  
* 语句处理阶段  
  1. 最初，我认为如果采用文法规则的方法去分析问句的话，需要使用一个文法分析效果比较好的工具。于是，我尝试了去使用nltk的一些库，但是在nltk中并没有找到关于已存分析的工具。
  2. 然后，我放弃了nltk，发现了工具HanLP,项目托管地址：https://github.com/hankcs/HanLP ，hanlp是主要针对java写的，所有需要导入jpype这个包，去实现在python中嵌入java。尝试后发现，hanlp在py下的工作状	态不如java，它的dp无法实现，但是其进行“摘要”的功能是不错的，可以留待后用。  
  3. 最终，我暂时使用了niuparse的相关功能。
* neo4j数据库链接阶段  
  1. 最初，进行了java的嵌入式尝试，效果还是不错的。具体实现可参见我的blog:http://blog.csdn.net/u014451076/article/details/50998957
  2. 接着，进行了python下的链接（最终个人希望是用py写一套系统）。开始的时候是参照官网给出的方式，http://neo4j.com/developer/ ，开始是成功实现的，后来个人电脑系统进行了升级，再次进行配置时，发现无法配	置成功。原因：使用bolt协议，这个协议只支持py2.7.9+和3.3+，但是本次自带的py是2.7.3和3.4，尝试手动进行py2.7.9的安装，但是安装neo4j-driver的过程需要使用pip，而pip的安装需要依赖py，而py的版本命令是取前两位（均为py2.7），没有找到相关的解决方案。最终	这种方法只能放弃，但是如果py版本正确，相关配置是可以实现的。
  3. 最终，选择了使用jpype和embeded进行py的开发。介绍和安装方法可简单参考：http://blog.csdn.net/dyllove98/article/details/8635965 和 http://docs.neo4j.org.cn/python-embedded.html
* 程序实现：
  代码托管在https://github.com/su526664687/NEUBAIDUNLP

#### Ver 0.11
* 最近有一整块的时间,我重新思考了关于Ver0.10版本中确定的关于通过jpype方法进行4j链接的方式的可扩展性,通过一些实验,发现这种方法有以下缺点:
  1. 如果链接默认的数据库graph.db,需要强制去修改数据库的lock,这个是为了确保互斥存在的
  2. 这种方法采用的cypher语法是旧版的,之前可以在 http://www.neo4j.org.cn 中查看相关文档,近期这个网站关闭了,网页重定向到了官网,我猜测可能是旧版的neo4j数据库被撇弃了.并且,旧版cpyher语法必须以start开头,可用性不好
  3. 大量api(crud)需要自己进行手动实现
* 基于以上的考虑,我花费了一些时间去重新配置neo4j数据库,对于遇到的问题查阅了许多的资料.总结如下:
  1. 首先是安装pip,这个比较简单,还是简单的说一下把, sudo apt-get install python-pip ,如果遇到问题,一般是软件源或者依赖引起的,自己手动更新下就行了
  2. 之后,下载了neo4j2.33版的,参考页面: http://neo4j.com/developer/ ,首先安装驱动,然后再把官方的例子拷贝进去,自己尝试一下就行了.如果这一步,运行成功,说明你没有遇到一下问题,就不需要继续看下边罗列的item了.
  3. 如果2遇到了问题,这些问题可能是:
    * 7687端口没有打开,服务没有运行
	  1. 这个问题实在是比较坑人的.于是,根据自己的理解,就把端口添加为 bolt://localhost:7474 ,继续运行
	  2. 提示: 协议错误,并且有个警告说:bolt协议只支持python2.7.9和3.3.
	  3. 于是,我就安装了python3.3,接着安装pip3,(话说,真的好麻烦),继续运行,依然是这次提示链接不安全
	  4. 由于之前我阅读了配置文件,记得7473端口是为了https,大家都知道这个是http的安全协议,于是我换成了7473端口,之后就开始提示协议错误
	  5. 这个地方,进行了各种的尝试,依然无解,要么是协议错误,要么就是不安全
	  6. 于是弃疗了
	* bolt
	  1. 由于问题1把我搞得头大了,我就让自己从另一个方面去考虑:bolt协议到底是什么
	  2. 各种google, 并没有找到太多对bolt协议的介绍
	  3. 之后继续去看neo4j官方的驱动文档,http://neo4j.com/docs/api/python-driver/current/ ,尝试之后,还是之前的错误
	  4. 只能继续把问题放在it社区去碰碰运气,最后找到了http://stackoverflow.com/questions/37128583/neo4j-bolt-driver-protocol-error .是国外的一个开发者,也遇到了协议错误的问题,有个开发者给出的解释是:7474 is by default used for http whereas 7687 is the default for binary bolt protocol. bolt driver is only for 3.0 (and newer).在这里,我得到了启发.
	  5. 重新去下载了neo4j3.0.1版本,进行使用python2.7.6进行了链接,提示我需要进行密码的更改,到这个地方,感觉离成功不远了,就去改了密码,果然成功了
	  6. 又拿python3.4进行了测试,提示未知主机,不过这个就不是问题了

## Ver 0.2  
---
time:2016.5
* change:  
  经过后期的讨论，现阶段决定使用统计的方法进行对query的处理。近期通读了《数学之美》，此书值得一读。在当今nlp界，自然语言处理从规则到统计已经成为一种必然的趋势。
* 基础：
  已经获取的三元组<s,r,e>
* 具体方式：
  1. 通过百度知道搜索s e的形式，收集到大量的“知道问题”，现阶段每个s e收集min(50,amount)
  2. 提取模板
* 文件位置：
 现阶段所有文件存储在172服务器~/sjming下	

