# Oracle

## 创建新用户

1. 开始菜单打开Oracle -> SQL Plus

2. 创建表空间

```sql
CREATE TABLESPACE AAA DATAFILE 'E:\OracleDB\AAA.DBF' size 400M autoextend on next 200M maxsize unlimited EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
```

3. 创建 Oracle 用户

```sql
-- Create the user 
-- user aaa 用户名
-- identified by xxx 密码
-- default tablespace AAA 默认表空间名称，使用刚创建的
-- quota unlimited on AAA 允许用户最大限度的使用表空间中的可用空间
create user aaa identified by xxx default tablespace AAA quota unlimited on AAA;
 
-- Grant/Revoke role privileges 
-- 授权用户登录数据库
grant connect to aaa;

-- 授权用户数据库管理员权限
grant dba to aaa;

-- 授权用户 RESOURCE 角色：仅具有创建CLUSTER,INDEXTYPE,OPERATOR,PROCEDEURE,SEQUENCE,TABLE,TRIGGER,TYPE的权限。同时，当把ORACLE resource角色授予一个user的时候，不但会授予ORACLE resource角色本身的权限，而且还有unlimited tablespace权限，但是，当把resource授予一个role时，就不会授予unlimited tablespace权限。
grant resource to aaa;

-- 授权用户创建视图权限
GRANT create view to aaa;

-- 创建目录，已经创建过可以不用创建
create or replace directory dbback as 'E:\OracleDB';
```

## 不同用户之间数据导入导出

### 使用数据泵导出数据

1. 打开cmd
2. 导出数据

``` shell
# expdp 用户名/密码@ip/ORCL directory=目录名称 dumpfile=表空间名称_日期.dmp logfile=表空间名称_日期.log
# 用户一定是需要导出的表空间对应的用户，不是管理员账号
# query="'where xxx=xxx'" 带条件导出，指定要添加的条件，把表中的数据进行过滤导出
# CONTENT用于指定要导入/出的内容.默认值为ALL，可不写，当设置CONTENT为ALL时,将导出对象定义及其所有数据，为DATA_ONLY时,只导出对象数据，为METADATA_ONLY时,只导出对象定义
expdp bbb/xxx@xxx.xxx.xxx.xxx/ORCL directory=dbback dumpfile=BBB_20210823.dmp logfile=BBB_20210823.log CONTENT={ALL | DATA_ONLY | METADATA_ONLY}
```

### 使用数据泵导入数据

1. 打开 cmd
2. 将导出的DMP文件放入目录中（远程导出的dmp文件在远程电脑对应的 oracle 目录中，如果需要导入的数据库在本机，需要把导出文件拷贝过来，如果导入导出的是同一个数据库则不需要）
3. 导入备份文件

```shell
# impdp 用户名/密码@ip/ORCL dumpfile=表空间名称_日期.dmp directory=目录名称 remap_schema=导出表空间名称:导入表空间名称 logfile=表空间名称_日期.log 
# table_exists_action= （skip 是如果已存在表，则跳过并处理下一个对象；append 是为表增加数据；truncate 是截断表，然后为其增加新数据；replace 是删除已存在表，重新建表并追加数据）
# CONTENT用于指定要导入/出的内容.默认值为ALL，可不写，当设置CONTENT为ALL 时,将导入/出对象定义及其所有数据，为DATA_ONLY时,只导入/出对象数据，为METADATA_ONLY时,只导入/出对象定义
impdp aaa/xxx@xxx.xxx.xxx.xxx/ORCL dumpfile=AAA_20210823.dmp directory=dbback remap_schema=BBB:AAA logfile=AAA_20210823.log table_exists_action=replace CONTENT={ALL | DATA_ONLY | METADATA_ONLY}
```

## 相同用户间的数据导入导出

### 使用数据泵导出数据

1. 打开cmd

2. 导出数据

```shell
# expdp 用户名/密码@ip/ORCL directory=目录名称 dumpfile=表空间名称_日期.dmp logfile=表空间名称_日期.log
# 用户一定是需要导出的表空间对应的用户，不是管理员账号
# query="'where xxx=xxx'" 带条件导出，指定要添加的条件，把表中的数据进行过滤导出
# CONTENT用于指定要导入/出的内容,可不写，默认值为ALL，当设置CONTENT为ALL或不定义CONTENT时,将导出对象定义及其所有数据，为DATA_ONLY时,只导出对象数据，	为METADATA_ONLY时,只导出对象定义
# 添加tables=test1,test2，可导出指定表
# 如果使用管理员用户，添加 schemas=test1,test2 可以导出指定用户关联的数据
# 添加 tablespaces=test,test 可以导出指定表空间
expdp bbb/xxx@xxx.xxx.xxx.xxx/ORCL directory=dbback dumpfile=BBB_20210823.dmp logfile=BBB_20210823.log CONTENT={ALL | DATA_ONLY | METADATA_ONLY}
```

### 使用数据泵导入数据

1. 开始菜单打开Oracle -> SQL Plus
2. 在目标数据库创建表空间和用户，用户名和表空间名称与导出表空间和用户名相同
3. 导入数据

```shell
# impdp 用户名/密码@ip/ORCL dumpfile=表空间名称_日期.dmp directory=目录名称 logfile=表空间名称_日期.log 
# table_exists_action= （skip 是如果已存在表，则跳过并处理下一个对象；append 是为表增加数据；truncate 是截断表，然后为其增加新数据；replace 是删除已存在表，重新建表并追加数据）
# CONTENT用于指定要导入/出的内容.默认值为ALL，可不写，当设置CONTENT为ALL 时,将导入/出对象定义及其所有数据，为DATA_ONLY时,只导入/出对象数据，为METADATA_ONLY时,只导入/出对象定义
impdp aaa/xxx@xxx.xxx.xxx.xxx/ORCL dumpfile=AAA_20210823.dmp directory=dbback logfile=AAA_20210823.log table_exists_action=replace CONTENT={ALL | DATA_ONLY | METADATA_ONLY}
```

## 一些常用方法

### 查询

```sql
# oracle 用户查询
select *　from dba_users;
# 表空间查询
select * from dba_data_files;
# 查询目录
SELECT * FROM dba_directories;
```

### 删除

```sql
# 删除用户，同时会删除用户连带的所有数据
drop user xxx cascade;
# 删除表空间
DROP TABLESPACE xxx INCLUDING CONTENTS AND DATAFILES;
```

