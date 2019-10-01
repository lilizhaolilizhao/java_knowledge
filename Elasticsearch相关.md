##Elasticsearch - 搜索类型与搜索位置
###搜索类型
Elasticsearch允许用户选择其所希望的处理查询的方式。因为存在一些不同的情形，对其使用不同的搜索类型才是合适的。为了控制查询的执行方式，我们可以在请求中使用search_type参数，以有下类型可以选择。

> **1、query\_and\_fetch：** 通常是最快也是最简单的搜索类型。查询语句在所有需检查的分片上并行执行，并且所有分片返回结果的规划为size参数的取值。因此，该类型返回的文档数目最大为size参数的取值与分片数目的乘积。    
> **2、query\_then\_fetch：** 查询语句首先得到将文档排序所需的信息，然后得到要获取的文档内容的相关分片。与query_and_fetch不同，该类搜索返回的文档数目最大为size参数的取值。   
> **3、dfs\_query\_and\_fetch：** 该类搜索类似于query_and_fetch。除了完成query_and_fetch的工作外，还执行初始查询阶段，该阶段计算分布式的词频以更精准地对返回文档打分。   
> **4、dfs\_query\_then\_fetch：** 该类搜索类似于query_then_fetch。除了完成query_then_fetch的工作外，还执行初始查询阶段，该阶段计算分布式的词频以更精准地对返回文档打分。   
> **5、count：**这是一种特殊的搜索类型，只返回匹配查询的文档数目。 

###Elasticsearch之四种查询类型和搜索原理
Elasticsearch Client发送搜索请求，某个索引库，一般默认是5个分片（shard）。

它返回的时候，由各个分片汇总结果回来。

![](http://images2015.cnblogs.com/blog/855959/201707/855959-20170705222926456-1485587310.png)

 es 在查询时， 可以指定搜索类型为下面四种：  
　　QUERY\_THEN\_FETCH  
　　QUERY\_AND\_FEATCH  
　　DFS\_QUERY\_THEN\_FEATCH  
　　DFS\_QUERY\_AND\_FEATCH   

**那么这 4 种搜索类型有什么区别？**   
在讲这四种搜索类型的区别之前， 先分析一下分布式搜索背景介绍：
ES 天生就是为分布式而生， 但分布式有分布式的缺点。 比如要搜索某个单词， 但是数据却分别在 5 个分片（Shard)上面， 这 5 个分片可能在 5 台主机上面。 因为全文搜索天生就要排序（ 按照匹配度进行排名） ,但数据却在 5 个分片上， 如何得到最后正确的排序呢？ ES是这样做的， 大概分两步：   
　　step1、 ES 客户端将会同时向 5 个分片发起搜索请求。   
　　step2、 这 5 个分片基于本分片的内容独立完成搜索， 然后将符合条件的结果全部返回。

客户端将返回的结果进行重新排序和排名，最后返回给用户。也就是说，ES的一次搜索，是一次scatter/gather过程（这个跟mapreduce也很类似）

**然而这其中有两个问题：**
> **第一、 数量问题。** 比如， 用户需要搜索"衣服"， 要求返回符合条件的前 10 条。 但在 5个分片中， 可能都存储着衣服相关的数据。 所以 ES 会向这 5 个分片都发出查询请求， 并且要求每个分片都返回符合条件的 10 条记录。当ES得到返回的结果后，进行整体排序，然后取最符合条件的前10条返给用户。 这种情况， ES 中 5 个 shard 最多会收到 10*5=50条记录， 这样返回给用户的结果数量会多于用户请求的数量。   
> **第二、 排名问题。** 上面说的搜索， 每个分片计算符合条件的前 10 条数据都是基于自己分片的数据进行打分计算的。计算分值使用的词频和文档频率等信息都是基于自己分片的数据进行的， 而 ES 进行整体排名是基于每个分片计算后的分值进行排序的(相当于打分依据就不一样， 最终对这些数据统一排名的时候就不准确了)， 这就可能会导致排名不准确的问题。如果我们想更精确的控制排序， 应该先将计算排序和排名相关的信息（ 词频和文档频率等打分依据） 从 5 个分片收集上来， 进行统一计算， 然后使用整体的词频和文档频率为每个分片中的数据进行打分， 这样打分依据就一样了。

###Elasticsearch在搜索问题的解决思路

**（1）返回数据数量问题**

第一步：先从每个分片汇总查询的数据id，进行排名，取前10条数据

第二步：根据这10条数据id，到不同分片获取数据

**（2）返回数据排名问题**

将各个分片打分标准统一

### Elasticsearch的搜索类型（SearchType类型）
**1、 query and fetch**  
　　向索引的所有分片 （ shard）都发出查询请求， 各分片返回的时候把元素文档 （ document）和计算后的排名信息一起返回。   
　　这种搜索方式是最快的。 因为相比下面的几种搜索方式， 这种查询方法只需要去 shard查询一次。 但是各个 shard 返回的结果的数量之和可能是用户要求的 size 的 n 倍。
优点：这种搜索方式是最快的。因为相比后面的几种es的搜索方式，这种查询方法只需要去shard查询一次。  
缺点：返回的数据量不准确， 可能返回(N\*分片数量)的数据并且数据排名也不准确，同时各个shard返回的结果的数量之和可能是用户要求的size的n倍。  

