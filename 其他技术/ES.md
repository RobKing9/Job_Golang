## ES和Mysql

- MySQL 中的数据库（DataBase），等价于 ES 中的索引（Index）。

- MySQL 中一个数据库下面有 N 张表（Table），等价于1个索引 Index 下面有 N 多类型（Type）。

- MySQL 中一个数据库表（Table）下的数据由多行（Row）多列（column，属性）组成，等价于1个 Type 由多个文档（Document）和多 Field 组成。

- MySQL 中定义表结构、设定字段类型等价于 ES 中的 Mapping。举例说明，在一个关系型数据库里面，Schema 定义了表、每个表的字段，还有表和字段之间的关系。与之对应的，在 ES 中，Mapping 定义索引下的Type的字段处理规则，即索引如何建立、索引类型、是否保存原始索引 JSON 文档、是否压缩原始 JSON 文档、是否需要分词处理、如何进行分词处理等。

  ![](https://raw.githubusercontent.com/RobKing9/Blog_Pic/master/20221012112834.png)

- MySQL 中的增 insert、删 delete、改 update、查 search 操作等价于 ES 中的增 PUT/POST、删 Delete、改 _update、查 GET。其中的修改指定条件的更新 update 等价于 ES 中的 update_by_query，指定条件的删除等价于 ES 中的 delete_by_query。

- MySQL 中的 group by、avg、sum 等函数类似于 ES 中的 Aggregations 的部分特性。

- MySQL 中的去重 distinct 类似 ES 中的 cardinality 操作。

- MySQL 中的数据迁移等价于 ES 中的 reindex 操作。

## API操作

- 查看集群的健康状态http://127.0.0.1:9200/_cat/health?v
- 查看集群索引http://127.0.0.1:9200/_cat/indices?v
- 查看集群的节点http://127.0.0.1:9200/_cat/nodes?v
- 查看集群的其它信息http://127.0.0.1:9200/_cat/
- 查看索引的配置http://172.19.34.18:30980/post
- 查看type的数据http://172.19.34.18:30980/post/post/_search



## 参考链接

- [es和mysql的结构对比](https://juejin.cn/post/6844903938550939661)
- [web端查看ES集群信息命令](https://www.cnblogs.com/yoyowin/p/12193415.html)