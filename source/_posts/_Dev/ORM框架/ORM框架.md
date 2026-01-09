# ORM框架

sql面向命令

ORM面向状态



## Java

### Mybatis

[Mybatis笔记](./Mybatis.md)



### Spring Data JPA (推荐)

[Spring Data JPA笔记](../Spring/spring-data/spring_data_jpa.md)



#### querydsl

单表查询 JPA，多表查询 querydsl



## Python

[DB-API2.0](https://peps.python.org/pep-0249/)：数据库的第三方驱动程序

| ORM                                                          | Driver                                           |
| ------------------------------------------------------------ | ------------------------------------------------ |
| [sqlalchemy](https://github.com/sqlalchemy/sqlalchemy)<br />pandas.dataframe.to_sql()底层就是sqlalchemy，所以只需要看sqlalchemy就好 | [psycopg3](https://github.com/psycopg/psycopg)   |
| [sqlmodel](https://github.com/fastapi/sqlmodel)<br />Fastapi同作者，但官网没有异步教程，需要自己踩坑，基于 SQLAlchemy + Pydantic |                                                  |
| [gino](https://github.com/python-gino/gino)<br />基于sqlalchemy |                                                  |
|                                                              |                                                  |
| [tortoise-orm](https://github.com/tortoise/tortoise-orm)     | [asyncpg](https://github.com/MagicStack/asyncpg) |
|                                                              |                                                  |

|               | 连接字符串                                                   |
| ------------- | ------------------------------------------------------------ |
| JDBC          | driveType:databaseType://host:port/database?user=root&password=MySQL1616<br />jdbc:mysql://localhost:3306/db_name?useUnicode=true&amp;characterEncoding=UTF-8&serverTimezone=Asia/Shanghai |
| Python DB-API | driver://user:pass@host/dbname?arg=value<br />postgresql+psycopg3://nemesis:***@192.168.1.151:5432/a?b=c |
|               |                                                              |



### sqlalchemy2.1



#### alembic

[alembic数据库迁移工具#github](https://github.com/sqlalchemy/alembic)



### tortoise-orm

基于 asyncio的异步ORM库

[官方文档#看Examples、Contrib](https://tortoise.github.io/reference.html)	|	[github](https://github.com/tortoise/tortoise-orm/)
[biup-7y记的笔记](https://www.yuque.com/u1362970/xyh2wn/dwd9iqkg0sheog6b)	|	[方法](https://juejin.cn/post/7172055019926077471)



**字段构造函数签名**

```python
# 字段构造函数签名
tortoise.fields.Field

def __init__(
    self,
    source_field: Optional[str] = None,
    generated: bool = False,
    primary_key: Optional[bool] = None,  # 
    null: bool = False,  # 
    default: Any = None,  # 
    unique: bool = False,  # 
    db_index: Optional[bool] = None,
    description: Optional[str] = None,  # 
    model: "Optional[Model]" = None,
    validators: Optional[List[Union[Validator, Callable]]] = None,
    **kwargs: Any,
)

# 外键构造函数签名，属于字段构造函数签名
# (model_name=appname.modelname指定related model, related_name指定related model引用名称, to_field指定related model外键字段, db_constraint) -> fields.ForeignKeyRelation[modeltype]
# 默认的app name为models
#related_name用于related model(referenced model)的反向解析关联查询
# to_field指定related model的外键属性名称
# source_field 数据库列名称
```

```python
# 创建实例
Tournament(name='New Tournament').save()
Tournament.create(name='Another Tournament')
```

**双下划线**

```python
# DB-backing field数据库支持字段: account_id，支持双下划线嵌套查询 `FKNAME__FOREIGNFIELD__gt=3`
Tournament.filter(name__contains='Another').first()
```





#### aerich

[aerich数据库迁移工具#github](https://github.com/tortoise/aerich)

ORM迁移工具(tortoise.models.Model转表，表转tortoise.models.Model)，好处团队成员同步数据库变更

```bash
> pip install aerich
> conda install aiomysql

> aerich init -t app.db.tortoise_mysql.TORTOISE_CONFIG  # 生成初始化配置pyproject.toml 和 迁移文件./migrations
Success create migrate location ./migrations
Success write config to pyproject.toml

> aerich init-db  # 初始化数据库
Success create app migrate location migrations\models
Success generate schema for app "models"

>aerich migrate  # 修改model类，重新生成迁移文件./migrations
>aerich heads
>aerich upgrade  # 执行修改
>aerich downgrade  # 回退修改
>aerich history
>aerich inspectdb -t tablename > xxmodel.py
```

问题：tortoise.exceptions.DBConnectionError: Can't connect to MySQL server: {'host': '127.0.0.1', 'port': 3306, 'user': 'root', 'db': 'db_hello_fastapi2', 'autocommit': True, 'charset': 'utf8mb4', 'minsize': 1, 'maxsize': 5, 'echo': True, 'sql_mode': 'STRICT_TRANS_TABLES'}
原因：tortoise_mysql.py 中指定的 DB_NAME 数据库不存在
解决：确保数据库存在

