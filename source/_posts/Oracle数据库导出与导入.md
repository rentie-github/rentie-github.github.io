title: Oracle数据库导出与导入
author: Mario
date: 2021-01-07 15:43:55
tags:
---
Oracle数据库迁移可参考操作

<!--more-->
### 数据导出

#### 导出准备
``` bash
1、进入linux命令行，切换oracle用户
  su - oracle
2、linux下任意位置创建任意目录文件夹，如：
  mkdir /home/oracle/dmpfile
3、以sysdba角色登录oracle，执行创建目录命令,as之后的路径为步骤2新建的路径
  create or replace directory data_dmp as '/home/oracle/dmpfile'
4、查看目录是否创建成功
  select * from dba_directories;
```
#### 执行导出

``` bash
1、查找oracle安装路径
  whereis oracle
  查询结果如：oracle: /oracle/app/oracle/product/11.2.0/dbhome_1/bin/oracle
2、进入bin目录下，执行下面命令
  ./expdp username/password@orcl schemas=username directory=data_dmp dumpfile=data.dmp logfile=data.log version=11.2.0.1.0
  其中：
    username：用户名
    password：密码
    orcl：SID根据实际情况做修改
    directory：与导出准备的步骤3创建的目录一致
    dumpfile：导出数据文件名称（可自定义）
    logfile：导出日志文件名称（可自定义）
    version：后续需要进行导入的oracle的版本号
```
### 数据导入

