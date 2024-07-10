## mongodb 维护

### collections 创建和用户创建

1、使用admin 账户登录

```bash
$ mongosh -u root -p example --authenticationDatabase admin

Current Mongosh Log ID: 668e53b083558e3eef149f47
Connecting to:          mongodb://<credentials>@127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&authSource=admin&appName=mongosh+2.2.10
Using MongoDB:          7.0.12
Using Mongosh:          2.2.10

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

------
   The server generated these startup warnings when booting
   2024-07-10T08:56:25.240+00:00: /sys/kernel/mm/transparent_hugepage/enabled is 'always'. We suggest setting it to 'never' in this binary version
   2024-07-10T08:56:25.240+00:00: vm.max_map_count is too low
------

test>

```

2、 创建collections, 创建一个为testxxx 的collection

```bash
$  use testexample
```

3、为 `testexample`的`collections` 创建一个读写用户和只读用户

```bash
# 创建只读用户
$ db.createUser({
   user: "readonly_user",
   pwd: "password123",
   roles: [
      { role: "read", db: "testexample" }
   ]
})
# 创建读写用户
$ db.createUser({
   user: "readwrite_user",
   pwd: "password123",
   roles: [
      {role: "readWrite", db: "testexample" }
   ]
})
```

4、 验证读写用户用户

```bash
$ mongosh -u readwrite_user -p password123 --authenticationDatabase testexample

# 数据库插入数据
$ test> use testexample
switched to db testexample
$ testexample> db.myCollection.insert({name: "test"})
DeprecationWarning: Collection.insert() is deprecated. Use insertOne, insertMany, or bulkWrite.
{
  acknowledged: true,
  insertedIds: { '0': ObjectId('668e5500a8307db5f7149f48') }
}
$ testexample> db.myCollection.insert({name: "test"})
{
  acknowledged: true,
  insertedIds: { '0': ObjectId('668e5501a8307db5f7149f49') }
}
$ testexample> db.myCollection.insert({name: "test"})
{
  acknowledged: true,
  insertedIds: { '0': ObjectId('668e5502a8307db5f7149f4a') }
}
读取数据
$ testexample> db.myCollection.find()
[
  { _id: ObjectId('668e5500a8307db5f7149f48'), name: 'test' },
  { _id: ObjectId('668e5501a8307db5f7149f49'), name: 'test' },
  { _id: ObjectId('668e5502a8307db5f7149f4a'), name: 'test' }
]


```

5、验证只读用户

```bash
$ mongosh -u readonly_user -p password123 --authenticationDatabase testexample

# 数据库插入数据
$ test> use testexample
switched to db testexample
$ testexample>  db.myCollection.insert({name: "test"})
DeprecationWarning: Collection.insert() is deprecated. Use insertOne, insertMany, or bulkWrite.
Uncaught:
MongoBulkWriteError[Unauthorized]: not authorized on testexample to execute command { insert: "myCollection", documents: [ { name: "test", _id: ObjectId('668e55d127ead48a7b149f48') } ], ordered: true, lsid: { id: UUID("3f3cd052-f47a-46f3-b78e-bf3beb25a05b") }, $db: "testexample" }
Result: BulkWriteResult {
  insertedCount: 0,
  matchedCount: 0,
  modifiedCount: 0,
  deletedCount: 0,
  upsertedCount: 0,
  upsertedIds: {},
  insertedIds: { '0': ObjectId('668e55d127ead48a7b149f48') }
}
Write Errors: []

# 读取数据
$ testexample> db.myCollection.find()
[
  { _id: ObjectId('668e5500a8307db5f7149f48'), name: 'test' },
  { _id: ObjectId('668e5501a8307db5f7149f49'), name: 'test' },
  { _id: ObjectId('668e5502a8307db5f7149f4a'), name: 'test' }
]



```



### mongo修改密码

1、使用admin 账户登录mongo

```bash
$ mongosh -u root -p example --authenticationDatabase admin
```

2、修改`readwrite_user` 的用户密码 `password123` -> `password1234`

使用如下命令进行update

```bash
$ use testexample
$ db.updateUser(
   "readwrite_user",
   {
     pwd: "password1234"
   }
 )

```

执行详情如下

```bash
$ test> use testexample
switched to db testexample
$ testexample> db.updateUser(
...    "readwrite_user",
...    {
...      pwd: "password1234"
...    }
...  )
{ ok: 1 }

```

2、验证

使用旧密码登录

```bash
$  mongosh -u readwrite_user -p password123 --authenticationDatabase testexample
Current Mongosh Log ID: 668e57916bc17d5a82149f47
Connecting to:          mongodb://<credentials>@127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&authSource=testexample&appName=mongosh+2.2.10
MongoServerError: Authentication failed.

```

使用新密码登录

```bash
$ mongosh -u readwrite_user -p password1234 --authenticationDatabase testexample
Current Mongosh Log ID: 668e57951a5fa8a2c5149f47
Connecting to:          mongodb://<credentials>@127.0.0.1:27017/?directConnection=true&serverSelectionTimeoutMS=2000&authSource=testexample&appName=mongosh+2.2.10
Using MongoDB:          7.0.12
Using Mongosh:          2.2.10

For mongosh info see: https://docs.mongodb.com/mongodb-shell/

test>

```

