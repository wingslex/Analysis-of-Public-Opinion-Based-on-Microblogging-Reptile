# Analysis-of-Public-Opinion-Based-on-Microblogging-Reptile
这是我参加招商银行fintech精英选拔时，做的一个课题。用Python对新浪微博进行爬虫，然后进行舆情分析。爬虫之前，需要模拟登陆，这里采用RSA加密模块模拟登陆。舆情分析的时候，我直接调用腾讯文智的感情分析API。


课题背景： 请设计微博爬虫，获取微博上最近N天(N<=10)内与招商银行相关的热点新闻与用户意见，代码可根据输入的天数返回最新的微博信息。在已收集的数据中对提及的重点内容（招行相关产品、服务和重点事件等）进行抽取并进行一定程度上的情绪判定。最终以浅显易懂的方式呈现该段时间范围内微博上与招行相关的舆情信息，其中呈现的具体内容和方式可由考生自行设计。
请提交解决方案：课题需将爬虫构建相关PPT和代码作为研究的最终成果，并在PPT中简要展示以下几点内容：
（1）构建爬虫和舆情分析的主要流程与模块；
（2）项目过程中使用到的工具，遇到的困难和问题，以及解决的方式；
（3）简要评价爬虫的效率与性能；
（4）最终基于微博信息进行舆情分析的结果展示；



1.爬虫与舆情分析的主要流程
1.1爬虫切入点
本课题主要对新浪微博进行爬虫，但是遗憾的是新浪微博并没有提供以“关键字+时间+区域”方式获取官方API。但是庆幸的是，新浪提供了高级搜索功能。

           


点击搜索微博后，我们看地址栏：
http://s.weibo.com/weibo/%25E6%258B%259B%25E5%2595%2586%25E9%2593%25B6%25E8%25A1%258C&typeall=1&suball=1&timescope=custom:2017-05-02:2017-05-02&Refer=g

解析如下固定地址部分：      http://s.weibo.com/wb/
关键字（2次URLEncode编码）：  %25E6%258B%259B%25E5%2595%2586%25E9%2593%25B6%25E8%25A1%258C
搜索时间范围：           timescope=custom:2017-05-02:2017-05-02
可忽略项：             Refer=g
某次请求的页数（未出现）：  page=1（某页请求页数）

既然是这么回事，我们接下来就可以使用网页爬虫的方式获取“关键字+时间”的微博了……
1.2爬虫思路
本课题所采用的爬虫语言是Python。在对新浪微博进行爬虫之前，首先需要模拟登陆，这里所采用的办法是：使用rsa加密模块进行模拟登陆。接下来要构造URL，爬取网页，然后解析网页中的微博信息，如图所示。
另外，高级搜索最多返回50页微博。时间范围（timescope）可设置为1天，如2017-05-02:201-05-02。

舆情分析思路流程
                         
信息抽取
爬虫得到微博信息存储在weiboData.xls这个EXCEL文件中，我抽取的是5017-05-02开始的最近10天的信息，一共691条微博信息。要想进行舆情分析，就必须对爬虫信息进行抽取。我通关关键词正则匹配的方式，从爬虫得到的信息中抽取了和招行相关相关的服务，黑金卡、信用卡等重点信息。
但是，在实现过程中发现正则表达式对中文汉字并不适用。查资料后，发现可以对汉字进行Unicode编码，经过编码后就可以进行正则匹配了。以关键词“服务”为例，其Unicode编码为\u670d\u52a1，正则表达式为：
[python] view plain copy 在CODE上查看代码片派生到我的代码片
pattern= re.compile(u"(\u670d\u52a1)+")  


情绪判定

     这个情绪分析算法就比较复杂了，自己在短时间内做不来。我选择了在大连理工情感词汇本体库，但是由于词库，词不够全，以及我自己算法的一些问题，获得的结果很差。后来查资料后，发现，腾讯有腾讯文智情感分析API，新手可以获得免费调用机会。按照官方文档，调用后，成功就算出每条微博的正面情绪和负面情绪。然后对相关微博的情绪值求平均，就可以得到对应微博信息的正面和负面的情绪值。

腾讯文智情感分析API
    腾讯文智情感分析API，初次使用，可以申请免费试用。具体用法，请参考如下官网：
1.https://www.qcloud.com/document/product/271/2072
2.https://www.qcloud.com/document/developer-resource/494/7244


以招商的服务关键词为例，在已经爬虫得到的和招商银行有关的981条微博中抽取
和“服务”有关的微博，进行情绪分析。代码如下：
