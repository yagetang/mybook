# python3.6连接Mysql数据库及简单的增删改查操作

## 方法一：使用pymysql

## 1.通过 pip 安装 pymysql

> ```shell
> # pip3 install pymysql
> Collecting pymysql
>   Downloading http://mirrors.aliyun.com/pypi/packages/e5/07/c0f249aa0b7b0517b5843eeab689b9ccc6a6bb0536fc9d95e65901e6f2ac/PyMySQL-0.8.0-py2.py3-none-any.whl (83kB)
>     100% |████████████████████████████████| 92kB 450kB/s 
> Installing collected packages: pymysql
> Successfully installed pymysql-0.8.0
>
> ```

## 2.测试连接

> ```shell
> # python3 
> Python 3.6.3 (default, Jan 26 2018, 09:26:22) 
> [GCC 4.8.5 20150623 (Red Hat 4.8.5-16)] on linux
> Type "help", "copyright", "credits" or "license" for more information.
> >>> import pymysql
> >>> 
> ```
>
> 未出错，表示连接正常

### 2.1查询操作

> ```python
> import pymysql  #导入 pymysql  
>   
> #打开数据库连接  
> db= pymysql.connect(host="localhost",user="root",  
>     password="123456",db="test",charset='utf8mb4',port=3306)  
>   
> # 使用cursor()方法获取操作游标  
> cur = db.cursor()  
>   
> #1.查询操作  
> # 编写sql 查询语句  user 对应我的表名  
> sql = "select * from user"  
> try:  
>     cur.execute(sql)    #执行sql语句  
>   
>     results = cur.fetchall()    #获取查询的所有记录  
>     print("id","name","password")  
>     #遍历结果  
>     for row in results :  
>         id = row[0]  
>         name = row[1]  
>         password = row[2]  
>         print(id,name,password)  
> except Exception as e:  
>     raise e  
> finally:  
>     db.close()  #关闭连接 
> ```

### 2.2插入操作

> ```python
> import pymysql  
> #2.插入操作  
> db= pymysql.connect(host="localhost",user="root",  
>     password="123456",db="test",charset='utf8mb4',port=3306)  
>   
> # 使用cursor()方法获取操作游标  
> cur = db.cursor()  
>   
> sql_insert ="""insert into user(id,username,password) values(4,'liu','1234')"""  
>   
> try:  
>     cur.execute(sql_insert)  
>     #提交  
>     db.commit()  
> except Exception as e:  
>     #错误回滚  
>     db.rollback()   
> finally:  
>     db.close()  
> ```

### 2.3更新操作

> ```python
> import pymysql  
> #3.更新操作  
> db= pymysql.connect(host="localhost",user="root",  
>     password="123456",db="test",charset='utf8mb4',port=3306)  
>   
> # 使用cursor()方法获取操作游标  
> cur = db.cursor()  
>   
> sql_update ="update user set username = '%s' where id = %d"  
>   
> try:  
>     cur.execute(sql_update % ("xiongda",3))  #像sql语句传递参数  
>     #提交  
>     db.commit()  
> except Exception as e:  
>     #错误回滚  
>     db.rollback()   
> finally:  
>     db.close() 
> ```

### 2.4删除操作

> ```python
> import pymysql  
> #4.删除操作  
> db= pymysql.connect(host="localhost",user="root",  
>     password="123456",db="test",charset='utf8mb4',port=3306)  
>   
> # 使用cursor()方法获取操作游标  
> cur = db.cursor()  
>   
> sql_delete ="delete from user where id = %d"  
>   
> try:  
>     cur.execute(sql_delete % (3))  #像sql语句传递参数  
>     #提交  
>     db.commit()  
> except Exception as e:  
>     #错误回滚  
>     db.rollback()   
> finally:  
>     db.close() 
> ```

### 3 关于pymysql防注入

3.1 字符串拼接查询，造成注入

正常查询语句：

