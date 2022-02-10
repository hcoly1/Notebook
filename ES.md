# ES

## 对象模型

ElasticSearch的对象模型，跟关系型数据库模型相比：

- **索引（Index）**：相当于数据库，用于定义文档类型的存储；在同一个索引中，同一个字段只能定义一个数据类型；

- **文档类型（Type）**：相当于关系表，用于描述文档中的各个字段的定义；不同的文档类型，能够存储不同的字段，服务于不同的查询请求；

- **文档（Document）**

  相当于关系表的数据行，存储数据的载体，包含一个或多个存有数据的字段；

  - **字段（Field）**：文档的一个Key/Value对；
  - **词（Term）**：表示文本中的一个单词；
  - **标记（Token）**：表示在字段中出现的词，由该词的文本、偏移量（开始和结束）以及类型组成；

索引是由段（Segment）组成的，段存储在硬盘（Disk）文件中，段不是实时更新的，这意味着，段在写入磁盘后，就不再被更新。ElasticSearch引擎把被删除的文档的信息存储在一个单独的文件中，在搜索数据时，ElasticSearch引擎首先从段中查询，再从查询结果中过滤被删除的文档，这意味着，段中存储着“被删除”的文档，这使得段中含有”正常文档“的密度降低。多个段可以通过段合并（Segment Merge）操作把“已删除”的文档将从段中物理删除，把未删除的文档合并到一个新段中，新段中没有”已删除文档“，因此，段合并操作能够提高索引的查找速度，但段合并是IO密集型的操作，需要消耗大量的硬盘IO。

### 创建索引

在创建索引之前，首先了解**RESTful API**的调用风格，在管理和使用ElasticSearch服务时，常用的HTTP动词有下面五个：

- **GET 请求：获取服务器中的对象**

  - 相当于SQL的Select命令
  - GET /blogs：列出所有博客

- **POST** 请求：在服务器上更新对象

  - 相当于SQL的Update命令
  - POST /blogs/ID：更新指定的博客

- **PUT 请求：在服务器上创建对象**

  - 相当于SQL的Create命令
  - PUT /blogs/ID：新建一个博客　　

- **DELETE 请求：删除服务器中的对象**

- - 相当于SQL的Delete命令
  - DELETE /blogs/ID：删除指定的博客

- **HEAD 请求：仅仅用于获取对象的基础信息**

### 索引映射节（mappings）

#### 索引结构

索引是由**文档类型**构成的，在mappings字段中定义索引的文档类型，示例代码中为blog索引定义了三个文档类型：articles，followers和comments

```json
{  
   "mappings":{  
      "articles":{ },
      "followers":{ },
      "comments":{ }
   }
}
```

#### 文档属性

**文档属性定义了文档类型的共用属性，适用于文档的所有字段:**·

- **dynamic_date_formats**属性：该属性定义可以识别的日期格式列表；

- dynamic属性

  默认值为true，允许动态地向文档类型中加入新的字段。推荐设置为false，禁止向文档中添加字段，这样，文档类型的所有字段必须在索引映射的properties属性中显式定义，在properties字段中未定义的字段都将会ElasticSearch忽略。

  - dynamic设置为ture：默认值，新增加的字段被添加到索引映射中；
  - dynamic设置为false：新增加的字段会被忽略；
  - dynamic设置为strict：当向文档中新增字段时，ElasticSearch引擎抛出异常；

```json
{  
   "mappings":{  
      "articles":{  "dynamic":false,
         "dynamic_date_formats":["yyyy-MM-dd hh:mm:ss", "yyyy-MM-dd" ],
         "properties":{  
            "id":{},
            "title":{},
            "author":{},
            "content":{},
            "postedat":{}
         }
      }
   }
}
```

**文档的字段属性**

**1，字段的数据类型**

字段的数据类型由字段的属性type指定，ElasticSearch支持的基础数据类型主要有：

- **字符串类型**：string；
- **数值类型**：字节（byte）、2字节（short）、4字节（integer）、8字节（long）、float、double；
- **布尔类型**：boolean，值是true或false；
- **时间/日期类型**：date，用于存储日期和时间；
- **二进制类型**：binary；
- **IP地址类型**：ip，以字符串形式存储IPv4地址；
- **特殊数据类型**：token_count，用于存储索引的字数信息