#### 导入准备
``` bash
0、sysdba角色登录oracle
1、创建表空间命令如下，其中表空间名称及文件位置需自行定义
  CREATE TABLESPACE YHTS_DATA DATAFILE 'D:\app\data\ythdata_01.dbf' SIZE 8 G AUTOEXTEND OFF LOGGING   ONLINE
  PERMANENT
  EXTENT MANAGEMENT LOCAL AUTOALLOCATE
  SEGMENT SPACE MANAGEMENT AUTO;
2. 创建临时表空间命令如下，其中表空间名称及文件位置需自行定义
  CREATE TEMPORARY TABLESPACE YTH_TEMP TEMPFILE
  'D:\app\data\YTH_TEMP_01.dbf' SIZE 3G REUSE AUTOEXTEND OFF
  EXTENT MANAGEMENT LOCAL UNIFORM SIZE 10M;
3、查看表空间
  select * from Dba_Tablespaces;
4、创建用户和设置默认表空间命令如下，
  create user 用户名 identified by 密码
  default tablespace 表空间名称
  temporary tablespace 临时表空间名称 profile default;
5、分配权限，用户名自行修改
  GRANT CONNECT TO yth;
  GRANT RESOURCE TO yth;
  GRANT DBA TO yth;
  GRANT EXP_FULL_DATABASE to yth;
  GRANT IMP_FULL_DATABASE to yth;
  GRANT UNLIMITED TABLESPACE TO yth;
  GRANT DEBUG CONNECT SESSION TO yth;
  GRANT DEBUG ANY PROCEDURE TO yth;
  GRANT create any view to yth;  
6、进入linux命令行，切换oracle用户
  su - oracle
7、linux下任意位置创建任意目录文件夹，如：
  mkdir /home/oracle/dmpfile
8、以sysdba角色登录oracle，执行创建目录命令,as之后的路径为步骤7新建的路径
  create or replace directory data_dmp as '/home/oracle/dmpfile'
9、将导出命令生成的数据文件和日志文件，本例中是data.dmp和data.log上传到步骤7创建的文件夹中
```
#### 执行导入
``` bash
1、查找oracle安装路径
  whereis oracle
  查询结果如：oracle: /oracle/app/oracle/product/11.2.0/dbhome_1/bin/oracle
2、若数据库有多个SID，需要指定实例
  windows下 set ORACLE_SID=xxxx 该修改仅对当前CMD窗口有效
  linux下 export ORACLE_SID=xxxx; 
3、进入bin目录下，执行下面命令，导入数据库
  ./impdp  userName02/password02 directory=data_dmp logfile=data.log dumpfile=data.dmp    remap_schema=userName01:userName02  remap_tablespace=tablespace01:tablespace02
  其中：
  userName02：导入数据库的用户名
  password02：导入数据库的密码
  directory：导入准备中步骤8创建的目录名称
  logfile：导入准备中步骤9上传的数据文件名称
  dumpfile：导入准备中步骤9上传的日志文件名称
  remap_schema： 指定用户。userName01是导出数据源的用户名 userName02是目标数据的用户名
  remap_tablespace: 指定表空间。tablespace01是导出数据源的表空间 tablespace02是目标数据库的表空间
3、导入注释和索引，
./impdp  userName02/password02 directory=data_dmp logfile=data.log dumpfile=data.dmp table_exists_action=skip include=index,constraint,comment remap_schema=userName01:userName02  remap_tablespace=tablespace01:tablespace02
  其中：
  userName02：导入数据库的用户名
  password02：导入数据库的密码
  directory：导入准备中步骤8创建的目录名称
  logfile：导入准备中步骤9上传的数据文件名称
  dumpfile：导入准备中步骤9上传的日志文件名称
  remap_schema： 指定用户。userName01是导出数据源的用户名 userName02是目标数据的用户名
  remap_tablespace: 指定表空间。tablespace01是导出数据源的表空间 tablespace02是目标数据库的表空间
```
#### 导入导出其他参数介绍
``` bash
1、导出参数
   expdp s/s directory=dump_dir1 dumpfile=sas1.dmp logfile=aa.log
   schemas=scott 
   TABLES=emp,dept 
   query='WHERE deptno=20'    带条件
   TABLESPACES=temp,example  按表空间
   FULL=y       整个库
   content=all/metadata_only/data_only 将导出对象定义及其所有数据/只导出对象定义/只导出对象数据
2、导入参数
    impdp
      REMAP_SCHEMA=scott:system    更改owner
      TABLESPACES=example   导入表空间
      partition_options=merge  将分区的数据合并在一起
      TABLE_EXISTS_ACTION   追加数据
      table_exists_action   {skip 是如果已存在表，则跳过并处理下一个对象；
      append是为表增加数据；
      truncate是截断表，然后为其增加新数据；
      replace是删除已存在表，重新建表并追加数据} 
```
#### 涉及命令记录
``` bash
1、删除目录命令（需要时执行），PATH为导出准备步骤4查询出来的DIRECTORY_NAME字段值
  drop directory PATH;
2、删除用户和关联关系（需要时执行）
  drop user yth cascade;  
3、删除表空间（需要时执行）
  DROP TABLESPACE YHTS_DATA INCLUDING CONTENTS AND DATAFILES;  
4、查看用户所在的表空间（需要时执行）
  select default_tablespace from dba_users where username='yth';
```

### 注意
导出是什么字符集，导入就要使用什么字符集
``` bash
1、导入/导出（乱码问题，需要设置Windows环境变量）
   NLS_LANG = AMERICAN_AMERICA.ZHS16GBK
```
### 问题及解决办法记录
##### 问题一
``` bash
1、导出时，提示报错信息
   UDE-12154: operation generated ORACLE error 12154
   ORA-12154: TNS:could not resolve the connect identifier specified
2、解决办法
  在SID前添加ip地址，完整导出语句示例如下
  ./expdp username/password@127.0.0.1/orcl schemas=username directory=data_dmp dumpfile=data.dmp logfile=data.log version=11.2.0.1.0
```
##### 问题二
``` bash
1、导出时，提示报错信息
   ORA-39002: invalid operation
   ORA-39070: Unable to open the log file.
   ORA-39087: directory name DUMP_DIR is invalid
2、分析
   数据泵导入导出的时候需要在Oracle内部创建一个directory来指定导出的目录，这个报错是指定的目录中，没有给出相应的read，write权限，导致最后的错误
3、解决办法
  sysdba角色登录oracle，给出相应的权限，执行如下命令
  grant read,write on directory DUMP_DIR to 用户名;
```