> ```python
> #! /usr/bin/env python
> # -*- coding:utf-8 -*-
> import pymysql
>  
> conn = pymysql.connect(host='127.0.0.1', port=3306, user='root', passwd='', db='tkq1')
> cursor = conn.cursor()
> user="u1"
> passwd="u1pass"
> #正常构造语句的情况
> sql="select user,pass from tb7 where user='%s' and pass='%s'" % (user,passwd)
> #sql=select user,pass from tb7 where user='u1' and pass='u1pass'
> row_count=cursor.execute(sql) row_1 = cursor.fetchone()
> print row_count,row_1
>  
> conn.commit()
> cursor.close()
> conn.close()
> ```
>
> 

构造注入语句：

> ```Python
> #! /usr/bin/env python
> # -*- coding:utf-8 -*-
> import pymysql
>  
> conn = pymysql.connect(host='127.0.0.1', port=3306, user='root', passwd='', db='tkq1')
> cursor = conn.cursor()
>  
> user="u1' or '1'-- "
> passwd="u1pass"
> sql="select user,pass from tb7 where user='%s' and pass='%s'" % (user,passwd)
>  
> #拼接语句被构造成下面这样，永真条件，此时就注入成功了。因此要避免这种情况需使用pymysql提供的参数化查询。
> #select user,pass from tb7 where user='u1' or '1'-- ' and pass='u1pass'
>  
> row_count=cursor.execute(sql)
> row_1 = cursor.fetchone()
> print row_count,row_1
>  
> conn.commit()
> cursor.close()
> conn.close()
> ```
>
> 

3. 2 避免注入，使用pymysql提供的参数化语句

正常参数化查询

> ```python
> #! /usr/bin/env python
> # -*- coding:utf-8 -*-
>  
> import pymysql
>  
> conn = pymysql.connect(host='127.0.0.1', port=3306, user='root', passwd='', db='tkq1')
> cursor = conn.cursor()
> user="u1"
> passwd="u1pass"
> #执行参数化查询
> row_count=cursor.execute("select user,pass from tb7 where user=%s and pass=%s",(user,passwd))
> row_1 = cursor.fetchone()
> print row_count,row_1
>  
> conn.commit()
> cursor.close()
> conn.close()
> ```

构造注入，参数化查询注入失败。

> ```python
> #! /usr/bin/env python
> # -*- coding:utf-8 -*-
> # __author__ = "TKQ"
> import pymysql
>  
> conn = pymysql.connect(host='127.0.0.1', port=3306, user='root', passwd='', db='tkq1')
> cursor = conn.cursor()
>  
> user="u1' or '1'-- "
> passwd="u1pass"
> #执行参数化查询
> row_count=cursor.execute("select user,pass from tb7 where user=%s and pass=%s",(user,passwd))
> #内部执行参数化生成的SQL语句，对特殊字符进行了加\转义，避免注入语句生成。
> # sql=cursor.mogrify("select user,pass from tb7 where user=%s and pass=%s",(user,passwd))
> # print sql
> #select user,pass from tb7 where user='u1\' or \'1\'-- ' and pass='u1pass'被转义的语句。
>  
> row_1 = cursor.fetchone()
> print row_count,row_1
>  
> conn.commit()
> cursor.close()
> conn.close()
> ```

结论：excute执行SQL语句的时候，必须使用参数化的方式，否则必然产生SQL注入漏洞。

参考来源：https://www.cnblogs.com/wt11/p/6141225.html

## 方法二：使用mysql-connector

### 1.通过 pip 安装 mysql-connector

由于MySQL服务器以独立的进程运行，并通过网络对外服务，所以，需要支持Python的MySQL驱动来连接到MySQL服务器。MySQL官方提供了mysql-connector-python驱动，但是安装的时候需要给pip命令加上参数`--allow-external`：

> ```
> pip3 install mysql-connector-python --allow-external mysql-connector-python
> ```
>
> 如果上面的命令安装失败，可以试试另一个驱动：
>
> ```
> pip3 install mysql-connector
> ```

### 2.测试连接

