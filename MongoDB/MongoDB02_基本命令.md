# 数据库
## 查看所有数据库
```
show dbs
```

## 查看当前数据库
```
db
```

## 创建数据库（切换数据库）
```
use DATABASE_NAME
```
如果数据库不存在，则创建数据库，否则切换到指定数据库。
MongoDB中默认的数据库为 test，如果你没有创建新的数据库，集合将存放在 test 数据库中。

## 删除数据库
```
db.dropDatabase()
```
删除当前数据库，默认为 test，可以使用 db 命令查看当前数据库名。

## 实例
```
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB

> use runoob
switched to db runoob
> db
runoob

> db.runoob.insert({"name":"菜鸟教程"})
WriteResult({ "nInserted" : 1 })
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
runoob  0.000GB

> db.dropDatabase()
{ "dropped" : "runoob", "ok" : 1 }
```

# 集合（表）
## 查看集合
```
show collections
或
show tables
```

## 创建集合
```
db.createCollection(COLLECTION_NAME, [OPTIONS])
```
**参数说明：**
name: 要创建的集合名称
options: 可选参数, 指定有关内存大小及索引的选项

**options 可以是如下参数：**
|字段			|类型	|描述	|
|--	|--	|--	|
|capped			|布尔	|（可选）如果为 true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。当该值为 true 时，必须指定 size 参数。|
|autoIndexId	|布尔	|（可选）如为 true，自动在 _id 字段创建索引。默认为 false。|
|size			|数值	|（可选）为固定集合指定一个最大值（以字节计）。如果 capped 为 true，也需要指定该字段。|
|max			|数值	|（可选）指定固定集合中包含文档的最大数量。|

## 删除集合
```
db.COLLECTION_NAME.drop()
```

## 实例
```
> db.createCollection("runoob")
{ "ok" : 1 }
>

> db.createCollection("runoob1", { capped : true, autoIndexId : true, size : 6142800, max : 10000 } )
{ "ok" : 1 }

>show collections
runoob
runoob1

>db.runoob1.drop()
true

>show collections
runoob
```

# 文档
## 查看文档
**find() 方法以非结构化的方式来显示所有文档。**
```
db.collection.find(query, projection)
```
- query ：可选，使用查询操作符指定查询条件
- projection ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

## 插入文档
MongoDB 使用 insert() 或 save() 方法向集合中插入文档，语法如下：
```
db.COLLECTION_NAME.insert(document)
```

## 更新文档
MongoDB 使用 update() 和 save() 方法来更新集合中的文档。

update() 方法用于更新已存在的文档。语法格式如下：
```
db.COLLECTION_NAME.update(
   <query>,
   <update>,
   {
     upsert: <boolean>,
     multi: <boolean>,
     writeConcern: <document>
   }
)
```
- query : update的查询条件，类似sql update查询内where后面的。
- update : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- upsert : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- multi : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- writeConcern :可选，抛出异常的级别。

save() 方法通过传入的文档来替换已有文档。语法格式如下：
```
db.COLLECTION_NAME.save(
   <document>,
   {
     writeConcern: <document>
   }
)
```
- document : 文档数据。
- writeConcern : 可选，抛出异常的级别。

## 删除文档
remove() 函数是用来移除集合中的数据。语法格式如下：
```
db.COLLECTION_NAME.remove(
   <query>,
   <justOne>
)
```
- query :（可选）删除的文档的条件。
- justOne : （可选）如果设为 true 或 1，则只删除一个文档，如果不设置该参数，或使用默认值 false，则删除所有匹配条件的文档。
- writeConcern :（可选）抛出异常的级别。

可以使用以下命令清空集合 "col" 的数据：
```
db.col.remove({})
```

## 实例
```
以下实例通过 by 和 title 键来查询 菜鸟教程 中 MongoDB 教程 的数据：
> db.col.find({"by":"菜鸟教程", "title":"MongoDB 教程"}).pretty()

以下实例可以存储在 MongoDB 的 runoob 数据库 的 col 集合中：
>db.col.insert({title: 'MongoDB 教程', 
    description: 'MongoDB 是一个 Nosql 数据库',
    by: '菜鸟教程',
    url: 'http://www.runoob.com',
    tags: ['mongodb', 'database', 'NoSQL'],
    likes: 100
})

通过 update() 方法来更新标题(title)：
>db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}})

删除 title 为 'MongoDB 教程' 的文档：
>db.col.remove({'title':'MongoDB 教程'})
```

# 高级查询
## 条件操作符
MongoDB中条件操作符有：
- (>) 大于 - $gt
- (<) 小于 - $lt
- (>=) 大于等于 - $gte
- (<= ) 小于等于 - $lte

如果你想获取 "col" 集合中 "likes" 大于 100 的数据，你可以使用以下命令：
```
db.col.find({likes : {$gt : 100}})
```

如果你想获取"col"集合中 "likes" 大于100，小于 200 的数据，你可以使用以下命令：
```
db.col.find({likes : {$lt :200, $gt : 100}})
```