**2、 query then fetch（ es 默认的搜索方式）**   
　　如果你搜索时， 没有指定搜索方式， 就是使用的这种搜索方式。 这种搜索方式， 大概分两个步骤：   
　　第一步， 先向所有的 shard 发出请求， 各分片只返回文档 id(注意， 不包括文档 document)和排名相关的信息(也就是文档对应的分值)， 然后按照各分片返回的文档的分数进行重新排序和排名， 取前 size 个文档。   
　　第二步， 根据文档 id 去相关的 shard 取 document。 这种方式返回的 document 数量与用户要求的大小是相等的。   
　　优点：   
　　　　返回的数据量是准确的。   
　　缺点：   
　　　　性能一般，并且数据排名不准确。  

**3、 DFS query and fetch**
　　这种方式比第一种方式多了一个 DFS 步骤，有这一步，可以更精确控制搜索打分和排名。也就是在进行查询之前， 先对所有分片发送请求， 把所有分片中的词频和文档频率等打分依据全部汇总到一块， 再执行后面的操作、
　　优点：   
　　　　数据排名准确   
　　缺点：   
　　　　性能一般   
　　　　返回的数据量不准确， 可能返回(N\*分片数量)的数据   

**4、 DFS query then fetch  **
　　比第 2 种方式多了一个 DFS 步骤。     
　　也就是在进行查询之前， 先对所有分片发送请求， 把所有分片中的词频和文档频率等打分依据全部汇总到一块， 再执行后面的操作、    

优点：      
　　　　返回的数据量是准确的     
　　　　数据排名准确     
缺点：  
　　　　性能最差【 这个最差只是表示在这四种查询方式中性能最慢， 也不至于不能忍受，如果对查询性能要求不是非常高， 而对查询准确度要求比较高的时候可以考虑这个】

DFS 是一个什么样的过程？   
　　从 es 的官方网站我们可以发现， DFS 其实就是在进行真正的查询之前， 先把各个分片的词频率和文档频率收集一下， 然后进行词搜索的时候， 各分片依据全局的词频率和文档频率进行搜索和排名。 显然如果使用 DFS\_QUERY\_THEN\_FETCH 这种查询方式， 效率是最低的，因为一个搜索， 可能要请求 3 次分片。 但， 使用 DFS 方法， 搜索精度是最高的。

总结一下， 从性能考虑 QUERY\_AND\_FETCH 是最快的， DFS\_QUERY\_THEN\_FETCH 是最慢的。从搜索的准确度来说， DFS 要比非 DFS 的准确度更高。

       检索出结果后，通过response..getHits()可以得到所有的SearchHit，得到Hit后，便可迭代Hit取到对应的Document，转化成为需要的实体。  前面提到如何进行搜索，并将SearchRequestBuilder的一些方法进行了列举，本文调用了SearchRequestBuilder的用于高亮的方法，处理了检索中的高亮问题：

**\[java\]** [view plain](http://blog.csdn.net/liyantianmin/article/details/45133993# "view plain") [copy](http://blog.csdn.net/liyantianmin/article/details/45133993# "copy")

 [print](http://blog.csdn.net/liyantianmin/article/details/45133993# "print") [?](http://blog.csdn.net/liyantianmin/article/details/45133993# "?")

1.  SearchResponse response1 = client.prepareSearch("user")
2.          .setTypes("tb\_person0", "tb\_person1" , "tb\_person2", "tb\_person3",  "tb\_person4")
3.          .setSearchType(SearchType.DFS\_QUERY\_THEN\_FETCH)
4.          .setQuery(QueryBuilders.fieldQuery("name",  "张三"))             // Query
5.          .addHighlightedField("name")
6.          .setHighlighterPreTags("<span style="color:red ">")
7.          .setHighlighterPostTags("</span>")
8.          .setFilter(FilterBuilders.rangeFilter("age").from( 20).to(22))   // Filter
9.          .setFrom(0).setSize(60 ).setExplain(true)
10.          .execute()
11.          .actionGet();

13.  SearchHits hits1 = response1.getHits();
14.        for(SearchHit hit : hits1){
15.      String json = hit.getSourceAsString();

17.      Person newPerson = mapper.readValue(json,
18.              Person.class);

20.            Map<String, HighlightField> result = hit.highlightFields();
21.            HighlightField titleField = result.get("name");
22.            Text\[\] titleTexts =  titleField.fragments();

24.            String name = "";

26.            for(Text text : titleTexts){
27.                name += text;
28.            }
29.            newPerson.setName(name);

31.      System.out.println("namett" + newPerson.getName());
32.      System.out.println("sextt" + newPerson.getSex());
33.      System.out.println("agett" + newPerson.getAge());
34.      System.out.println("isStudenttt" + newPerson.getIsStudent());

37.            System.out.println("\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-\-");
38.        }

        addHighlightedField(String fieldName)指明要进行高亮处理的Field；setHighlighterPreTags设定了高亮文字的前缀；setHighlighterPostTags设定了高亮文字的后缀。

  取得hit后，使用hit.highlightFields()取得结果中进行了高亮标识的域名\-域值对，然后对这些域名\-域值对进行分析得到高亮的域结果。