> ```shell
> # python3 
> Python 3.6.3 (default, Jan 26 2018, 09:26:22) 
> [GCC 4.8.5 20150623 (Red Hat 4.8.5-16)] on linux
> Type "help", "copyright", "credits" or "license" for more information.
> >>> import mysql.connector
> >>> 
> ```
>
> 未出错，表示连接正常
>
> ```python
> # 导入MySQL驱动:
> >>> import mysql.connector
> # 注意把password设为你的root口令:
> >>> conn = mysql.connector.connect(user='root', password='123456', database='test')
> >>> cursor = conn.cursor()
> # 创建user表:
> >>> cursor.execute('create table user (id varchar(20) primary key, name varchar(20))')
> # 插入一行记录，注意MySQL的占位符是%s:
> >>> cursor.execute('insert into user (id, name) values (%s, %s)', ['1', 'Michael'])
> >>> cursor.rowcount
> 1
> # 提交事务:
> >>> conn.commit()
> >>> cursor.close()
> # 运行查询:
> >>> cursor = conn.cursor()
> >>> cursor.execute('select * from user where id = %s', ('1',))
> >>> values = cursor.fetchall()
> >>> values
> [('1', 'Michael')]
> # 关闭Cursor和Connection:
> >>> cursor.close()
> True
> >>> conn.close(
> ```

### 2.1 连接数据库

> ```python
> #!/usr/bin/env python
> #coding:utf-8
> import mysql.connector
> import time
> try:
>     #配置信息
>     config={
>         'host':'localhost',
>         'port':3306,
>         'user':'root',
>         'password':'123456',
>         'database':'test',
>         'charset':'utf8'
>     }
>     #连接数据库
>     # con=mysql.connector.connect(host='localhost',port=3306,user='root',
>     #                         password='root',database='test',charset='utf8')
>     con=mysql.connector.connect(**config)
>     print con.connection_id
>     time.sleep(5)
>     #断开
>     con.close()
> except mysql.connector.Error,e:
>     print e.message
> ```

### 2.2 插入数据

2.2.1 通过字符串直接插入

> ```shell
> #!/usr/bin/env python
> #coding:utf-8
>
> from mysql import connector
>
> #连接
> try:
>     #配置信息
>     config={
>         'host':'localhost',
>         'port':3306,
>         'user':'root',
>         'password':'123456',
>         'database':'test',
>         'charset':'utf8'
>     }
>     #连接数据库
>     # con=mysql.connector.connect(host='localhost',port=3306,user='root',
>     #                         password='root',database='test',charset='utf8')
>     con=connector.connect(**config)
>     cursor=con.cursor()
>
>     #通过字符串直接插入
>     insert1=("insert into user(name,age) values('Tom',20)")
>     cursor.execute(insert1)
>     #提交
>     con.commit()
>     #关闭
>     cursor.close()
>     con.close()
> except connector.Error,e:
>     print e.message
> ```

2.2.2 通过tuple方式插入

> ```python
> #!/usr/bin/env python
> #coding:utf-8
>
> from mysql import connector
>
> #连接
> try:
>     #配置信息
>     config={
>         'host':'localhost',
>         'port':3306,
>         'user':'root',
>         'password':'123456',
>         'database':'test',
>         'charset':'utf8'
>     }
>     #连接数据库
>     # con=mysql.connector.connect(host='localhost',port=3306,user='root',
>     #                         password='root',database='test',charset='utf8')
>     con=connector.connect(**config)
>     cursor=con.cursor()
>
>     #通过tuple方式插入（利用%s作为占位符）
>     insert2=("insert into user(name,age) values(%s,%s)")
>     data=('Tom',20)
>     cursor.execute(insert2,data)
>
>     #提交
>     con.commit()
>     #关闭
>     cursor.close()
>     con.close()
> except connector.Error,e:
>     print e.message
> ```
>
> 

2.2.3 通过dict方式插入