在文档类型的properties属性中，定义字段的type属性，指定字段的数据类型，属性properties 用于定义文档类型的字段属性，或字段对象的属性：

```json
{
  "properties" : {
    "id":{"type":"long"}
  }
}
```

**2，字段的公共属性**

- **index**：该属性控制字段是否编入索引被搜索，该属性共有三个有效值：analyzed、no和not_analyzed：
  - analyzed：表示该字段被分析，编入索引，产生的token能被搜索到；
  - not_analyzed：表示该字段不会被分析，使用原始值编入索引，在索引中作为单个词；
  - no：不编入索引，无法搜索该字段；
  - 其中analyzed是分析，分解的意思，默认值是analyzed，表示将该字段编入索引，以供搜索。
- **store**：指定是否将字段的原始值写入索引，默认值是no，字段值被分析，能够被搜索，但是，字段值不会存储，这意味着，该字段能够被查询，但是不会存储字段的原始值。
- **boost**：字段级别的助推，默认值是1，定义了字段在文档中的重要性/权重；
- **include_in_all**：该属性指定当前字段是否包括在_all字段中，默认值是ture，所有的字段都会包含_all字段中；如果index=no，那么属性include_in_all无效，这意味着当前字段无法包含在_all字段中。
- **copy_to**：该属性指定一个字段名称，ElasticSearch引擎将当前字段的值复制到该属性指定的字段中；
- **doc_values**：文档值是存储在硬盘上的索引时（indexing time）数据结构，对于not_analyzed字段，默认值是true，analyzed string字段不支持文档值;
- **fielddata**：字段数据是存储在内存中的查询时（querying time）数据结构，只支持analyzed string字段；
- **null_value**：该属性指定一个值，当字段的值为NULL时，该字段使用null_value代替NULL值；在ElasticSearch中，NULL 值不能被索引和搜索，当一个字段设置为NULL值，ElasticSearch引擎认为该字段没有任何值，使用该属性为NULL字段设置一个指定的值，使该字段能够被索引和搜索。

**3，字符串类型常用的其他属性**

- **analyzer**：该属性定义用于建立索引和搜索的分析器名称，默认值是全局定义的分析器名称，该属性可以引用在配置结点（settings）中自定义的分析器；
- **search_analyzer**：该属性定义的分析器，用于处理发送到特定字段的查询字符串；
- **ignore_above**：该属性指定一个整数值，当字符串字段（analyzed string field）的字节数量大于该数值之后，超过长度的部分字符数据将不能被analyzer处理，不能被编入索引；对于 not analyzed string字段，超过长度的部分字符将被忽略，不会被编入索引。默认值是0，禁用该属性；
- **position_increment_gap**：该属性指定在相同词的位置上增加的gap，默认值是100；
- **index_options**：索引选项控制添加到倒排索引（Inverted Index）的信息，这些信息用于搜索（Search）和高亮显示：
  - docs：只索引文档编号(Doc Number)
  - freqs：索引文档编号和词频率（term frequency）
  - positions：索引文档编号，词频率和词位置（序号）
  - offsets：索引文档编号，词频率，词偏移量（开始和结束位置）和词位置（序号）
  - 默认情况下，被分析的字符串（analyzed string）字段使用positions，其他字段使用docs; 

分析器（analyzer）把analyzed string 字段的值，转换成标记流（Token stream），例如，字符串"The quick Brown Foxes"，可能被分解成的标记（Token）是：quick,brown,fox。这些词（term）是该字段的索引值，这使用对索引文本的查找更有效率。字段的属性 analyzer 用于指定在index-time和search-time时，ElasticSearch引擎分解字段值的分析器名称。

**4，数值类型的其他属性**

- **precision_step**：该属性指定为数值字段每个值生成的term数量，值越低，产生的term数量越高，范围查询越快，索引越大，默认值是4；
- **ignore_malformed**：忽略格式错误的数值，默认值是false，不忽略错误格式，对整个文档不处理，并且抛出异常；
- **coerce**：默认值是true，尝试将字符串转换为数值，如果字段类型是整数，那么将小数取整；

**5，日期类型的其他属性**

