# MySQL概述

![mysql架构](/images/mysql/architecture_view.png)

* 1 连接器
  
* 2 查询缓存
  
* 3 分析器
  分析器包括两段**词法分析**和**语法分析**
* 4 优化器
  
* 5 执行器
  
* 6 存储引擎
  在存储引擎层，MySQL采用插件的形式，支持多种存储引擎，不同的存储引擎，功能也不同，自从MySQL5.5后InnoDB成为默认存储引擎。

