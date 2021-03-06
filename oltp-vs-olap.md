# 《敏捷数据仓库设计》第一章/第一节 OLTP Vs DW/BI 两个不同的世界
![image](https://user-images.githubusercontent.com/20431533/111724176-c0c14000-889f-11eb-8265-314d8f502618.png)   
在本章节中，我们旨在说明将敏捷方法运用于数仓设计领域的动机。我们将在文章的开头总结DW/BI系统和业务系统（OLTP）之间的基础性差异，在此基础上进一步解释这两者使用不同数据建模技巧的原因。接着我们会对比ER（Entity-relationship）建模和纬度建模，并解释为什么纬度建模更适合DW/BI系统。与此同时，我们会介绍纬度建模是如何支持增量设计和交付的，毕竟**增量**是敏捷软件开发的核心原则。   

对于熟悉传统纬度建模好处的读者，可以跳过本章节，直接阅读下一节[数仓需求分析和设计](https://github.com/linuxProber/agile-data-warehouse-design/tree/main)。在该章节中我们将会介绍敏捷纬度建模，通过回顾DW/BI的开发流程和审视传统数据需求分析方法，我们强调了传统方法在处理复杂数据源和应对快节奏数仓开发项目时的缺点。而为了解决这种问题，我们积极地让业务利益相关者参与到需求分析和设计过程。在本章的最后，我们将介绍一套利于跨部门协同的纬度建模的敏捷工具 - BEAM（Business Event Analysis and Modeling），该工具带有**协同**、**增量**和**迭代**的重要特性。

OLTP系统和DW/BI系统首先在目的性上就存在根本性不同。OLTP主要是为了支持业务流程的执行，而DW/BI则是为了评估分析业务流程的执行结果。为了提升执行效率，OLTP系统需要在联机交易处理的应用方向上不断优化。相反，数据仓库则需要在查询处理和易用性的方向进行优化。下表突出了两者在使用模式上的重要区别，并从数据管理系统的角度去描述两者需求的差异。   

| 比较纬度  | OLTP数据库 | 数据仓库 |
| ------------- | ------------- | -------- |
| 目的  | 执行独立的业务过程，如下单  | 分析评估多个业务过程，如观察带看量 |
| 事务类型  | insert、select、update、delete |  select |
| 事务风格  | 预定义、可预测、面向稳定的应用  |  不可预测、多变、面向多元的分析 |
| 优化方向  | 更新效率和写一致性  |  查询性能和可用性 |
| 更新频率  | 实时：需要及时相应业务需求  |  近实时：周期性更新 |
| 更新并发度  | 高  |  低 |
| 历史数据访问情况  | 当前和近段时期的数据  |  当前和近几年的数据 |
| 选择粒度  | 精准和微观实体  |  模糊和宏观群体 |
| 比较频率  | 低频  |  高频 |
| 查询条件复杂度  | 低  |  高 |
| 每事务的Join数量  | 低（1-3）  |  高（10+） |
| 每事务涉及记录数  | 十来条  |  百万级 |
| 日均事务数  | 百万级  |  上千条 |
| 数据量  | GB-TB  |  TB-PB |
| 数据类型  | 主要是原始明细数据  |  明细数据、汇总数据和衍生数据 |



## 1. 案例：ER建模的问题
ER建模是OLTP系统数据库设计的标准方法。
该方法将所有的数据类型分成了实体（entity）、关系（relationship）和属性（attribute）三种。
图1-1展示了实体级ER图的示例。图中实体用方块表示，关系则使用方块间连线来表示。
![image](https://user-images.githubusercontent.com/20431533/111636369-5d99c400-8833-11eb-969a-18ab8bd780e1.png)      
每对关系的基数（1对1的关系、1对多的关系和多对多的关系）则是在边的两端用不同的符号表示，如 | 表示1，O 表示0或者可选择，“鱼尾”则表示多个。
举例，每位房产经纪人在一段时间内只能一个门店入职，则经纪人实体和门店实体之间的关系是1对1；每位房产经纪人在一段时间内可能售出多套二手房源，则经纪人实体和房源实体之间是“1对多”的人关系；

在关系型数据库中，实体对应的是数据库表，属性对应的则是数据库表的列。
关系则可以使用表示实体的数据库表的列或者额外的数据库表的列来表示。
比如经纪人和身份信息是1:1的关系，则在经纪人表和身份信息表上可各有一列来存放对方的外链；比如经纪人和维护房源是1:N的关系，则在房源表上有一列存放经纪人的外链；再比如产品和订单之间是M:N的关系，则需要有一张额外的映射表来存放产品外链和订单外链的对应关系。

ER建模常常和范式化有关，尤其是第三范式。ER建模和范式化的技术目标很明确：尽可能降低数据冗余和显式描述数据里的1:1和1:N的关系。而这些要求目前也已经被RDMS系统所强制实现。  

### 1.1. ER建模应用于OLTP的优势 
**第三范式（3NF）对于事务处理是高效的。**      
OLTP需要写事务（insert、updates、deletes）非常高效。通过遵循3NF降低数据冗余，事务操作保持了最大限度的小和简单。   
譬如，通过引用C用户记录和S电信服务记录，就可表示C用户反复使用S电信服务的事务操作，而不需要每次都记录C用户和S电信服务的详细信息。而当C用户或者S服务的信息需要改变时，只需更改对应用户表或者服务表上的一条记录信息即可。假如我们并非通过引用记录而是通过不断复制记录副本的方式来记录事务，则更改信息时极有可能由于副本更改不完全而导致数据库状态的不一致。除了3NF之外，还有其他更高阶的范式，但3NF已能满足大部分ER建模人员的实际需求。对于3NF，关系型模型的发明者Edgar (Ted) Codd有一句非常著名的言论：“The key, the whole key, and nothing but the key, so help 
me Codd”。
 
### 1.2. ER建模应用于数据仓库的缺点    

**1） 3NF在查询处理方面的表现较为低效**       
虽然3NF有利于数据写入操作，但不利于数据读出操作，这使得3NF在数据仓库和BI应用方面有不可忽视的缺点。范式使得数据库表倾向于拆分成多个不同的实体类别，而JOIN查询多个数据库表的效率较低且易导致编码错误。譬如，回顾前面的ER图，读者能想象将**PRODUCT CATEGORY**连接（JOIN）到**ORDER**的方式有多少种吗？在实现了**PRODUCT CATEGORY**和**ORDER**之间的M:M的关系的3NF物理数据库表上，需要连接（JOIN）至少20张表。   

总的来说，当我们使用基于3NF的数据库时，即使是最简单的BI查询，往往也需要多张表连接另外的中间表。这些冗长的连接（JOIN）路径不仅难以优化而且运行起来及其缓慢。   

**2） 3NF模型不容易理解**      
更重要的，BI用户的查询只有正确使用连接（JOIN）路径时才能产生正确结果，也就是说，需要使用SQL语言问出正确的问题。如果使用了错误的连接（JOIN）路径，那么BI用户会在不知情的情况下得到其他无意义问题的答案。3NF无论是对人还是机器来说都是相对复杂的，虽然近些年数据仓库设备（专家硬件）在不断优化查询性能，但对人类而言复杂性是更加难以处理的问题。虽然智能BI软件能将数据库模式的复杂性隐藏在易用的交互下方，但也仅仅是将理解3NF的负担从BI用户的查询端转移到BI开发者的配置端。这虽然也算是进步，但对业务相关利益者（业务人员、BI用户、BI开发者、数据产品经理和数据工程师等）而言，3NF模型在回顾和质量保障（QA）方面的复杂度还是过大。    

## 2. 案例：纬度建模
   
**吸引BI用户的维度建模方法**    
维度建模方法基于度量（Fact）和描述（dimensions）来定义业务过程和组成业务过程的独立业务事件。描述（dimensions）在此处的作用是过滤、分组和聚合度量（Facts）。   
数据立方体常被用于可视化展示简单的维度模型。如图，该数字立方体展示了一个销售过程的多维分析。   
![image](https://user-images.githubusercontent.com/20431533/111723614-c8ccb000-889e-11eb-9e6d-91ee88ee51e1.png)   
这里一共包括了产品（what）、时间（when）和位置（location）三个维度（Dimensions）描述。而诸如销售数量、销售收入和销售成本等事实类度量（Facts）则是三个维度的交叉处。这种审视数据的视角对BI用户有较大吸引力，原因在于数据立方体可用多层的二维电子表格来展示，较为直观。譬如，为每个位置（location）创建一个电子表格，每个电子表格的行头是产品序列，列头是时间序列，而行列交叉处的空格（cell）则是销售收入。   
### 2.1. 星型建模      
**1）星型模式被用于可视化展示维度建模**      
电子表格可描述简单的维度模型，但在反映复杂多维业务过程的真实世界维度模型时，电子表格就显得乏力了。和超立方体可视化展示高维模型的困难相比，星型模式图可以轻松解决高维模型的展示问题。如图描绘了一个典型的星型模式图，该图除了包括前面数据立方体的三个维度（Dimensions）和度量（Facts）外，还额外包括了第四个维度-促销。   
![image](https://user-images.githubusercontent.com/20431533/111723700-f1ed4080-889e-11eb-9682-1e00f26a9a45.png)    

**2） 星型模式同样还是描述维度模型物理实现的术语。**
星型模式是维度模型的非范式ER模型的表示方式。当使用数据库建模工具时，可将星型模式转换为在RDMS中创建事实表和维度表的SQL语句。此外，还可将星型模式用于记录和定义多维数据库的数据立方体。     

### 2.2. 事实表和纬度表
一个星型模式图由居中的一张事实表和周围环绕的若干维度表组成。事实表包含了事实（对某事件的数字量化度量），而维度表则主要是包含对某事件的度量背景（文本非量化描述）。事实表中同样包含了维度表的外键，对ER建模人员而言，这些外键表示了维度表之间的M:N关系。事实表外键的子集构成了事实表的复合主键，同时也定义了事实表的粒度和层级。本书中的**维度（Dimension）**指的是维度表，而**维度属性（Dimensional Attribute）**则指的是维度表的列。   

维度表中包含了一系列描述性属性，这些属性被用于过滤数据和聚合事实度量，具有提供优秀报表的行头和标题/脚注描述的作用。除此之外，维度属性的结构层次关系海可支持BI工具进行下钻分析。譬如，从季度下钻到月，从国家下钻到门店，从类别下钻到产品，等等。    

并非所有的维度属性都是文本，维度表可包含数字和日期等，可这些数字和日期并不会像事实度量一样被计算，而是被当作文本数据来过滤和聚合事实度量。尽管有的维度表的宽度较大，但和事实表相比还是微小的存在：绝大部分的维度表的行数远小于百万级别，而事实表超过百万行体量的比比皆是。   

对BI用户而言，最好用的事实表是维度可自由组合的可累加事实表，最好用的维度表则是BI用户熟悉且属性丰富的维度表。   

### 2.3. 纬度建模应用于数据仓库的优势   
**1） 简单性**   
一张居中的事实表和环绕四周的若干张维度表，足可让人轻松想象如何度量销售额以及如何创建必需的查询语句。譬如，如果BI用户想按门店统计二手房销售情况，只需要将销售额事实表、门店维度表和二手房源类别维度表连接（JOIN）起来查询即可。使用纬度建模方式，可限制表数量和连接（Join）路径的长度，同时利用DBMS的星型连接优化特性，从而最大化查询性能。   

**2） 面向过程特性**       
星型模式是对物理数据模型的优化：将3NF的ER模型转化成更少表数量，从而克服数据库在处理较长连接路径（JOIN PATH）的BI查询问题。除此之外，维度模型还是某类探索问题的答案：究竟哪一类业务过程需要被测量，如何使用业务术语描述业务过程以及如何度量业务过程。通过回答这类问题，得到的事实表和维度表并非由随机内容，而是系统性的基于**7Ws**框架对值得度量的独立业务事件进行描述的细节。   

7Ws框架内容包括Who、What、When、Where、When、How Many、How 和 Why一共7个方面，分别需要回答7个方面的问题：   
**Who** Who is involved？ 谁参与其中？  
**What** What did they do？ 他们干了什么？    
**When** When did it happen？ 什么时候发生的？    
**Where** Where did it take place？在哪儿发生的？  
**How Many** How many/much is recorded？ 有多少被记录了？  
**How** How did it happen？ 怎么发生的？  
**Why** Why did it happen？ 为什么会发生这样的事情？
7Ws框架是论文作者或者记者用来描述完整故事的5W1H方法论的拓展，其中每一个W都代表了一个问题的类型。7Ws对数据仓库建模极其有用的原因，在于7Ws框架将设计聚焦于BI业务的重要活动-**问问题**上。   

**事实表表示动作（verbs），动作是用来记录业务过程动作的。事实表里的事实度量和周围环绕的维度表都是名词（Nouns），事实表里的事实用How Many表示，维度表里的维度可用Who、What、When、Where、How和Why来表示。我们后面将要隆重介绍的BEAM数据故事既是用7Ws框架来探索重要的动作（Verbs）和名词（Nouns）组合**   

具体的维度模型往往包括超过6个维度表，因为6Ws中每个方向都有可能出现多次。譬如，下单过程可用3个Who维度表（顾客-Customer、雇员-Employee和搬运工-Carrier），2个When维度表（下单日期-ORDER DATE、交付日期-DELIVERY DATE）。但话说回来，大部分维度模型不会有超过10-12个维度表，即使是最复杂的业务事件也很少有超过20个维度表的。    

面向业务过程的维度建模的最大好处，就是其自然地将数据仓库需求范围、设计和开发分割成可管理的部分，每部分由需要度量的独立业务过程组成。将每个业务过程建模成星型模式的方式，天然支持增量地设计、开发和投入使用。敏捷纬度建模人员和BI相关利益者可在一段时间内聚焦在一个业务过程上，从而充分思考和理解如何度量该业务过程。和瀑布型设计方式相比，敏捷开发团队更快地建设和增量交付星型模式。敏捷BI用户在初期可独立分析每个业务过程，这个给BI用户带来了早期价值，而后期又可进行更有价值和复杂的跨过程分析。敏捷的意义很明显，项目团队明明可在低风险投入的情况下更快地增量交付，又为什么要一次性全量交付呢？   
   
**维度建模给出了明确的交付粒度定义：星型模式。该交付粒度和敏捷原则不谋而合：一是通过尽可能早和持续的软件价值交付来满足用户需求，二是通过更短的迭代周期和更多的交付次数（从几周到几个月）来交付软件价值**   
