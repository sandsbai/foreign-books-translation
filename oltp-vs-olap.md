# 第一节 OLTP Vs DW/BI 两个不同的世界
>In this first chapter we set out the motivation for adopting an agile approach to 
data warehouse design. We start by summarizing the fundamental differences 
between data warehouses and online transaction processing (OLTP) databases to 
show why they need to be designed using very different data modeling techniques. 
We then contrast entity-relationship and dimensional modeling and explain why
dimensional models are optimal for data warehousing/business intelligence
(DW/BI). While doing so we also describe how dimensional modeling enables 
incremental design and delivery: key principles of agile software development.   

在本章节中，我们旨在说明将敏捷方法运用于数仓设计领域的动机。我们将在文章的开头总结DW/BI系统和业务系统（OLTP）之间的基础性差异，在此基础上进一步解释这两者使用不同数据建模技巧的原因。接着我们会对比ER（Entity-relationship）建模和纬度建模，并解释为什么纬度建模更适合DW/BI系统。与此同时，我们会介绍纬度建模是如何支持增量设计和交付的，毕竟**增量**是敏捷软件开发的核心原则。   

>Readers who are familiar with the benefits of traditional dimensional modeling 
may wish to skip to Data Warehouse Analysis and Design on Page 11 where we begin 
the case for agile dimensional modeling. There, we take a step back in the DW/BI
development lifecycle and examine the traditional approaches to data requirements analysis, and highlight their shortcomings in dealing with ever more complex data sources and aggressive BI delivery schedules. We then describe how agile
data modeling can significantly improve matters by actively involving business 
stakeholders in the analysis and design process. We finish by introducing BEAM
(Business Event Analysis and Modeling): the set of agile techniques for collaborative dimensional modeling described throughout this book.      

对于熟悉传统纬度建模好处的读者，可以跳过本章节，直接阅读下一节[数仓需求分析和设计](https://github.com/linuxProber/agile-data-warehouse-design/tree/main)。在该章节中我们将会介绍敏捷纬度建模，通过回顾DW/BI的开发流程和审视传统数据需求分析方法，我们强调了传统方法在处理复杂数据源和应对快节奏数仓开发项目时的缺点。而为了解决这种问题，我们积极地让业务利益相关者参与到需求分析和设计过程。在本章的最后，我们将介绍一套利于跨部门协同的纬度建模的敏捷工具 - BEAM（Business Event Analysis and Modeling），该工具带有**协同**、**增量**和**迭代**的重要特性。
