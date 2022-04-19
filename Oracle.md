# Oracle

## 创建新用户

1. 开始菜单打开Oracle -> SQL Plus

2. 创建表空间

```sql
CREATE TABLESPACE tablespace_name DATAFILE 'E:\OracleDB\tablespace_name.DBF' size 400M autoextend on next 200M maxsize unlimited EXTENT MANAGEMENT LOCAL SEGMENT SPACE MANAGEMENT AUTO;
```

3. 创建 Oracle 用户

```sql
-- Create the user 
-- user aaa 用户名
-- identified by xxx 密码
-- default tablespace AAA 默认表空间名称，使用刚创建的
-- quota unlimited on AAA 允许用户最大限度的使用表空间中的可用空间
create user user_name identified by xxx default tablespace tablespace_name quota unlimited on tablespace_name;
 
-- Grant/Revoke role privileges 
-- 授权用户登录数据库
grant connect to user_name;

-- 授权用户数据库管理员权限
grant dba to user_name;

-- 授权用户 RESOURCE 角色：仅具有创建CLUSTER,INDEXTYPE,OPERATOR,PROCEDEURE,SEQUENCE,TABLE,TRIGGER,TYPE的权限。同时，当把ORACLE resource角色授予一个user的时候，不但会授予ORACLE resource角色本身的权限，而且还有unlimited tablespace权限，但是，当把resource授予一个role时，就不会授予unlimited tablespace权限。
grant resource to user_name;

-- 授权用户创建视图权限
GRANT create view to user_name;
```

4. 创建目录，已经创建过可以不用创建

```sql
-- 格式：create or replace directory 目录别名 as '目录路径';
create or replace directory dbback as 'E:\OracleDB';
```

## 不同用户之间数据导入导出

### 使用数据泵导出数据

1. 打开cmd
2. 导出数据

``` shell
# 格式：expdp 用户名/密码@ip/服务名（默认服务名为ORCL,如果不是，请改为对应服务名） directory=目录名称 dumpfile=模式名称_日期.dmp logfile=模式名称_日期.log
# 连接本机数据库可不设置ip,格式为：expdp 用户名/密码@服务名，使用SID登录格式为：expdp 用户名/密码
# 登录用户是需要导出的数据对应的用户，如果使用管理员账号，需要指定“schemas=schema_1,schema_2”
# query="'where xxx=xxx'" 带条件导出，指定要添加的条件，把表中的数据进行过滤导出
# CONTENT用于指定要导入/出的内容.默认值为ALL，参数可省略，当设置CONTENT为ALL时,将导出对象定义及其所有数据，为DATA_ONLY时,只导出对象数据，为METADATA_ONLY时,只导出对象定义
# 添加tables=test1,test2，可导出指定表
# 添加 tablespaces=tablespaces_1,tablespaces_2 可以导出指定表空间
expdp user_name/password@ip/server_name directory=dbback dumpfile=schemas_name_%DATE:~,4%%DATE:~5,2%%DATE:~8,2%.dmp logfile=schemas_name_%DATE:~,4%%DATE:~5,2%%DATE:~8,2%.log CONTENT={ALL | DATA_ONLY | METADATA_ONLY}
```

### 使用数据泵导入数据

1. 打开 cmd
2. 在要导入的数据库创建表空间和用户，用户名和表空间名称与导出表空间和用户名不同
3. 将导出的DMP文件放入目录中（远程导出的dmp文件在远程电脑对应的 oracle 目录中，如果需要导入的数据库在本机，需要把导出文件拷贝过来，如果导入导出的是同一个数据库则不需要）
4. 导入备份文件

```shell
# 格式：impdp 用户名/密码@ip/服务名（默认服务名为ORCL,如果不是，请改为对应服务名） dumpfile=需要导入的dmp文件名 directory=目录名称 remap_schema=旧导出库用户名:新导入库用户名 remap_tablespace=旧导出库表空间:新导入库表空间 logfile=模式名称_日期.log transform=OID:N
# 连接本机数据库可不设置ip,格式为：impdp 用户名/密码@服务名，使用SID登录格式为：impdp 用户名/密码
# table_exists_action= （skip 是如果已存在表，则跳过并处理下一个对象；append 是为表增加数据；truncate 是截断表，然后为其增加新数据；replace 是删除已存在表，重新建表并追加数据）
# CONTENT用于指定要导入/出的内容.默认值为ALL，参数可省略，当设置CONTENT为ALL 时,将导入/出对象定义及其所有数据，为DATA_ONLY时,只导入/出对象数据，为METADATA_ONLY时,只导入/出对象定义
# 如果需要导入数据的用户具有DBA权限，那么导入时会按照原来的位置导入数据，即导入到原表空间，所以要设置remap_tablespace
# 如果数据带有对象，需要加上 transform=OID:N
impdp user_name/password@ip/server_name dumpfile=imp_dmp directory=dbback remap_schema=schemas_name_exp:schemas_name_imp remap_tablespace=tablespace_exp:tablespace_imp logfile=schemas_name_%DATE:~,4%%DATE:~5,2%%DATE:~8,2%.log table_exists_action=replace CONTENT={ALL | DATA_ONLY | METADATA_ONLY} transform=OID:N
```

