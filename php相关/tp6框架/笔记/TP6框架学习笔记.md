## TP6框架学习笔记

### 绪论

> #### lesson 2 开发规范和目录结构

魔术方法：

无需调用 满足条件自启动的方法 例如构造方法，命名时前缀 __

app目录：

应用目录 开发在该目录下完成

当有多个app_name时 则为多应用模式

保证对外（网站页面）只有public文件夹可打开



> #### lesson 3  调试与配置

.env文件中的 APP_DEBUG 来控制调试模式的开启与否

部署后会被忽略 仅适用于本地测试开发

本地测试时 .env优先级高于config目录下的配置



> #### lesson 4 URL访问模式

https://ServerName like 127.0.0.1:8000 or tp6/public/index.php

例如 https://localhost:8000/index.php/test/hello/value/world

形如 服务器地址/index.php（可省略，需要设置URL重启）/控制器/操作/参数/值

兼容模式（如果不兼容为啥用tp6框架呢）



> #### lesson 5 控制器定义

controller 

系统默认控制器文件目录 config/route.php 路由设置

可以通过该文件避免文件冲突

渲染输出

① return  ② json()   ③ halt()中断函数



> #### lesson 6 多种控制器

一、基础控制器

继承表达 extends 关键字

二、空控制器

单应用模式 Error控制器类来提醒错误

三、多级控制器

在controller目录下新建其他目录 url访问时控制器输入格式为 目录.控制器类





### 数据库操作

> #### lesson 7 连接数据库

数据抽象层-PDO模式

.env 执行本地数据库配置 config/database 执行服务器数据库配置

connections 用于配置多个数据库的信息



> #### lesson 8 数据库查询

**① 单条查询 (链式查询)**

```php
Db::connect('databasename')->table('tablename')->where('field','op')->find();
```

如果find()未查询到数据则返回空，使用findOrFail则返回异常，findOrEmpty返回空

**② 多条查询**

变数组 ->toArray

将table() 方法改为 name() 可以不用填写在.env/config文件里填写的前缀

**③ 其他查询**

value(’field‘) 方法 查询指定字段的值，没有返回null

column(’field‘,'key') 方法 查询指定列的值，没有返回空数组

**④ 优化查询**

数据量巨大的情况下（千万数据集）

**chunk() 方法**分批处理 节约内存开销

```php
Db::Connect('databasename')->name('tablename')->chunk(patchsize,function({foreach...}));
```

**游标查询功能**<-php生成器特性（>php5.5） 

读取超大型文件 每次读一行 依次读取 将一行放进内存处理

```php
Db::connect('databasename')->name('tablename')->cursor();
```



> #### lesson 9 数据库链式查询

通过 -> 符号连续调用

Query对象

find() 返回Array select()返回Collection => 二者为结果查询方法

多次查询同一个实例可以保存 否则反复生成实例浪费空间

当同一个对象实例第二次查询后会保留第一次查询的值，使用removeOption方法清理上一次查询保留的值

```php
$userQuery = Db::name('user');
$user = $userQuery->where('id',27)->select();
$userSelect = $userQuery->removeOption('where')->select();
```



> #### lesson 10 数据库数据新增

**① 单数据新增**

insert() 方法 如果添加成功 return 会返回一个1；如果添加了一个不存在的字段，会抛出异常；如果要忽略该异常，使用strict(false)；

mysql数据库支持replace()写入，后者对相同主键进行修改；

insertGetId() 返回插入时的id

**② 批量新增**

insertAll() 用法同① 也支持replace() 方法

**③ save方法**

通用方法 自行判断新增or修改数据，判定标准->主键的存在与否



> #### lesson 11 数据库的修改和删除

**① 修改**

update() 方法 修改成功return 修改的行数，没有修改（新值==旧值）返回0；如果修改的数据本身包含了主键，可以省略where条件

字段修改时执行SQL函数操作，使用exp()方法实现

```php
Db::name('tablename')->where('field',op)->exp('field',func)->update();
```

自增自减 inc() dec() 可定义步长

::raw方法

save方法修改数据必须指定主键

**② 删除**

i>指定主键

```php
Db::name('tablename')->delete([id1,id2]);
```

ii>根据where删除

```php
Db::name('tablename')->where('field',op)->delete();
```

delete(true)表示删除所有数据



> #### lesson 12 数据库查询表达式

**① 比较查询**

opreation: = < > <>等等

```php
where('field','opreation','conditions')
```

例如 当 expression 为 = 时可以省略

**② 区间查询**

i> 模糊匹配

opreation = like

ii> 也支持数组传递进行模糊查询

```php
Db::name('user')->where('email','like',['xiao%','wu%'],'or')->select();
```

需要配上or 否则默认为and

iii> 快捷方式 whereLike/whereNotLike

iv> between() ->快捷方式 whereBetween/whereNotBetween

v> in() ->快捷方式 whereIn/whereNotIn

vi> null表达式 ->快捷方式 whereNull/whereNotNull

vii> exp拼接查询（按照sql语句写）

```php
Db::name('user')->where('id','exp','in [19,24,27]')->select();
```