> ```python
> #!/usr/bin/env python
> #coding:utf-8
>
> from mysql import connector
>
> #连接
> try:
>     #配置信息
>     config={
>         'host':'localhost',
>         'port':3306,
>         'user':'root',
>         'password':'123456',
>         'database':'test',
>         'charset':'utf8'
>     }
>     #连接数据库
>     # con=mysql.connector.connect(host='localhost',port=3306,user='root',
>     #                         password='root',database='test',charset='utf8')
>     con=connector.connect(**config)
>     cursor=con.cursor()
>
>     #通过dict方式插入（利用%(字段)s作为占位符）
>     insert3=("insert into user(name,age) values(%(name)s,%(age)s)")
>     data={
>         'name':'Tom',
>         'age':21
>     }
>     cursor.execute(insert3,data)
>
>     #提交
>     con.commit()
>     #关闭
>     cursor.close()
>     con.close()
> except connector.Error,e:
>     print e.message
> ```

2.2.4 批量插入

> ```Python
> #!/usr/bin/env python
> #coding:utf-8
>
> from mysql import connector
>
> #连接
> try:
>     #配置信息
>     config={
>         'host':'localhost',
>         'port':3306,
>         'user':'root',
>         'password':'123456',
>         'database':'test',
>         'charset':'utf8'
>     }
>     #连接数据库
>     # con=mysql.connector.connect(host='localhost',port=3306,user='root',
>     #                         password='root',database='test',charset='utf8')
>     con=connector.connect(**config)
>     cursor=con.cursor()
>
>     #批量插入
>     insertmany=("insert into user(name,age) values(%s,%s)")
>     data=[
>         ('Tom1',20),
>         ('Tom2',21),
>         ('Tom3',22)
>     ]
>     cursor.executemany(insertmany,data)
>
>     #提交
>     con.commit()
>     #关闭
>     cursor.close()
>     con.close()
> except connector.Error,e:
>     print e.message
> ```

### 2.3 删除操作

> ```Python
> #!/usr/bin/env python
> #coding:utf-8
>
> from mysql import connector
>
> try:
>     #配置信息
>     config={
>         'host':'localhost',
>         'port':3306,
>         'user':'root',
>         'password':'123456',
>         'database':'test',
>         'charset':'utf8'
>     }
>
>     #连接数据库
>     con=connector.connect(**config)
>     cursor=con.cursor()
>
>     #直接通过字符串方式
>     delete1=("delete from user where name='Tom'")
>     cursor.execute(delete1)
>
>     #通过tuple方式
>     delete2=("delete from user where name=%s and age=%s")
>     data=('Tom',20)
>     cursor.execute(delete2,data)
>
>     #通过dict方式
>     delete3=("delete from user where name=%(name)s and age=%(age)s")
>     data={
>         'name':'Tom',
>         'age':20
>     }
>     cursor.execute(delete3,data)
>
>     #提交
>     con.commit()
>
>     #关闭
>     cursor.close()
>     con.close
> except connector.Error,e:
>     print e.message
> ```

### 2.4 更新操作

> ```python
> #!/usr/bin/env python
> #coding:utf-8
>
> from mysql import connector
>
> try:
>     #配置信息
>     config={
>         'host':'localhost',
>         'port':3306,
>         'user':'root',
>         'password':'123456',
>         'database':'test',
>         'charset':'utf8'
>     }
>     #连接数据库
>     con=connector.connect(**config)
>     cursor=con.cursor()
>
>     #通过字符串方式直接更新
>     update1=("update user set name='Tom1',age=20 where Id=81")
>     cursor.execute(update1)
>
>     #通过tuple方式
>     update2=("update user set name=%s,age=%s where Id=%s")
>     data=('Tom2',21,81)
>     cursor.execute(update2,data)
>
>     #通过dict方式
>     update3=("update user set name=%(name)s,age=%(age)s where Id=%(Id)s")
>     data={
>         'name':'Tom3',
>         'age':29,
>         'Id':81
>     }
>     cursor.execute(update3,data)
>
>     #提交
>     con.commit()
>
>     #关闭
>     cursor.close()
>     con.close()
> except connector.Error,e:
>     print e.message
> ```

### 2.5 查询操作

也支持多种方式，下面以字符串方式为例：

