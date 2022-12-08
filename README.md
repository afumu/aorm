[English](https://github.com/tangpanqing/aorm) | [简体中文](https://github.com/tangpanqing/aorm/blob/main/README_zh.md)

# Aorm
Operate Database So Easy For GoLang Developer  

Give a ⭐ if this project helped you!
## 🌟 Feature
- [x] Simply And Fast
- [x] Support MySQL DataBase
- [x] Support Null Value When Query Or Exec
- [x] Support Auto Migrate
- [X] Support SQL Builder
- [ ] Support Other DataBase, like MSSQL

## 🌟 Usage
- [Import](#import)
- [Define data struct](#define-data-struct)   
- [Connect database](#connect-database)   
- [Migrate](#migrate)
- [Basic CRUD](#basic-crud)   
  - [Insert one record](#insert-one-record)
  - [Get one record](#get-one-record)
  - [Get many record](#get-many-record)
  - [Update record](#update-record)
  - [Delete record](#delete-record)
- [Advanced Query](#advanced-query)
  - [Table](#table)
  - [Select](#select)
  - [Where](#where)
  - [Where Opts](#where-opts)
  - [Join](#join)
  - [GroupBy](#groupBy)
  - [Having](#having)
  - [OrderBy](#orderBy)
  - [Limit and Page](#limit-and-page)
  - [Lock](#lock)
  - [Increment](#increment)
  - [Decrement](#decrement)
  - [Value](#value)
  - [ValueInt64](#valueInt64)
  - [ValueFloat32](#valueFloat32)
  - [ValueFloat64](#valueFloat64)
- [Aggregation Function](#aggregation-function)
  - [Count](#count)
  - [Sum](#sum)
  - [AVG](#avg)
  - [Min](#min)
  - [Max](#max)
- [Common](#common)
  - [Query](#query)
  - [Exec](#exec)
- [Transaction](#transaction)
- [Truncate](#truncate)
- [Utils](#utils)

### Import
```go
    import (
        "database/sql"
        _ "github.com/go-sql-driver/mysql" 
        "github.com/tangpanqing/aorm"
    )
```
`database/sql` the go std package, provide sql operate database interface    
`github.com/go-sql-driver/mysql` the driver for mysql database    
`github.com/tangpanqing/aorm` wapper the sql operate, make it easy for use    
you can download them like this
```cmd
    go get -u github.com/go-sql-driver/mysql
```
```cmd
    go get -u github.com/tangpanqing/aorm
```
### Define data struct
```go
    type Person struct {
        Id         aorm.Int    `aorm:"primary;auto_increment" json:"id"`
        Name       aorm.String `aorm:"size:100;not null;comment:名字" json:"name"`
        Sex        aorm.Bool   `aorm:"index;comment:性别" json:"sex"`
        Age        aorm.Int    `aorm:"index;comment:年龄" json:"age"`
        Type       aorm.Int    `aorm:"index;comment:类型" json:"type"`
        CreateTime aorm.Time   `aorm:"comment:创建时间" json:"createTime"`
        Money      aorm.Float  `aorm:"comment:金额" json:"money"`
        Test       aorm.Float  `aorm:"type:double;comment:测试" json:"test"`
    }
```

### Connect database
```go
    //connect
    db, err := sql.Open("mysql", "root:root@tcp(localhost:3306)/database_name?charset=utf8mb4&parseTime=True&loc=Local")
    if err != nil {
        panic(err)
    }
    defer db.Close()
    
    //ping test
    err1 := db.Ping()
    if err1 != nil {
        panic(err1)
    }
```

### Migrate
by `AutoMigrate` function, the table name will be `person`, underline style string with the struct name
```go
    aorm.Use(db).Opinion("ENGINE", "InnoDB").Opinion("COMMENT", "用户表").AutoMigrate(&Person{})
```
by `Migrate` function, You can also use other table name
```go
    aorm.Use(db).Opinion("ENGINE", "InnoDB").Opinion("COMMENT", "用户表").Migrate("person_1", &Person{})
```
by `ShowCreateTable` function, You can get the create table sql
```go
    showCreate := aorm.Use(db).ShowCreateTable("person")
    fmt.Println(showCreate)
```
like this
```sql
    CREATE TABLE `person` (
        `id` int NOT NULL AUTO_INCREMENT,
        `name` varchar(100) COLLATE utf8mb4_general_ci NOT NULL COMMENT '名字',
        `sex` tinyint DEFAULT NULL COMMENT '性别',
        `age` int DEFAULT NULL COMMENT '年龄',
        `type` int DEFAULT NULL COMMENT '类型',
        `create_time` datetime DEFAULT NULL COMMENT '创建时间',
        `money` float DEFAULT NULL COMMENT '金额',
        `article_body` text COLLATE utf8mb4_general_ci COMMENT '文章内容',
        `test` double DEFAULT NULL COMMENT '测试',
        PRIMARY KEY (`id`),
        KEY `idx_person_sex` (`sex`),
        KEY `idx_person_age` (`age`),
        KEY `idx_person_type` (`type`)
    ) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='人员表'
```

### Basic CRUD

#### Insert one record
```go
    id, errInsert := aorm.Use(db).Debug(true).Insert(&Person{
        Name:       aorm.StringFrom("Alice"),
        Sex:        aorm.BoolFrom(false),
        Age:        aorm.IntFrom(18),
        Type:       aorm.IntFrom(0),
        CreateTime: aorm.TimeFrom(time.Now()),
        Money:      aorm.FloatFrom(100.15987654321),
        Test:       aorm.FloatFrom(200.15987654321987654321),
    })
    if errInsert != nil {
        fmt.Println(errInsert)
    }
    fmt.Println(id)
```
then get the sql and params like this
```sql
    INSERT INTO person (name,sex,age,type,create_time,money,test) VALUES (?,?,?,?,?,?,?)
    Alice false 18 0 2022-12-07 10:10:26.1450773 +0800 CST m=+0.031808801 100.15987654321 200.15987654321987
```

#### Get one record

```go
    var person Person
    errFind := aorm.Use(db).Debug(true).Where(&Person{Id: aorm.IntFrom(id)}).GetOne(&person)
    if errFind != nil {
        fmt.Println(errFind)
    }
    fmt.Println(person)
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE id = ? Limit ?,?
    1 0 1
```

#### Get many record
```go
    var list []Person
    errSelect := aorm.Use(db).Debug(true).Where(&Person{Type: aorm.IntFrom(0)}).GetMany(&list)
    if errSelect != nil {
        fmt.Println(errSelect)
    }
    for i := 0; i < len(list); i++ {
        fmt.Println(list[i])
    }
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE type = ?
    0
```

#### Update record

```go
    countUpdate, errUpdate := aorm.Use(db).Debug(true).Where(&Person{Id: aorm.IntFrom(id)}).Update(&Person{Name: aorm.StringFrom("Bob")})
    if errUpdate != nil {
        fmt.Println(errUpdate)
    }
    fmt.Println(countUpdate)
```
then get the sql and params like this
```sql
    UPDATE person SET name=? WHERE id = ?
    Bob 1
```

#### Delete record

```go
    countDelete, errDelete := aorm.Use(db).Debug(true).Where(&Person{Id: aorm.IntFrom(id)}).Delete()
    if errDelete != nil {
        fmt.Println(errDelete)
    }
    fmt.Println(countDelete)
```
then get the sql and params like this
```sql
    DELETE FROM person WHERE id = ?
    1
```

### Advanced Query
#### Table
by `Table` function, you can set table name easy
```go
    aorm.Use(db).Debug(true).Table("person_1").Insert(&Person{Name: aorm.StringFrom("Cherry")})
```
then get the sql and params like this
```sql
    INSERT INTO person_1 (name) VALUES (?)
    Cherry
```
#### Select
by `Select` function, you can select field name easy
```go
    var listByFiled []Person
    aorm.Use(db).Debug(true).Select("name,age").Where(&Person{Age: aorm.IntFrom(18)}).GetMany(&listByFiled)
```
then get the sql and params like this
```sql
    SELECT name,age FROM person WHERE age = ?
    18
```
#### Where
```go
    var listByWhere []Person
    
    var where1 []aorm.WhereItem
    where1 = append(where1, aorm.WhereItem{Field: "type", Opt: aorm.Eq, Val: 0})
    where1 = append(where1, aorm.WhereItem{Field: "age", Opt: aorm.In, Val: []int{18, 20}})
    where1 = append(where1, aorm.WhereItem{Field: "money", Opt: aorm.Between, Val: []float64{100.1, 200.9}})
    where1 = append(where1, aorm.WhereItem{Field: "money", Opt: aorm.Eq, Val: 100.15})
    where1 = append(where1, aorm.WhereItem{Field: "name", Opt: aorm.Like, Val: []string{"%", "li", "%"}})
    
    aorm.Use(db).Debug(true).Table("person").WhereArr(where1).GetMany(&listByWhere)
    for i := 0; i < len(listByWhere); i++ {
        fmt.Println(listByWhere[i])
    }
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE type = ? AND age IN (?,?) AND money BETWEEN (?) AND (?) AND CONCAT(money,'') = ? AND name LIKE concat('%',?,'%')
    0 18 20 100.1 200.9 100.15 li
```
#### Where Opts
`aorm.Eq` same as `=`  
`aorm.Ne` same as `!=`  
`aorm.Gt` same as `>`   
`aorm.Ge` same as `>=`   
`aorm.Lt` same as `<`  
`aorm.Le` same as `<=`  

`aorm.In` same as `IN`  
`aorm.NotIn` same as `NOT IN`  
`aorm.Like` same as `LIKE`   
`aorm.NotLike` same as `NOT LIKE`  
`aorm.Between` same as `BETWEEN`  
`aorm.NotBetween` same as `NOT BETWEEN`
#### JOIN
```go
    var list2 []ArticleVO
    
    var where2 []aorm.WhereItem
    where2 = append(where2, aorm.WhereItem{Field: "o.type", Opt: aorm.Eq, Val: 0})
    where2 = append(where2, aorm.WhereItem{Field: "p.age", Opt: aorm.In, Val: []int{18, 20}})
    
    aorm.Use(db).Debug(true).
        Table("article o").
        LeftJoin("person p", "p.id=o.person_id").
        Select("o.*").
        Select("p.name as person_name").
        WhereArr(where2).
        GetMany(&list2)
```
then get the sql and params like this
```sql
    SELECT o.*,p.name as person_name FROM article o LEFT JOIN person p ON p.id=o.person_id WHERE o.type = ? AND p.age IN (?,?)
    0 18 20
```
some other join function like this `RightJoin`, `Join`
#### GroupBy
```go
    type PersonAge struct {
        Age         aorm.Int
        AgeCount    aorm.Int
    }

    var personAge PersonAge
    
    var where []aorm.WhereItem
    where = append(where, aorm.WhereItem{Field: "type", Opt: aorm.Eq, Val: 0})

    err := aorm.Use(db).Debug(true).
        Table("person").
        Select("age").
        Select("count(age) as age_count").
        GroupBy("age").
        WhereArr(where).
        GetOne(&personAge)
    if err != nil {
        panic(err)
    }
	fmt.Println(personAge)
```
then get the sql and params like this
```sql
    SELECT age,count(age) as age_count FROM person WHERE type = ? GROUP BY age Limit ?,?
    0 0 1
```
#### Having
```go
    var listByHaving []PersonAge
    
    var where3 []aorm.WhereItem
    where3 = append(where3, aorm.WhereItem{Field: "type", Opt: aorm.Eq, Val: 0})
    
    var having []aorm.WhereItem
    having = append(having, aorm.WhereItem{Field: "age_count", Opt: aorm.Gt, Val: 4})
    
    err := aorm.Use(db).Debug(true).
        Table("person").
        Select("age").
        Select("count(age) as age_count").
        GroupBy("age").
        WhereArr(where3).
        HavingArr(having).
        GetMany(&listByHaving)
    if err != nil {
        panic(err)
    }
    fmt.Println(listByHaving)
```
then get the sql and params like this
```sql
    SELECT age,count(age) as age_count FROM person WHERE type = ? GROUP BY age Having age_count > ?
    0 4
```
#### OrderBy
```go
    var listByOrder []Person

    var where []aorm.WhereItem
    where = append(where, aorm.WhereItem{Field: "type", Opt: aorm.Eq, Val: 0})
	
    err := aorm.Use(db).Debug(true).
        Table("person").
        WhereArr(where).
        OrderBy("age", aorm.Desc).
        GetMany(&listByOrder)
    if err != nil {
        panic(err)
    }
    fmt.Println(listByOrder)
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE type = ? Order BY age DESC
    0
```
#### Limit and Page
```go
    var list3 []Person

    var where1 []aorm.WhereItem
    where1 = append(where1, aorm.WhereItem{Field: "type", Opt: aorm.Eq, Val: 0})
	
    err1 := aorm.Use(db).Debug(true).
        Table("person").
        WhereArr(where1).
        Limit(50, 10).
        GetMany(&list3)
    if err1 != nil {
        panic(err1)
    }
    fmt.Println(list3)
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE type = ? Limit ?,?
    0 50 10
```
```go
    var list4 []Person

    var where2 []aorm.WhereItem
    where2 = append(where2, aorm.WhereItem{Field: "type", Opt: aorm.Eq, Val: 0})
	
    err := aorm.Use(db).Debug(true).
        Table("person").
        WhereArr(where2).
        Page(3, 10).
        GetMany(&list4)
    if err != nil {
        panic(err)
    }
    fmt.Println(list4)
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE type = ? Limit ?,?
    0 20 10
```
#### Lock
by `Lock` function, you can lock the query
```go
    var itemByLock Person
    err := aorm.Use(db).Debug(true).LockForUpdate(true).Where(&Person{Id: aorm.IntFrom(id)}).GetOne(&itemByLock)
    if err != nil {
        panic(err)
    }
    fmt.Println(itemByLock)
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE id = ? Limit ?,?  FOR UPDATE
    2 0 1
```

#### Increment
```go
    count, err := aorm.Use(db).Debug(true).Where(&Person{Id: aorm.IntFrom(id)}).Increment("age", 1)
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
then get the sql and params like this
```sql
    UPDATE person SET age=age+? WHERE id = ?
    1 2
```

#### Decrement
```go
    count, err := aorm.Use(db).Debug(true).Where(&Person{Id: aorm.IntFrom(id)}).Decrement("age", 2)
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
then get the sql and params like this
```sql
    UPDATE person SET age=age-? WHERE id = ?
    2 2
```

#### Value
```go
    name, err := aorm.Use(db).Debug(true).Where(&Person{Id: aorm.IntFrom(id)}).Value("name")
    if err != nil {
        panic(err)
    }
    fmt.Println(name)
```
then get the sql and params like this
```sql
    SELECT name FROM person WHERE id = ? Limit ?,?
    2 0 1
```
then print the value `Alice`

#### ValueInt64
```go
    age, err := aorm.Use(db).Debug(true).Where(&Person{Id: aorm.IntFrom(id)}).ValueInt64("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(age)
```
then get the sql and params like this
```sql
    SELECT age FROM person WHERE id = ? Limit ?,?
    2 0 1
```
then print the value `17`

#### ValueFloat32
```go
    money, err := aorm.Use(db).Debug(true).Where(&Person{Id: aorm.IntFrom(id)}).ValueFloat32("money")
    if err != nil {
        panic(err)
    }
    fmt.Println(money)
```
then get the sql and params like this
```sql
    SELECT money FROM person WHERE id = ? Limit ?,?
    2 0 1
```
then print the value `100.159874`

### ValueFloat64
```go
    test, err := aorm.Use(db).Debug(true).Where(&Person{Id: aorm.IntFrom(id)}).ValueFloat64("test")
    if err != nil {
        panic(err)
    }
    fmt.Println(test)
```
then get the sql and params like this
```sql
    SELECT test FROM person WHERE id = ? Limit ?,?
    2 0 1
```
then print the value `200.15987654321987`


### Aggregation Function
#### Count
```go
    count, err := aorm.Use(db).Debug(true).Where(&Person{Age: aorm.IntFrom(18)}).Count("*")
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
then get the sql and params like this
```sql
    SELECT count(*) as c FROM person WHERE age = ?
    18
```
 
#### Sum
```go
    sum, err := aorm.Use(db).Debug(true).Where(&Person{Age: aorm.IntFrom(18)}).Sum("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(sum)
```
then get the sql and params like this
```sql
    SELECT sum(age) as c FROM person WHERE age = ?
    18
```
 
#### Avg
```go
    avg, err := aorm.Use(db).Debug(true).Where(&Person{Age: aorm.IntFrom(18)}).Avg("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(avg)
```
then get the sql and params like this
```sql
    SELECT avg(age) as c FROM person WHERE age = ?
    18
```



#### min
```go
    min, err := aorm.Use(db).Debug(true).Where(&Person{Age: aorm.IntFrom(18)}).Min("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(min)
```
then get the sql and params like this
```sql
    SELECT min(age) as c FROM person WHERE age = ?
    18
```



#### Max
```go
    max, err := aorm.Use(db).Debug(true).Where(&Person{Age: aorm.IntFrom(18)}).Max("age")
    if err != nil {
        panic(err)
    }
    fmt.Println(max)
```
then get the sql and params like this
```sql
    SELECT max(age) as c FROM person WHERE age = ?
    18
```
 
### Common
#### Query
```go
    resQuery, err := aorm.Use(db).Debug(true).Query("SELECT * FROM person WHERE id=? AND type=?", 1, 3)
    if err != nil {
        panic(err)
    }
    fmt.Println(resQuery)
```
then get the sql and params like this
```sql
    SELECT * FROM person WHERE id=? AND type=?
    1 3
```

#### Exec
```go
    resExec, err := aorm.Use(db).Debug(true).Exec("UPDATE person SET name = ? WHERE id=?", "Bob", 3)
    if err != nil {
        panic(err)
    }
    fmt.Println(resExec.RowsAffected())
```
then get the sql and params like this
```sql
    UPDATE person SET name = ? WHERE id=?
    Bob 3
```

### Transaction
```go
    tx, _ := db.Begin()
    
    id, errInsert := aorm.Use(tx).Insert(&Person{
        Name: aorm.StringFrom("Alice"),
    })
    
    if errInsert != nil {
        fmt.Println(errInsert)
        tx.Rollback()
        return
    }
    
    countUpdate, errUpdate := aorm.Use(tx).Where(&Person{
        Id: aorm.IntFrom(id),
    }).Update(&Person{
        Name: aorm.StringFrom("Bob"),
    })
    
    if errUpdate != nil {
        fmt.Println(errUpdate)
        tx.Rollback()
        return
    }
    
    fmt.Println(countUpdate)
    tx.Commit()
```
then get the sql and params like this
```sql
    INSERT INTO person (name) VALUES (?)
    Alice
                              
    UPDATE person SET name=? WHERE id = ?
    Bob 3
```

### Truncate
```go
    count, err := aorm.Use(db).Table("person").Truncate()
    if err != nil {
        panic(err)
    }
    fmt.Println(count)
```
then get the sql and params like this
```sql
    TRUNCATE TABLE person
```

### Utils
by `Ul` or `UnderLine`, you can transform camel case string to underline case    
for example, transform `personId` to `person_id`
```go
    var list2 []ArticleVO
    var where2 []aorm.WhereItem
    where2 = append(where2, aorm.WhereItem{Field: "o.type", Opt: aorm.Eq, Val: 0})
    where2 = append(where2, aorm.WhereItem{Field: "p.age", Opt: aorm.In, Val: []int{18, 20}})
	
    aorm.Use(db).Debug(true).
        Table("article o").
        LeftJoin("person p", aorm.Ul("p.id=o.personId")).
        Select("o.*").
        Select(aorm.Ul("p.name as personName")).
        WhereArr(where2).
        GetMany(&list2)
```
then get the sql and params like this
```sql
    SELECT o.*,p.name as person_name FROM article o LEFT JOIN person p ON p.id=o.person_id WHERE o.type = ? AND p.age IN (?,?)
    0 18 20
```

## Author

👤 **tangpanqing**

* Twitter: [@tangpanqing](https://twitter.com/tangpanqing)
* Github: [@tangpanqing](https://github.com/tangpanqing)

## Show Your Support
Give a ⭐ if this project helped you!