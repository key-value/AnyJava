1. 保存执行语句
2. 保存修改数据
3. 复合型 以语句为主，如果含有语句 则使用数据


# redolog
redolog里记录的内容

原来并不像我想象的那样写个sql语句进去，有点太夸张了

而是记录下数据块的改动。

session_id + date + file_id + block_id + 修改数据

Update操作较多数据时，会产生大量日志

首先记录的是将数据读进undo和db buffer cache直道读完了也写完了

然后的commit又去记录datafile的变化。

Log写完就认为完成了，不管当时是否已写进datafile.

所以不会出现一个操作写满全部日志却不能switch的问题。

日志和写db没有直接的联系。