> ```shell
> #!/usr/bin/env python
> #coding:utf-8
>
> from mysql import connector
>
> try:
>     #配置信息
>     config={
>         'host':'localhost',
>         'port':3306,
>         'user':'root',
>         'password':'123456',
>         'database':'test',
>         'charset':'utf8'
>     }
>     #连接
>     con=connector.connect(**config)
>     cursor=con.cursor()
>
>     #利用字符串方式查询
>     query1=("select Id,name,age from user where Id>10")
>     cursor.execute(query1)
>     #取出字段名称集合
>     columns=cursor.column_names
>     #取出全部数据
>     result=cursor.fetchall()
>     print '数据表字段名称：{0}'.format(columns)
>     print '查询结果：{0}'.format(result)
>
>     #关闭
>     cursor.close()
>     con.close()
> except connector.Error,e:
>     print e.message
> ```
>
> 结果如下：
>
> ```
> 数据表字段名称：(u'Id', u'name', u'age')
> 查询结果：[(80, u'Tome', 30), (81, u'Tom3', 29), (82, u'Andy', 25)]12
> ```
>
> 查询出来的字符串都是Unicode编码方式。另外，我们可以看到，查询的结果是一个list类型，其中每一项都是tuple，但是直观上不知道tuple中的每个值对应哪一个字段，为此，我们可以对cursor对象重新定义，让其输出dict:
>
> ```python
> #!/usr/bin/env python
> #coding:utf-8
>
> from mysql import connector
>
> try:
>     #配置信息
>     config={
>         'host':'localhost',
>         'port':3306,
>         'user':'root',
>         'password':'123456',
>         'database':'test',
>         'charset':'utf8'
>     }
>
>     #连接
>     con=connector.connect(**config)
>     cursor=con.cursor(cursor_class=connector.cursor.MySQLCursorDict)
>
>     #利用字符串方式查询
>     query1=("select Id,name,age from user where Id>10")
>     cursor.execute(query1)
>     #取出字段名称集合
>     columns=cursor.column_names
>     #取出全部数据
>     result=cursor.fetchall()
>     print '数据表字段名称：{0}'.format(columns)
>     print '查询结果：{0}'.format(result)
>
>     #关闭
>     cursor.close()
>     con.close()
> except connector.Error,e:
>     print e.message
> ```
>
> 结果执行如下：
>
> ```shell
> 数据表字段名称：(u'Id', u'name', u'age')
> 查询结果：[{u'age': 30, u'Id': 80, u'name': u'Tome'}, {u'age': 29, u'Id': 81, u'name': u'Tom3'}, {u'age': 25, u'Id': 82, u'name': u'Andy'}
> ```
>
> 两者的区别在于我们显示定义：
>
> ```shell
> cursor_class=connector.cursor.MySQLCursorDict
> ```

要特别注意中文字符的问题，比如需要插入含有中文的字符串，或者表名、字段名是中文等等，这里给出一个例子作为参考：

> ```Python
> # /usr/bin/env python
> # coding:utf-8
>
> import mysql.connector
>
> # 配置信息
> config = {
>     'host': 'localhost',
>     'port': 3306,
>     'user': 'root',
>     'password': '123456',
>     'database': 'test',
>     'charset': 'utf8'
> }
>
> conn = mysql.connector.connect(**config)
> cursor = conn.cursor(cursor_class=mysql.connector.cursor.MySQLCursorDict)
>
> #插入数据
> data = [
>     ('Tom', '2010-01-01 00:00:00', '牛逼'),
>     ('kikay', '2010-11-11 10:33:33', '相当牛逼'),
> ]
> cursor.executemany(u'insert into member(name,bir,简介) values(%s,%s,%s)', data)
> conn.commit()
>
> # 查询
> cursor.execute('select * from member')
> rows = cursor.rowcount
> columns = cursor.column_names
> data = cursor.fetchall()
> # data=cursor.fetchone()
>
> # 处理含中文的字段
> if data and len(data) > 0:
>     print data[0][u'简介']
>
> cursor.close()
> conn.close()
> ```

参考来源：http://blog.csdn.net/kikaylee/article/details/53569058