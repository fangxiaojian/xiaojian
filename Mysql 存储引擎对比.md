







| Aeature                                              | MyISAM | Memory | InnoDB |
| ---------------------------------------------------- | ------ | ------ | ------ |
| storage limits 存储限制                              | NO     | Yes    | 64TB   |
| transactions(commit,rollback,etc) 事务(提交、回滚等) |        | true   | true   |
| locking granularity 锁的粒度                         | table  | table  | row    |
| MVCC/Snapshot read MVCC /快照读取                    |        |        | true   |
| geospatial support 地理空间支持                      | true   |        |        |
| B-Tree indexes b -树索引                             | true   | true   | true   |
| hash indexes hash索引                                |        | true   | true   |
| full text search index 全文检索索引                  | true   |        |        |
| clustered index 聚集索引                             |        |        | true   |
| data caches 数据缓存                                 |        | true   | true   |
| index caches 索引缓存                                | true   | true   | true   |
| compressed data 压缩数据                             | true   |        |        |
| encrypted data(via function) 加密的数据(通过函数)    | true   | true   | true   |
| storage cost(space used) 使用存储成本(空间)          | low    | N/A    | high   |
| memory cost 内存成本                                 | low    | medium | high   |
| bulk insert speed 批量插入的速度                     | high   | high   | low    |
| cluster database support 集群数据库支持              |        |        |        |
| replication support 副本支持                         | true   | true   | true   |
| foreign key support  外键                            |        |        | true   |
| backup/Point-in-time recovery                        | true   | true   | true   |
| query cache support                                  | true   | true   | true   |
| update statistics for data dictionaty                | true   | true   | true   |

