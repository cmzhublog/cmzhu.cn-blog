## mongo 删除重复数据

### 背景

1、 mongo 再创建唯一索引时候，发现报错，说数据库中存在重复的值

```bash
$ db.getCollection("cmzhu-test").createIndex({projectId: 1, method: 1, matchUrl: 1}, {background:true, name:"udx_projectId_method_matchUrl", unique:true});
MongoServerError[DuplicateKey]: Index build failed: 29fee8a4-2ee1-40b1-96f1-99772b27b53b: Collection atp-test.cmzhu-test ( eff5cb37-b7b7-49e0-8019-b1858daa120a ) :: caused by :: E11000 duplicate key error collection: atp-test.atp_api_perspective index: cmzhu-test dup key: { projectId: "1814199950649589760", method: "POST", matchUrl: "/outaddresspaging" }
```

发现是数据库中存在重复数据

### 解决思路

删除重复的数据即可， 可参照以下的脚本, 需要删除以`projectId`、 `method` 、 `matchUrl` 存在的重复数据， 具体可参照下面命令；

```javascript
const duplicates = db.atp_api_perspective.aggregate([
    {
        $group: {
            _id: {
                projectId: "$projectId",
                method: "$method",
                matchUrl: "$matchUrl"
            },
            ids: { $push: "$_id" }, // 收集所有重复文档的 _id
            count: { $sum: 1 }
        }
    },
    {
        $match: {
            count: { $gt: 1 } // 只选择出现次数大于 1 的组合
        }
    }
]).map(doc => doc.ids.slice(1)); // 只保留每组中的多余 _id

// 第二步：打印待删除的内容并删除重复文档
duplicates.forEach(idArray => {
    // 查找待删除的文档内容
    const documentsToDelete = db.atp_api_perspective.find({ _id: { $in: idArray } }).toArray();
    
    // 打印待删除的文档
    print("Documents to delete:");
    printjson(documentsToDelete);

    // 删除重复文档
    db.atp_api_perspective.deleteMany({ _id: { $in: idArray } });
});
```

