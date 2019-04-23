# DB Query Builder

[![Build Status](https://travis-ci.org/wazsmwazsm/DB.svg?branch=master)](https://travis-ci.org/wazsmwazsm/DB)

一个简单好用的查询构造器，基于 PDO，支持 mysql、postgresql、sqlite，支持常驻内存模式的断线重连。

## 安装
```bash
composer require wazsmwazsm/db
```
## 依赖

- php pdo 扩展
- php pdo_mysql 扩展（可选）
- php pdo_pgsql 扩展（可选）
- php pdo_sqlite 扩展（可选）

# 如何开始
## 初始化配置

查询构造器支持多个数据库连接，初始化后每个连接以单例的形式存在。

```php
use DB/DB;

// 配置信息
$db_conf = [
	// mysql 配置样例
    'con1' => [ 
        'driver'      => 'mysql',
        'host'        => 'localhost',
        'port'        => '3306',
        'user'        => 'username',
        'password'    => 'password',
        'dbname'      => 'database',
        'charset'     => 'utf8',
        'prefix'      => '',
        'timezone'    => '+8:00',
        'collection'  => 'utf8_general_ci',
        // 'strict'      => false,
        // 'unix_socket' => '/var/run/mysqld/mysqld.sock',
        // 'options'     => [],
    ],
    
	// postgresql 配置样例
    'con2' => [
        'driver'           => 'pgsql',
        'host'             => 'localhost',
        'port'             => '5432',
        'user'             => 'username',
        'password'         => 'password',
        'dbname'           => 'database',
        'charset'          => 'utf8',
        'timezone'         => '+8:00',
        'prefix'           => '',
        // 'schema'           => '',
        // 'application_name' => '',
        // 'sslmode'          => 'disable',
        // 'options'          => [],
    ],
	// sqlite 配置样例
    'con3' => [ 
        'driver'  => 'sqlite',
        'dbname'  => 'database.db',
        'prefix'  => '',
        // 'options' => [],
    ],
    // 其他连接
    'con3' => [ 
        // ...
    ]
];

// 初始化连接
DB::init($db_conf);

```
## 获取连接

```php
use DB\DB;

$con = DB::connection('con1');

$result = $con->table('test_table')->get();
```

## 模型
这里的模型并不是 ORM 模型，它只是为了方便操作而对查询构造器 \\DB\\DB 的一个封装。

```php
use DB\Model;

class MyModel extends Model
{
    // 指定数据库连接
    protected $connection;
	// 指定数据表
    protected $table;
    // 自定义方法
    public function getData()
    {
    	// 使用查询构造器的方法
        return $this->get();
    }
}

$m = new MyModel;

$result = $m->getData();

```

# 查询数据

## 获取结果

获取结果的方法有：get，row，getList，count，sum，avg，max，min。参数等信息参考 [ConnectorInterface](https://github.com/wazsmwazsm/DB/blob/master/src/DB/Drivers/ConnectorInterface.php)。

### 获取结果集

table 方法用来指定查询的数据表。

get 方法用来获取结果集，返回值是一个查询结果的数组。


```php
// SELECT * FROM test;
$data = DB::connection('con1')->table('test')->get();
```

### 获取一行数据

select 方法用来指定要取的列，select('\*\') 代表取所有列 (此时 select 可以省略不写)。

row 方法用来获取一行数据。

```php
// SELECT id, name, age, score FROM test WHERE id = 10;
$row = DB::connection('con1')
    ->table('test')
    ->select('id', 'name', 'age', 'score')
    ->where('id', 10)
    ->row();
```

### 获取一列

有时候我们需要获得某一列的数据，这时 getList 方法就派上用场了，getList 方法直接返回该列值的数组。

```php
// 获取 name 字段的结果集
// SELECT name FROM test;
$data = DB::connection('con1')->table('test')->getList('name');
```

### 聚合函数

查询构造器为获取聚合函数的结果做了封装，你可以使用 count，sum 等方法获取聚合数据。
```php
// 获取查询结果的条数
// SELECT COUNT(*) FROM test;
$data = DB::connection('con1')->table('test')->count(*);
// 获取 score 的总和
// SELECT SUM(score) FROM test;
$data = DB::connection('con1')->table('test')->sum('score');
```

## where 子句

构建 where 子句的方法有 where、orWhere。

where 、orWhere 方法的参数支持三种模式：

- where(['name' => 'mike', 'age' => 18])
- where('age', 24)
- where('score', '>=', 60)、where('tag', 'like', '%php%')

一些常用的例子：
```php
// SELECT * FROM test WHERE name = 'mike' AND age = 18;
$data = DB::connection('con1')
    ->table('test')
    ->where([
        'name' => 'mike', 
        'age' => 18
    ])
    ->get();

// SELECT * FROM test WHERE name = 'jack' AND age >= 18;
$data = DB::connection('con1')
    ->table('test')
    ->where('name', 'jack')
    ->where('age', '>=',  18)
    ->get();
// SELECT * FROM test WHERE name = 'jack' OR name = 'mike';
$data = DB::connection('con1')
    ->table('test')
    ->where('name', 'jack')
    ->orWhere('name', 'mike')
    ->get();
```

对于 null 的条件查询，查询构造器提供了 whereNull、whereNotNull 等方法：

```php
// SELECT * FROM test WHERE name IS NOT null;
$data = DB::connection('con1')
    ->table('test')
    ->whereNotNull('name')
    ->get();
```

> 注：where is null 子句的构造提供 whereNull、orWhereNull、whereNotNull、orWhereNotNull 四个方法。


where between 子句：

```php
// SELECT * FROM test WHERE age BETWEEN 18 AND 30;
$data = DB::connection('con1')
    ->table('test')
    ->whereBetween('age', 18, 30)
    ->get();
```

> 注：where between 子句的构造提供 whereBetween、orWhereBetween 个方法。

where in 子句：
```php
// SELECT * FROM test WHERE age IN (18, 19, 20);
$data = DB::connection('con1')
    ->table('test')
    ->whereIn('age', [18, 19, 20])
    ->get();
```

> 注：where in 子句的构造提供 whereIn、orWhereIn、whereNotIn、orWhereNotIn 四个方法。

## 复杂 where 子句

where exists 子句：
```php
// SELECT * FROM user WHERE EXISTS ( SELECT * FROM user_group WHERE id = 3 ) AND g_id = 3;
$data = DB::connection('con1')
    ->table('user')
    ->whereExists(function($query) {
        $query->table('user_group')->where('id', 3);
    })
    ->where('g_id', 3)
    ->get();
```

> 注：where exists 子句的构造提供 whereExists、orWhereExists、whereNotExists、orWhereNotExists 四个方法。

where in 子查询：
```php
// SELECT * FROM user WHERE g_id IN (SELECT id FROM user _group);
$data = DB::connection('con1')
    ->table('user')
    ->whereInSub('g_id', function($query) {
        $query->table('user_group')->select('id');
    })
    ->get();
```

> 注：where in 子查询的构造提供 whereInSub、orWhereInSub、whereNotInSub、orWhereNotInSub 四个方法。

对于复杂的 where 逻辑，我们经常使用圆括号来确定优先级。查询构造器提供了 whereBrackets() 和 orWwhereBrackets() 方法来添加圆括号。

```php
// SELECT * FROM user WHERE (id < 50 OR username IS NOT NULL) AND sort_num = 20;
$data = DB::connection('con1')
    ->table('user')
    ->whereBrackets(function($query) {
        $query->where('id', '<', 50)
                ->orWhereNotNull('username');
    })
    ->where('sort_num', 20)
    ->get();
        
// SELECT * FROM user WHERE sort_num = 20 OR (id < 10 AND id > 5);
$data = DB::connection('con1')
    ->table('user')
    ->where('sort_num', 20)
    ->orWhereBrackets(function($query) {
        $query->where('id', '<', 10)
                ->where('id', '>', 5);
    })
    ->get();
```

## 子查询

查询构造器提供了 fromSub 方法进行子查询：
```php
// SELECT id, username, email FROM ( SELECT * FROM user WHERE id < 20 ) AS tb;
$data = DB::connection('con1')
    ->select('id', 'username', 'email')
    ->fromSub(function($query) {
        $query->table('user')->where('id', '<', '20');
    })
    ->get();
```


## 分组

查询构造器提供了 group 方法对字段进行分组，having (orHaving) 方法进行分组条件筛选。

having 方法和 where 方法一样支持三种参数模式，这里就不再赘述。

```php
// SELECT sort_num, COUNT(sort_num) FROM user GROUP BY sort_num;
$data = DB::connection('con1')
    ->table('user')
    ->select('sort_num', 'COUNT(sort_num)')
    ->groupBy('sort_num')
    ->get();
        
// SELECT sort_num, COUNT(sort_num) FROM user GROUP BY sort_num HAVING COUNT(sort_num) < 20;
$data = DB::connection('con1')
    ->table('user')
    ->select('sort_num', 'COUNT(sort_num)')
    ->groupBy('sort_num')
    ->having('COUNT(sort_num)', '<', 20)
    ->get();
```

查询构造器还提供了一个输入原生字符串的 havingRaw 方法，可以处理一些复杂的情况：
```php
// SELECT sort_num, COUNT(sort_num) FROM user GROUP BY sort_num HAVING COUNT(sort_num) < 20;
$data = DB::connection('con1')
    ->table('user')
    ->select('sort_num', 'COUNT(sort_num)')
    ->groupBy('sort_num')
    ->havingRaw('COUNT(sort_num) < 20')
    ->get();
```

## 排序

查询构造器提供了 orderBy 方法对查询结果排序：
```php
// SELECT * FROM user ORDER BY sort_num DESC, id ASC;
$data = DB::connection('con1')
    ->table('user')
    ->orderBy('sort_num', 'DESC')
    ->orderBy('id', 'ASC')
    ->get();
```

## limit

查询构造器提供了 limit 方法来取一定范围的数据：
```php
// SELECT * FROM user LIMIT 10 OFFSET 3
$data = DB::connection('con1')
    ->table('user')
    ->limit(3, 10)
    ->get();
```

## 分页

查询构造器提供了 paginate 方法实现了数据的分页读取，底层基于 limit 方法。

假设有 35 条数据，每页有 10 条数据，我们取第 2 页的数据，做如下查询：

```php
$data = DB::connection('con1')
    ->table('user')
    ->paginate(10, 2);  // 每页 10 条数据，当前第 2 页
```

paginate 方法返回结果：
```php
[
	'total'        => 35;
    'per_page'     => 10;
    'current_page' => 2;
    'next_page'    => 3;
    'prev_page'    => 1;
    'first_page'   => 1;
    'last_page'    => 4;
    'data'         => $yourdata;
]
```


## 关联

查询构造器提供了 join、leftJoin、rightJoin 三个方法来实现关联查询的构造。

> 注：
> 使用关联查询后，防止字段重名字段需要带表前缀。
> sqlite 不支持 RIGHT JOIN 语言，rightJoin 方法在 sqlite 中不生效。

一些例子：
```php
// SELECT * FROM user INNER JOIN user_group ON user.g_id = user_group.id;
$data = DB::connection('con1')
    ->table('user')
    ->join('user_group', 'user.g_id', 'user_group.id')
    ->get();
        
        
// SELECT user.username, user_group.groupname FROM user LEFT JOIN user_group ON user.g_id = user_group.id;
$data = DB::connection('con1')
    ->table('user')
    ->select('user.username', 'user_group.groupname')
    ->leftJoin('user_group', 'user.g_id', 'user_group.id')
    ->get();
```

## 构造复杂的语句

查询构造器提供了灵活的接口，有了上面的基础，我们现在构造一些复杂的查询：

```php
// SELECT user.username, user_group.groupname FROM user 
// LEFT JOIN user_group ON user.g_id = user_group.id 
// WHERE username = 'Jackie aa' 
// OR ( NOT EXISTS ( SELECT * FROM user WHERE username = 'Jackie aa' ) AND username = 'Jackie Conroy' );
$data = DB::connection('con1')
    ->table('user')
    ->select('user.username', 'user_group.groupname')
    ->leftJoin('user_group', 'user.g_id', 'user_group.id')
    ->where('user.username', 'Jackie aa')
    ->orWhereBrackets(function($query) {
        $query->whereNotExists(function($query) {
            $query->table('user')->where('username', 'Jackie aa');
        })->where('user.username', 'Jackie Conroy');
    })
    ->get();

// SELECT user.sort_num, COUNT(*) FROM user 
// INNER JOIN user_group ON user.g_id = user_group.id 
// WHERE user.activated <> 0 
// GROUP BY user.sort_num 
// HAVING user.sort_num = 20 OR user.sort_num = 50 ORDER BY user.sort_num DESC;
$data = DB::connection('con1')
    ->table('user')
    ->select('user.sort_num', 'COUNT(*)')
    ->join('user_group', 'user.g_id', 'user_group.id')
    ->where('user.activated', '<>', 0)
    ->groupBy('user.sort_num')
    ->having('user.sort_num', '50')
    ->orHaving('user.sort_num', '20')
    ->orderBy('user.sort_num', 'DESC')
    ->get();

// SELECT user.username, user_group.groupname, company.companyname FROM company 
// LEFT JOIN user_group ON user_group.c_id = company.id 
// LEFT JOIN user ON user.g_id = user_group.id 
// ORDER BY user.sort_num ASC, user.id DESC LIMIT 25 offset 10;
$data = DB::connection('con1')
    ->table('user')
    ->select('user.username', 'user_group.groupname', 'company.companyname')
    ->leftJoin('user_group', 'user_group.c_id', 'company.id')
    ->leftJoin('user', 'user.g_id', 'user_group.id')
    ->orderBy('user.sort_num', 'ASC')
    ->orderBy('user.id', 'DESC')
    ->limit(10, 25)
    ->get();
        
// SELECT * FROM user 
// WHERE username = 'Jackie aa' 
// OR ( NOT EXISTS ( SELECT * FROM user WHERE username = 'Jackie aa' ) AND (username = 'Jackie Conroy' OR username = 'Jammie Haag') ) 
// AND g_id IN ( SELECT id FROM user_group) ORDER BY id DESC LIMIT 1 OFFSET 0 ;
$data = DB::connection('con1')
    ->table('user')
    ->where('username', 'Jackie aa')
    ->orWhereBrackets(function($query) {
        $query->whereNotExists(function($query) {
            $query->table('user')->where('username', 'Jackie aa');
        })->WhereBrackets(function($query) {
            $query->where('username', 'Jackie Conroy')
                    ->orWhere('username', 'Jammie Haag');
        });
    })
    ->whereInSub('g_id', function($query) {
        $query->table('user_group')->select('id');
    })
    ->orderBy('id', 'DESC')
    ->limit(0, 1)
    ->get();
```

## DML 语句

除了基本的查询，查询构造器还提供了基础的 DML 操作，如插入、跟新、删除。

### 插入

insert 方法：
```php
// INSERT INTO user (id, name, age) VALUES (5, 'jack', 18)
$insert_data = [
  'id'   => 5,
  'name' => 'jack',
  'age'  => 18,
];
// 默认返回受影响行数
$effect_row = DB::connection('con1')->table('user')->insert($insert_data);
```

获取插入 ID，insertGetLastId 方法：

```php
// INSERT INTO user (id, name, age) VALUES (5, 'jack', 18)
$insert_data = [
  'id'   => 5,
  'name' => 'jack',
  'age'  => 18,
];
// 默认插入行的 id
$effect_row = DB::connection('con1')->table('user')->insertGetLastId($insert_data);
```

### 更新

update 方法。

> 注：为了安全考虑，update 方法必须要和 where 子句一起使用，否则会抛出 InvalidArgumentException 异常。

```php
//  UPDATE user SET name = 'mike', age = 23 WHERE name = 'jack';
$update_data = [
  'name' => 'mike',
  'age'  => 23,
];
// 默认返回受影响行数
$effect_row = DB::connection('con1')
    ->table('user')
    ->where('name', 'jack')
    ->update($update_data);
```

## 删除

delete 方法。

> 注：为了安全考虑，delete 方法必须要和 where 子句一起使用，否则会抛出 InvalidArgumentException 异常。

```php
// DELETE FROM user WHERE id = 1;
$effect_row = DB::connection('con1')
    ->table('user')
    ->where('id', 1)
    ->delete();
```

## 事务

查询构造器提供了 beginTrans、commitTrans、rollBackTrans 三个方法来支持事务：

事务回滚：
```php
// 开始事务
DB::connection('con1')->beginTrans();
// DML 操作
DB::connection('con1')
    ->table('user')
    ->where('id', 1)
    ->delete(); 
// 回滚事务
DB::connection('con1')->rollBackTrans();
```

提交事务：

```php
// 开始事务
DB::connection('con1')->beginTrans();
// DML 操作
DB::connection('con1')
    ->table('user')
    ->where('id', 1)
    ->delete(); 
// 提交事务
DB::connection('con1')->commitTrans();
```

## License

The DB is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT).
