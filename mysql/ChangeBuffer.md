# Change Bufferr
是InnoDB存储引擎的一个机制,用于暂存非唯一二级索引的插入和更新操作的变更

写多读少的场景效果明显

高并发写入场景中,减少磁盘I/Ojing'z竞争,提高系统吞吐量

## 大小
系统变量`innodb_change_buffer_max_size`
- 默认值25%的缓冲池大小,最大50%
- 过大可能导致内存不足

## 机制
`insert`,`update`,`delete`操作二级索引的时候,不会立即在磁盘中修改,而是暂存到`change buffer`,在查询的时候才会更新到磁盘,或者后台定期合并到磁盘中