## $type 操作符
$type操作符是基于BSON类型来检索集合中匹配的数据类型，并返回结果。
```
|类型					|数字	|备注			|
|Double					|1		|				|
|String					|2		|				|
|Object					|3		|				|
|Array					|4		|				|
|Binary data			|5		|				|
|Undefined				|6		|已废弃。		|
|Object id				|7		|				|
|Boolean				|8		|				|
|Date					|9		|				|
|Null					|10		|				|
|Regular Expression		|11		|				|
|JavaScript				|13		|				|
|Symbol					|14		|				|
|JavaScript (with scope)|15		|				|
|32-bit integer			|16		|				|
|Timestamp				|17		|				|
|64-bit integer			|18		|				|
|Min key				|255	|Query with -1.	|
|Max key				|127	|				|
```

如果想获取 "col" 集合中 title 为 String 的数据，你可以使用以下命令：
```
db.col.find({"title" : {$type : 2}})
或
db.col.find({"title" : {$type : 'string'}})
```

## 总数 count() 查询记录总数
```
> db.users.count()

> db.users.find().count()
```

当使用limit()方法限制返回的记录数时，默认情况下count()方法仍然返回全部记录条数。 例如，下面的示例中返回的不是5，而是user表中所有的记录数量：
```
> db.users.find().skip(10).limit(5).count()
```

如果希望返回限制之后的记录数量，要使用count(true)或者count(非0)：
```
> db.users.find().skip(10).limit(5).count(true)
```

## 分页 limit() 与 skip() 方法
limit()方法接受一个数字参数，该参数指定从MongoDB中读取的记录条数。
```
db.COLLECTION_NAME.find().limit(NUMBER)
```

以下实例为显示查询文档中的两条记录：
```
> db.col.find({},{"title":1,_id:0}).limit(2)
```

我们除了可以使用limit()方法来读取指定数量的数据外，还可以使用skip()方法来跳过指定数量的数据，skip方法同样接受一个数字参数作为跳过的记录条数。
```
db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)
```

以下实例只会显示第二条文档数据：
```
>db.col.find({},{"title":1,_id:0}).limit(1).skip(1)
```

## 排序 sort() 方法
在 MongoDB 中使用 sort() 方法对数据进行排序。
sort() 方法可以通过参数指定排序的字段。
并使用 1 和 -1 来指定排序的方式，其中 1 为升序排列，而 -1 是用于降序排列。
```
db.COLLECTION_NAME.find().sort({KEY:1})
```

以下实例演示了 col 集合中的数据按字段 likes 的降序排列：
```
>db.col.find({},{"title":1,_id:0}).sort({"likes":-1})
```

## 聚合 aggregate() 方法
```
>db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```

一些聚合的表达式：
```
|表达式		|描述										|实例																					|
|$sum		|计算总和。									|db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}])	|
|$avg		|计算平均值									|db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}])	|
|$min		|获取集合中所有文档对应值得最小值。				|db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}])	|
|$max		|获取集合中所有文档对应值得最大值。				|db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}])	|
|$push		|在结果文档中插入值到一个数组中。				|db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}])				|
|$addToSet	|在结果文档中插入值到一个数组中，但不创建副本。	|db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}])		|
|$first		|根据资源文档的排序获取第一个文档数据。			|db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}])		|
|$last		|根据资源文档的排序获取最后一个文档数据			|db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}])		|

```

已知集合中的数据如下：
```
{
   _id: ObjectId(7df78ad8902c)
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by_user: 'runoob.com',
   url: 'http://www.runoob.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
},
{
   _id: ObjectId(7df78ad8902d)
   title: 'NoSQL Overview', 
   description: 'No sql database is very fast',
   by_user: 'runoob.com',
   url: 'http://www.runoob.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 10
},
{
   _id: ObjectId(7df78ad8902e)
   title: 'Neo4j Overview', 
   description: 'Neo4j is no sql database',
   by_user: 'Neo4j',
   url: 'http://www.neo4j.com',
   tags: ['neo4j', 'database', 'NoSQL'],
   likes: 750
},
```

计算每个作者所写的文章数，使用aggregate()计算结果如下：
```
> db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{
   "result" : [
      {
         "_id" : "runoob.com",
         "num_tutorial" : 2
      },
      {
         "_id" : "Neo4j",
         "num_tutorial" : 1
      }
   ],
   "ok" : 1
}
>
```

## 管道操作符
管道在Unix和Linux中一般用于将当前命令的输出结果作为下一个命令的参数。

MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果传递给下一个管道处理。管道操作是可以重复的。

表达式：处理输入文档并输出。表达式是无状态的，只能用于计算当前聚合管道的文档，不能处理其它的文档。

这里我们介绍一下聚合框架中常用的几个操作：
- $project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
- $match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
- $limit：用来限制MongoDB聚合管道返回的文档数。
- $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
- $unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
- $group：将集合中的文档分组，可用于统计结果。
- $sort：将输入文档排序后输出。
- $geoNear：输出接近某一地理位置的有序文档。

级联操作示例：
```	
db.articles.aggregate( [
                        { $match : { score : { $gt : 70, $lte : 90 } } },
                        { $group: { _id: null, count: { $sum: 1 } } }
                       ] );
```

