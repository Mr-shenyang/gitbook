# 目标
Commit Log的格式精确控制，增加可读性，便于查看变更历史

# Commit Log Format
Commit Log包含三部分header、body、footer，其中header是必须的，格式固定，body在变更有必要详细解释时使用。

```uml
<types>(<scopes>): <subject>
<空行>
<body>
<空行>
<footer>
```

注意：冒号后面必须有一个小写空格，types和scopes可为多个，中间用逗号分隔。

## header

### Type

英文，小写。必须为下列中一个或多个：

***func:*** function，小功能，一个feature拆分成多个小功能点，分批提交

***fix:*** bug修复，包括编码过程中的逻辑修复，不特指线上bug修复

***refactor:*** 重构代码，非bug修复和性能优化，包括编码过程中的代码结构调整，不特指重构项目

***perf:*** 性能优化

***apm:*** 仅监控打点、异常日志处理相关

***chore:*** 无关紧要的改动，例如删除用不到的注解、调整日志内容等

***impr:*** improvement，代码设计改进

***jvm:*** 仅JVM参数变更

***pom:*** 仅依赖和版本变化

***conf:*** 仅配置变化，Spring配置、properties文件

***docs:*** 仅文档变更

***style:*** 代码格式调整，如import清理，代码格式化

***test:*** 单测和自动化case相关

***typo:*** 修复小的拼写错误

***wip:*** work in progress，少用，用于开发中的不完整提交，新工程开始时偶尔使用

### Scope
英文，小写。表示变更的包或模块范围，可多个组合，若涉及范围较大，可用 * 代替。各服务可以自行定义，组内同学可轻易理解。通用scope列表如下：

***dto:*** dto结构变化

***core:*** core包

***service:*** service层代码

***dao:*** dao层代码

***sql:*** sql代码变更

***idl:*** IDL文件变化

## Subject
中文。标题简述修改，结尾不要有句号。

## Body
中文。修改的背景（为什么做这次修改），说明修改逻辑。

## Footer
中文。可以放置需求wiki或task链接，对以后其他同学blame很有用。

# demo

* 仅header
  ```
  fix(service,dao): 修复××××
  ```
* 仅header，涉及模块较多用*代替
  ```
  func(*): 实现××××功能
  ```
* 有header和body
  ```
  func(*): 实现××××功能

  body
  ```
* 有header、body和footer
  ```
  func(*): 实现××××功能

  body

  footer
  ```

# 实战经验

* 推荐采用header模式；
* 避免一个任务只提交一次，最好能按照type划分进行commit操作；