## 相同用户间的数据导入导出

### 使用数据泵导出数据

1. 打开cmd

2. 导出数据

```shell
# 格式：expdp 用户名/密码@ip/服务名（默认服务名为ORCL,如果不是，请改为对应服务名） directory=目录名称 dumpfile=模式名称_日期.dmp logfile=模式名称_日期.log
# 连接本机数据库可不设置ip,格式为：expdp 用户名/密码@服务名，使用SID登录格式为：expdp 用户名/密码
# 登录用户是需要导出的数据对应的用户，如果使用管理员账号，需要指定“schemas=schema_1,schema_2”
# query="'where xxx=xxx'" 带条件导出，指定要添加的条件，把表中的数据进行过滤导出
# CONTENT用于指定要导入/出的内容,参数可省略，默认值为ALL，当设置CONTENT为ALL或不定义CONTENT时,将导出对象定义及其所有数据，为DATA_ONLY时,只导出对象数据，为METADATA_ONLY时,只导出对象定义
# 添加tables=test1,test2，可导出指定表
# 添加 tablespaces=tablespaces_1,tablespaces_2 可以导出指定表空间
expdp user_name/password@ip/server_name directory=dbback dumpfile=schemas_name_%DATE:~,4%%DATE:~5,2%%DATE:~8,2%.dmp logfile=schemas_name_%DATE:~,4%%DATE:~5,2%%DATE:~8,2%.log CONTENT={ALL | DATA_ONLY | METADATA_ONLY}
```

### 使用数据泵导入数据

1. 开始菜单打开Oracle -> SQL Plus
2. 在要导入的数据库创建表空间和用户，用户名和表空间名称与导出表空间和用户名相同
3. 将导出的DMP文件放入目录中（远程导出的dmp文件在远程电脑对应的 oracle 目录中，如果需要导入的数据库在本机，需要把导出文件拷贝过来，如果导入导出的是同一个数据库则不需要）
4. 导入数据

```shell
# 格式：impdp 用户名/密码@ip/服务名（默认服务名为ORCL,如果不是，请改为对应服务名） dumpfile=需要导入的dmp文件名 directory=目录名称 logfile=表空间名称_日期.log
# 连接本机数据库可不设置ip,格式为：impdp 用户名/密码@服务名，使用SID登录格式为：impdp 用户名/密码
# table_exists_action= （skip 是如果已存在表，则跳过并处理下一个对象；append 是为表增加数据；truncate 是截断表，然后为其增加新数据；replace 是删除已存在表，重新建表并追加数据）
# CONTENT用于指定要导入/出的内容.默认值为ALL，参数可省略，当设置CONTENT为ALL 时,将导入/出对象定义及其所有数据，为DATA_ONLY时,只导入/出对象数据，为METADATA_ONLY时,只导入/出对象定义
impdp user_name/password@ip/ORCL dumpfile=imp_dmp directory=dbback logfile=schemas_name_%DATE:~,4%%DATE:~5,2%%DATE:~8,2%.log table_exists_action=replace CONTENT={ALL | DATA_ONLY | METADATA_ONLY}
```

## 使用 DBLINK导出数据

当不方便从远程数据库拿到导出的dmp文件时，可以使用DBLINK把dmp文件直接导入到本机数据库。

1. 在本机 Oracle 添加一个 DBLINK

```sql
create public database link dblink_name connect to user_name identified by "password" using 'ip:port/server_name';
```

2. 在cmd中运行下列命令，导出数据

```sh
# 此时登录用户为本机Oracle用户，并非远程导出的数据库用户
expdp user_name/password network_link=dblink_name directory=directory_name dumpfile=CO2_DEV%DATE:~,4%%DATE:~5,2%%DATE:~8,2%.dmp  SCHEMAS=schema_name logfile=CO2_DEV%DATE:~,4%%DATE:~5,2%%DATE:~8,2%.log 
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
# 检查dblink是否可以成功连接远程数据库
select sysdate from dual @dblink_name;
# 查询所有dblink
select * from ALL_DB_LINKS;
# 查询所有profile
select * from dba_profiles order by PROFILE;
# 查询出指定表空间下的表
select TABLE_NAME,TABLESPACE_NAME from dba_tables where TABLESPACE_NAME='表空间名称';
# 查询指定用户的默认表空间
select default_tablespace from dba_users where username='用户名称';
# 查询出单一表对应的表空间
select tablespace_name,table_name from user_tables where table_name='表名';
```

### 新增

```sql
# 赋予用户无限表空间
grant unlimited tablespace to 用户名
```

### 删除

```sql
# 删除用户，同时会删除用户连带的所有数据
drop user xxx cascade;
# 删除表空间
DROP TABLESPACE xxx INCLUDING CONTENTS AND DATAFILES;
# 回收用户对表空间无限制的权限
revoke unlimited tablespace from 用户名;
# 删除 dblink
drop public database link dblinkname;
```