> #### lesson 13 数据库的时间查询

**① 传统方式**

通过比较查询、区间查询（between）匹配时间

*时间格式 2019-1-1*

**② 快捷方式**

whereTime, whereBetween，whereBetweenTime

**③ 固定查询**

whereYear('field','year') 查询某年的数据，默认今年

类似的有 whereMonth() ，whereDay()

**④ 其他查询**

查询两个小时内的数据 whereTime('field','-2 hours')->select();

查询两个字段之间的数据whereBetweenTimeField('start_time','end_time')



> #### lesson 14 聚合 原生 子查询

**① 聚合查询**

count() 查询所有数量，count('field') 计算所有非空值的数量

max/min('field')查出最大/小值

avg('field'),sum('field') 平均值/总和

**② 子查询**

fetchSql(bool)	bool= true 不执行sql 返回该sql表达式

buildSql(bool)	bool = true 生成子查询sql语句

闭包方法

```php
//求出所有男性uid
$subQuery =Db::name('two')->field('uid')->where('gender','男')->buildSql(true);
$result = Db::name('one')->where('id','exp','IN'.$subQuery)->select();

//闭包方法
$result = Db::name('one')->where('id','in',function($query){
		$query->name('two')->field('uid')->where('gender','男');
        })->select();
```

**③ 原生查询**

读操作	Db::query('sql语句');

写操作	Db::execute('sql语句');



> #### lesson 15 链式查询方法（一）

**① where**

表达式查询

关联数组查询 where(gender=>'男'，’price‘=>100)

whereRaw 暴力查询

SQL查询使用了预处理模式：id=:id 也是支持的

**② field**

查询指定字段，默认时为* 建议显示查询用true

fieldRaw('field','function')

withoutField('field')排除 多个加逗号，或者用数组表示

调用insert()方法时field()会自检进行字段合法性验证

**③ alias**

起别名



> #### lesson 16 链式查询方法（二）

**① limit**

limit('offset','length') 单一参数时表示length

**② page**

page('pageNum','length') =>分页功能

**③ order**

排序功能 order('field','desc/asc') => 参数可以是数组，完成多字段升降序排列，orderRaw

**④ group**

分字段统计，可以多字段分组

**⑤ having**

group分组后 再筛选



> #### lesson 17 数据库高级查询

① 使用|(OR) 或 &(AND) 实现where高级查询

```php
$user = Db::name('user')
        ->where('username|email','like','%xiao%')
        ->where('uid&price','>',0)
        ->select();
```

② 可以使用数组（默认AND）

③ 可以使用exp对数组中的元素拼接

④ 多个where

⑤ 多次出现一个字段 whereOr ( 否则会触发② )

⑥ 闭包查询可以连缀

⑦ 复杂或者不知道如何拼装时直接使用whereRaw

⑧ 支持参数绑定

```php
$user = Db::name('user')
       ->whereRaw('(username LIKE :username AND status =:status) OR id>:id',
       ['username'=>'%小%','status'=>1, 'id'=>0])
       ->select();
```



> #### lesson 18 数据库快捷查询

① where的封装方法

whereColumn(’field1‘,'op','field2') 比较两个字段

② whereFieldName (FieldName是数据库里的字段名)

③ getFieldbyFieldName(’fieldname‘,'field');

④ when() 条件判断查询 => 等同于if



> #### lesson 19 数据库事务与获取器

（一）事务

InnoDB->执行多个sql时，对相同数据执行操作时定义的先后顺序

① 自动处理

Db::Transaction(function(){需要执行的sql语句})

② 手动处理

```php
Db::startTrans();
try{
    //需要执行的sql语句
    Db::commit();
}catch(\Exception){
    echo '执行失败，回滚中';
    Db::rollback();
}


```

（二）获取器



> #### lesson 20 数据库的数据集和代码提示



### 模型

> #### lesson 21 模型的定义方式

① 定义一个和数据库里的表相匹配的模型

class User extends Model

② 模型会自动对应数据表（记得去除前缀）

如 tp_user => User

③ 通过User::* 方法名去调用 如select()等

④ 别名设置（防止重复命名的冲突）

use app\model\User as 表的别名

⑤ 初始化操作



> #### lesson 22 模型的新增和删除

**① 实例化**

$user = new User(); (实例化（表名）)

$user->字段名 = ’value‘;

$user->save();

也可以通过save([value_array])的方式进行传值修改；

allowField()方法：只允许方法内确认的字段才能赋值

replace() 方法：实现REPLACE INTO

saveAll() 方法：批量存储

**② 静态方法::create()创建数据**

不需要实例化

create([value_array],[allow_filed],bool false);

默认insert true时为replace

**③ 数据删除**

$user -> User::find(id);

$user->delete;

destroy(主键 or [主键数组]) 静态方法 通过主键直接删除

or User::destroy(function($query)){$query->where()}

或者通过User::where(条件)->delete();



>#### lesson 23 模型数据更新

① 更新数据









### 路由

> #### lesson 40 路由的定义

路由：让URL地址栏更规范

config/app.php 开启路由	

路由配置在config/route.php 定义在route/app.php

