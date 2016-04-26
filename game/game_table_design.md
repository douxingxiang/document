#数据表设计

##思考1：横向建表还是纵向建表
2016-04-26 18:35:10
对于一个对象
```typescript
class User {
    id:number;
    name:string;
    gender:number;
    ...
}

```
`User`对象包含数个属性，实际业务中，属性的数量可能达到数十个，如何将这些属性存入数据表中？

**思路一：纵向建表**
将所有的属性一致对待，建表如下
```
CREATE TABLE IF NOT EXISTS user
(
    userId UNSIGNED INT NOT NULL COMMENT '用户ID',
    attrType UNSIGNED INT NOT NULL COMMENT '属性类型',
    attrValue TEXT COMMENT '属性值',
    PIMARY KEY(userId, attrType)
) ENGINE = INNODB CHARSET = UTF8;
```
类似一个key/value对的存储方式，这里有两个问题
- `attrType`使用`int`型，而不是`User`的属性字符串，可以节省存储空间，便于建索引，
    带来的问题是需要写一个常量定义的文件将对象属性编号
- `attrValue`使用`text`，因为对象的属性可以为任意类型，所以属性值既要足够大，又要能够存储所有的类型，
    带来的问题是存储空间浪费

所以纵向建表的优缺点
- 扩展性良好，对象增加属性，数据表无需修改
- 数据库存储空间浪费，以及数据从数据表到对象的类型转换成本
- 如果属性值的类型一致，或者取值长度变化不大，可以选用

**思路二：横向建表**
将每个属性分别对待，建表如下
```
CREATE TABLE IF NOT EXISTS user
(
    userId UNSIGNED INT NOT NULL COMMENT '用户ID',
    name VARCHAR(32) NOT NULL COMMENT '用户昵称',
    gender TINYINT COMMENT '用户性别',
    ...
    PRIMARY KEY(userId)
) ENGINE = INNODB CHARSET = UTF8;
```
每个属性的类型和存储大小都可以控制，没有必要多谢一份常量定义的文件，同时减少了存储空间浪费，方便对某些字段建索引

所以横向建表优缺点
- 扩展性稍差，属性增加，需要修改表结构
- 节约存储空间
- 查询和插入语句效率更高，一条语句可以覆盖一整个对象
- 方便对特定字段建索引，提高查询效率

**如何选择**
- 纵向建表针对属性数量巨大，但是属性类型和值长度变化不大的存储，可以兼顾扩展性和存储效率
- 横向建表对属性数量不是特别大，并且对查询和插入效率有较高要求的对象