- **format**：指定日期的格式，例如：“yyyy-MM-dd hh:mm:ss”
- **precision_step**：该属性指定数值字段每隔多少数值，生成一个词（term）；step值越低，产生的词数量越高，范围查询越快，索引越大，占用存储空间越大；
- **ignore_malformed**：忽略错误格式，默认值是false，不忽略错误格式；

**6，多字段（fields）**

在fields属性中定义一个或多个字段，该字段的值和当前字段值相同，可以设置一个字段用于搜索，一个字段用于排序等。

```json
{
  "properties": {
    "id":{  "type":"long",
      "fields":{  "id2":{"type":"long","index":"not_analyzed"} }
    }
  }
}
```

**7，文档值（doc_values）** 

默认情况下，多数字段都被一起编入索引，用户使用倒排索引（Inverted Index）可以搜索到相应的词（Term），倒排索引支持在唯一的有序词列表中查找特定词，或检查文档中是否包含某个词，但是，对于排序（Sort），聚合和在脚本中访问特定字段的值（Field value)，这三个操作需要执行不同的数据访问模式，即单字段数据访问：在文档中查找特定的字段，检查该字段是否包含指定的词。

文档值（doc_values）属性指定将字段的值写入到**硬盘上**的列式结构，实现了单个字段的数据访问模式，能够高效执行排序和聚合搜索。使用文档值的字段将有专属的字段数据缓存实例，无需像普通字段一样倒排。是存储在硬盘上的数据结构，在文档索引时创建。文档值数据存在硬盘上，在**文档索引时**创建，存储的数据和字段存储在_source 字段的数据相同，文档值支持所有的字段类型，除了analyzed string 字段之外。

默认情况下，所有的字段都支持文档值，默认是启用的（enabled），如果不需要在单个字段上执行排序或聚合操作，或者从脚本中访问指定字段的值，那么，可以禁用文档值，字段的值将不会存储在硬盘空间中。

**8，字段数据（Fielddata）**

字段数据（Fielddata）是存储在内存中的查询时数据结构，只支持analyzed string字段。该数据结构在字段第一次执行聚合，排序或被脚本访问时创建。创建的过程是：在读取整个倒排索引（Inverted Index）时，ElasticSearch从硬盘上加载倒排索引的每个段（Segment），倒转词（Term）和文档的关系，并将其存储在JVM堆内存中。加载字段数据的过程是非常消耗IO资源的，一旦被加载，就被存储在内存中，直到段的生命周期结束。

对于analyzed string字段，fielddata字段是默认启用的，

```json
{
  "text": {  
   "type":"string",
   "fielddata":{  "loading":"lazy"
   }
  }
}
```

详细信息，请参考[Mapping parameters » fielddata](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/fielddata.html)

Analyzed strings use a query-time data structure called fielddata. This data structure is built on demand the first time that a field is used for aggregations, sorting, or is accessed in a script. It is built by reading the entire inverted index for each segment from disk, inverting the term ↔︎ document relationship, and storing the result in memory, in the JVM heap.

**9，存储（store）**

存储（store）属性指定是否将字段的原始值写入索引，默认值是no，字段值被分析，能够被搜索，但是，字段的原始值不会存储，这意味着，该字段能够被查询，但是无法获取字段的原始值。默认情况下，该字段的值会被存储到_source字段中，如果想要获取单个或多个字段的值，而不是整个_source字段，可以使用 [source filtering](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/search-request-source-filtering.html) 来实现；但是在特定的条件下，只存储一个字段的值是有意义的（make sense），例如，一个article文档包含：title，postdate和content字段，从文档中只获取title和postdate字段，并且使_source 字段包含content字段，必须通过store属性来控制：

```json
{
  "mappings": {
    "my_type": {
      "properties": {
        "title": {
          "type": "string",
          "store": true
        },
        "date": {
          "type": "date",
          "store": true
        },
        "content": {
          "type": "string",          "store": false
        }
      }
    }
  }
}
```

**10，位置增加间隔（position_increment_gap）**

对于analyzed string字段，都会考虑把词的位置信息，用于支持位置和短语匹配查询（[proximity or phrase queries](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/query-dsl-match-query.html#query-dsl-match-query-phrase)），例如，有一个字符串字段，该字段中存在多个词“fake”，ElasticSearch引擎会在每个值之间增加一个gap，以防止短语匹配或位置匹配查询出现跨越多个词的异常，这个gap的值就是属性position_increment_gap，默认值是100；



