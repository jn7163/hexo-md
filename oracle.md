---
title: oracle 12c 简明笔记
date: 2017-05-06 18:22:00
categories:
- 数据库
- oracle
tags:
- 数据库
- oracle
keywords: oracle 12c安装配置
---

> 
Oracle Database，又名Oracle RDBMS，或简称Oracle, 是甲骨文公司的一款关系数据库管理系统，到目前仍在数据库市场上占有主要份额

<!-- more -->

## java环境配置
[SunJDK for Linux 配置](https://www.zfl9.com/jdk.html)

## 图形界面安装
<pre><code class="language-bash line-numbers"><script type="text/plain">## ssh_X11_forward
yum -y install xorg-x11-xauth xorg-x11-server-utils xterm
/etc/ssh/sshd_config    打开x11_forward
systemctl restart sshd
export DISPLAY=localhost:10.0   # 如果有该变量则无需设置
xhost +
Xmanager5 进行图形界面安装

## 环境变量 /etc/profile.d/oracle.sh
export ORACLE_BASE=/opt/oracle
export ORACLE_HOME=$ORACLE_BASE/product/12.1.0/dbhome_1
export TMP=/tmp
export TMPDIR=/tmp
export ORACLE_SID=orcl
export PATH=$PATH:$ORACLE_HOME/bin
export NLS_LANG=AMERICAN_AMERICA.UTF8

## /etc/pam.d/login
session    required     pam_limits.so

## unzip oracel安装包
unzip 1of2.zip -d /tmp/
unzip 2of2.zip -d /tmp/

## oracle for centos/rhel源，oracle环境配置工具
# 6.x
wget http://yum.oracle.com/public-yum-ol6.repo -O /etc/yum.repos.d/oracle.repo
# 7.x
wget http://yum.oracle.com/public-yum-ol7.repo -O /etc/yum.repos.d/oracle.repo
# gpg-key
wget http://public-yum.oracle.com/RPM-GPG-KEY-oracle-ol6 -O /etc/pki/rpm-gpg/RPM-GPG-KEY-oracle

## 安装oracle环境配置包
该rpm包会自动设置好oracle需要的环境(limits.conf, sysctl.conf, 新建oracle用户...)
yum -y install oracle-rdbms-12cR1-preinstall
echo 123456 | passwd --stdin oracle     # 设置密码，激活帐户

## /etc/sysctl.conf
"vm.hugetlb_shm_group=54321"
sysctl -p

## swap虚拟内存设置
内存大小    swap大小
1-2G        1.5倍
2-16G       1倍
16G+        16G

mkswap /dev/sdb1
swapon /dev/sdb1
free -h

--- /etc/fstab ---
/dev/sdb1 swap swap defaults 0 0

## 创建目录
mkdir /opt/{oracle,oraInventory}
chown oracle:oinstall /opt/{oracle,oraInventory}
chmod 775 /opt/{oracle,oraInventory}

## Xmanager5 进行安装
ssh -Y oracle@127.0.0.1
cd /tmp/database/
./runInstaller

"encoding: utf-8"
"取消勾选cdb"

## sqlplus方向键，退格键乱码问题
rpm -ivh https://raw.github.com/zfl9/rlwrap/master/rlwrap-0.41-1.el6.x86_64.rpm
alias sqlplus='rlwrap sqlplus'

## $ORACLE_HOME/bin/dbstart|dbshut
"ORACLE_HOME_LISTENER＝$ORACLE_HOME"
/etc/oratab 把'N'改为'Y'
</script></code></pre>

## silent静默安装
<pre><code class="language-bash line-numbers"><script type="text/plain">## 安装数据库软件
/tmp/database/runInstaller -silent -responseFile ~/install.rsp
--- ~/install.rsp ---
oracle.install.responseFileVersion=/oracle/install/rspfmt_dbinstall_response_schema_v12.1.0
oracle.install.option=INSTALL_DB_SWONLY
ORACLE_HOSTNAME=localhost.localdomain
UNIX_GROUP_NAME=oinstall
INVENTORY_LOCATION=/opt/oraInventory
SELECTED_LANGUAGES=en,zh_CN
ORACLE_HOME=/opt/oracle/product/12.1.0/dbhome_1
ORACLE_BASE=/opt/oracle
oracle.install.db.InstallEdition=EE
oracle.install.db.DBA_GROUP=dba
oracle.install.db.OPER_GROUP=dba
oracle.install.db.BACKUPDBA_GROUP=dba
oracle.install.db.DGDBA_GROUP=dba
oracle.install.db.KMDBA_GROUP=dba
oracle.install.db.isRACOneInstall=false
oracle.install.db.rac.serverpoolCardinality=0
SECURITY_UPDATES_VIA_MYORACLESUPPORT=false
DECLINE_SECURITY_UPDATES=true

## 配置监听
netca -silent -responseFile /tmp/database/response/netca.rsp

## 创建数据库
dbca -silent -responseFile ~/dbca.rsp
--- dbca.rsp ---
[GENERAL]
RESPONSEFILE_VERSION = "12.1.0"
OPERATION_TYPE = "createDatabase"
[CREATEDATABASE]
GDBNAME = "orcl"
SID = "orcl"
TEMPLATENAME = "General_Purpose.dbc"
SYSPASSWORD = "123456"
SYSTEMPASSWORD = "123456"
DATAFILEDESTINATION = /opt/oracle/oradata
CHARACTERSET = "AL32UTF8"
TOTALMEMORY = "512"
</script></code></pre>

## oracle基础
<pre><code class="language-bash line-numbers"><script type="text/plain">## oracle基本概念
# oracle启动流程
nomount -> mount -> open
nomount 读取初始化参数文件，启动实例，此时可以创建数据库；
mount   打开控制文件，进行维护数据库操作；
open    打开数据文件，日志文件，可以为所有用户提供服务了；

startup 相当于startup nomount; alter database mount; alter database open;

# 基本概念
db_name     数据库名，不宜修改 "show parameter db_name"
sid         数据库实例名，数据库和数据库实例的关系类似于类与实例，一对一，一对多的关系，"show parameter instance"
oracle_sid  环境变量，当sqlplus sys/PASS as sysdba时，默认连接该变量的sid
db_domain   数据库域名，主要用于oracle分布式环境的复制，"show parameter domain"
global_name 全局数据库名，"db_name + db_domain" 等于 "service_name"
service_name数据库服务名，当存在db_domain时，等于全局数据库名，否则等于数据库名 "show parameter service_name"

# 表空间
数据库有一个或多个表空间，表空间由数据文件组成;
Oracle 10g版本之前是"system"
Oracle 10g之后(含)是"users"为默认表空间
用户管理各自的表空间，互不干扰
</script></code></pre>

## listener.ora tnsnames.ora
<pre><code class="language-bash line-numbers"><script type="text/plain">## $ORACLE_HOME/network/admin/listener.ora  用于server端
## $ORACLE_HOME/network/admin/tnsnames.ora  用于client端

--- sqlnet.ora ---
NAMES.DIRECTORY_PATH= (TNSNAMES, EZCONNECT)

--- listener.ora ---
LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 0.0.0.0)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
SID_LIST_LISTENER=
  (SID_LIST=
  (SID_DESC=
         (GLOBAL_DBNAME=orcl)
         (SID_NAME=orcl)
         (ORACLE_HOME=/opt/oracle/product/12.1.0.2.0/dbhome_1)
      )
  (SID_DESC=
         (GLOBAL_DBNAME=zfl)
         (SID_NAME=zfl)
         (ORACLE_HOME=/opt/oracle/product/12.1.0.2.0/dbhome_1)
      )
   )

--- tnsnames.ora ---
ZFL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = zfl)
    )
  )
LISTENER_ZFL =
  (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521))
ORCL =
  (DESCRIPTION =
    (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521))
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
LISTENER_ORCL =
  (ADDRESS = (PROTOCOL = TCP)(HOST = 127.0.0.1)(PORT = 1521))
</script></code></pre>

## 基本命令
<pre><code class="language-bash line-numbers"><script type="text/plain">dbca    数据库配置助手(建库，删库，管理配置等)

dbstart|dbshut  启动关闭脚本

lsnrctl start|stop|reload [监听项] 1521/tcp

sqlplus /nolog
sqlplus sys/123456 as sysdba
sqlplus sys/123456@192.168.255.101/orcl as sysdba
sqlplus sys/123456@zfl as sysdba

startup|shutdown    启动关闭数据库
conn | disc 连接|断开连接
@/root/create_db_zfl.sql    执行sql文件
</script></code></pre>

## 归档模式
<pre><code class="language-bash line-numbers><script type="text/plain"># 查询
select name,log_mode from v$database;
archive log list;

# 修改为归档模式
mkdir /opt/oracle/archdata && chown oracle:oinstall /opt/oracle/archdata && chmod 775 /opt/oracle/archdata
alter system set log_archive_dest_1='location=/opt/oracle/archdata' scope=both;
shutdown immediate;
startup mount
alter database archivelog;
alter database open;
alter system archive log start;
</script></code></pre>

## 基本sql
<pre><code class="language-bash line-numbers"><script type="text/plain">## startup|shutdown
startup open            默认参数为open,打开数据库，允许数据库的访问
startup mount           给dba进行管理操作，不允许数据库的访问
startup nomount         仅仅通过初始化文件，分配sga区，启动数据库后台进程，不能访问任何数据库
startup pfile=FileName  以FileName作为初始化文件，启动数据库
startup force           终止当前数据库的运行，并重新打开数据库
startup restrict        只允许具有 restricted session 权限的用户访问数据库
startup recover         数据库启动，并开始介质恢复

shutdown normal         默认参数为normal,等待会话结束，等待事务结束
shutdown transactional  不等待会话结束，等待事务结束
shutdown immediate      不等待会话结束，不等待事务结束
shutdown abort          不等待会话结束，不等待事务结束，启动时自动进行实例恢复

## select
select database_name from v$database;   # 查看所有库名
select name from v$database;
show user;                              # 查看当前用户
desc v$database;                        # 查看数据库结构
select instance_name from v$instance;   # 查询实例名
select username,default_tablespace from dba_users where username='QBOA';    # 查询用户所属表空间

## 表连接
select * from all_tables;
select table_name from all_tables;
select table_name from all_tables where owner='zfl';
select tab1.email,tab2.* from tab1,tab2 where tab1.id = tab2.id;
select * from t1 [inner] join t2 on t1.id=t2.id;# 内连接
select * from t1 left join t2 on t1.id=t2.id;   # 左连接
select * from t1 right join t2 on t1.id=t2.id;  # 右连接
select * from t1 full join t2 on t1.id=t2.id;   # 完全外连接，等价于左连接+右连接

## grant
desc TABLE_NAME;
create user 用户名 identified by 密码 default tablespace users Temporary TABLESPACE Temp;
grant connect, resource, dba to 用户;
grant sysdba to 用户;
alter user 用户名 identified by 密码;

## show parameter
show parameter db_name;
show parameter instance;
show parameter service_name;
show parameter domain;

## 表空间与用户
# 查看所有表空间
select tablespace_name from dba_tablespaces;

# 创建临时表空间，默认temp
create temporary tablespace db_temp tempfile '/opt/oracle/oradata/ZFL/db_temp.dbf' size 32m autoextend on next 32m maxsize unlimited extent management local;

# 创建表空间(单个数据文件Max_Size 30G)
create tablespace zfl logging datafile '/opt/oracle/oradata/ZFL/zfl.dbf' size 32m autoextend on next 32m maxsize unlimited extent management local;

# 创建用户
create user zfl identified by 123456 account unlock default tablespace zfl temporary tablespace db_temp;

# 授权用户
grant connect,resource,dba to zfl;
commit;

eg: tablespace: xcboa   user: qboa
create tablespace xcboa datafile '/opt/oracle/oradata/orcl/xcboa1.dbf' size 10000M autoextend on next 20M maxsize unlimited;
create user qboa identified by 123456 default tablespace xcboa;
grant connect,resource,dba to qboa;
commit;

# 删除表空间
drop user test cascade;         # 删除用户并删除其数据
alter tablespace test offline;  # 让表空间离线
drop tablespace test including contents and datafiles;  # 删除表空间
</script></code></pre>

## exp/imp
<pre><code class="language-bash line-numbers"><script type="text/plain"># 解决编码问题
服务端："select userenv('language') from dual;"

修改服务器编码, 假定我们要统一编码为GBK

shutdown immediate;
startup mount;
ALTER SYSTEM ENABLE RESTRICTED SESSION;
ALTER SYSTEM SET JOB_QUEUE_PROCESSES=0;
ALTER SYSTEM SET AQ_TM_PROCESSES=0;
alter database open;
ALTER DATABASE CHARACTER SET ZHS16GBK; 报错则执行下句
ALTER DATABASE character set INTERNAL_USE ZHS16GBK;
shutdown immediate;
startup;
select userenv('language') from dual;

客户端：设置环境变量 NLS_LANG=`服务器的编码`

# exp   ignore=y buffer=(bytes)
exp system/123456 full=y file=full.dmp                          # 完全导出,整个db
exp zfl/123456@zfl owner=zfl file=zfl.dmp                       # 指定用户
exp zfl/123456@zfl tables=\(test1,test2\) file=zfl_test1_2.dmp  # 指定表

# imp   ignore=y commit=y buffer=(bytes)
imp system/123456 full=y file=full.dmp
imp zfl/123456@zfl fromuser=zfl touser=zfl file=zfl.dmp
imp zfl/123456@zfl tables=\(test1,test2\) file=zfl_test1_2.dmp
</script></code></pre>

## expdp/impdp 数据泵
<pre><code class="language-bash line-numbers"><script type="text/plain">## 创建directory目录
create directory dpdata as '/opt/oracle/dump'   # 该目录须实际存在，且保证有相应的权限

## 授权
grant read,write on directory dpdata to qboa;

## 查询当前所有目录
select * from dba_directories;

## 删除目录
drop directory dpdata;

expdp导出：expdp qboa/123456@orcl directory=dpdata dumpfile=QBOA.dmp logfile=QBOA.log schemas=qboa

impdp导入：impdp qboa/123456@orcl directory=dpdata dumpfile=QBOA.dmp logfile=QBOA.log schemas=qboa exclude=user
</script></code></pre>

## sql数据类型
<pre><code class="language-bash line-numbers"><script type="text/plain">## 字符串
char    定长字符串，最多2000bytes，默认1byte，可以指定单位为"bytes"|"char"
char(10)        bytes单位
char(10 char)   char单位，utf-8中汉字占3-4字节，英文1字节

nchar       定长字符串，最多2000bytes，包含Unicode
varchar2    变长字符，最多4000bytes

number      数字类型，number(p,s) p:长度(max=38,不含左边的0)，s:精度
s > 0
   精确到小数点右边s位，并四舍五入，然后检验有效位是否 <= p
s < 0
   精确到小数点左边s位，并四舍五入，然后检验有效位是否 <= p + |s|
s = 0
   此时NUMBER表示整数

integer     number子类型，相当于number(38,0)

binary_float    32位,单精度浮点数字数据类型,可以支持至少6位精度,每个BINARY_FLOAT的值需要5个字节，包括长度字节
binary_double   64位,双精度浮点数字数据类型,每个BINARY_DOUBLE的值需要9个字节，包括长度字节

float   number子类型，float(n),n表示精度，按二进制算得精度

date                        日期，时间
timestamp                   时间戳
timestamp with time zone    包含时区偏移量的值
timestamp with local time zone

lob 类型
BLOB、CLOB、NCLOB、BFILE（外部存储）的大型化和非结构化数据，如文本、图像、视屏、空间数据存储;

# CLOB
它存储单字节和多字节字符数据,支持固定宽度和可变宽度的字符集;
CLOB对象可以存储最多(4 gigabytes-1) * (database block size)大小的字符;

# NCLOB
它存储UNICODE类型的数据,支持固定宽度和可变宽度的字符集;
NCLOB对象可以存储最多(4 gigabytes-1) * (database block size)大小的文本数据

# BLOB
它存储非结构化的二进制数据大对象,它可以被认为是没有字符集语义的比特流;一般是图像、声音、视频等文件;
BLOB对象最多存储(4 gigabytes-1) * (database block size)的二进制数据

# BFILE
二进制文件,存储在数据库外的系统文件,只读的,数据库会将该文件当二进制文件处理

# long
类型最多存储2Gbytes数据，推荐用clob代替
</script></code></pre>

## 管理数据库
<pre><code class="language-bash line-numbers"><script type="text/plain">## 建库
1. dbca建库
2. 手动建库
--- create_db.sh ---
#!/bin/bash
export ORACLE_SID=otokaze
orapwd file=$ORACLE_HOME/dbs/orapwotokaze password=123456 entries=10
cat << EOF > $ORACLE_HOME/dbs/initotokaze.ora
db_name=otokaze
control_files='/opt/oracle/oradata/otokaze/control01.ctl'
sga_target=512M
undo_management=auto
undo_tablespace=undotbs
EOF
chown oracle:oinstall $ORACLE_HOME/dbs/*
umask 027
mkdir $ORACLE_BASE/oradata/otokaze && chown oracle:oinstall $ORACLE_BASE/oradata/otokaze
mkdir -p $ORACLE_BASE/admin/otokaze/{adump,bdump,pfile} && chown oracle:oinstall -R $ORACLE_BASE/admin/otokaze/
umask 022
sqlplus sys/123456 as sysdba << EOF
create spfile from pfile;
startup nomount;
CREATE DATABASE otokaze
USER SYS IDENTIFIED BY 123456
USER SYSTEM IDENTIFIED BY 123456
LOGFILE GROUP 1 ('/opt/oracle/oradata/otokaze/redo01a.log') SIZE 20M,
GROUP 2 ('/opt/oracle/oradata/otokaze/redo02a.log') SIZE 20M,
GROUP 3 ('/opt/oracle/oradata/otokaze/redo03a.log') SIZE 20M
MAXLOGFILES 5
MAXLOGMEMBERS 5
MAXLOGHISTORY 1
MAXDATAFILES 100
MAXINSTANCES 2
CHARACTER SET AL32UTF8
DATAFILE '/opt/oracle/oradata/otokaze/system01.dbf' SIZE 400M REUSE
EXTENT MANAGEMENT LOCAL 
SYSAUX DATAFILE '/opt/oracle/oradata/otokaze/sysaux01.dbf' SIZE 400M REUSE
DEFAULT TABLESPACE users
DATAFILE '/opt/oracle/oradata/otokaze/users01.dbf' SIZE 20M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED
DEFAULT TEMPORARY TABLESPACE tempts01
TEMPFILE '/opt/oracle/oradata/otokaze/tempts01.dbf' SIZE 20M REUSE
UNDO TABLESPACE undotbs
DATAFILE '/opt/oracle/oradata/otokaze/undotbs01.dbf' SIZE 200M REUSE AUTOEXTEND ON MAXSIZE UNLIMITED
/
@?/rdbms/admin/catalog.sql 
@?/rdbms/admin/catproc.sql
@?/sqlplus/admin/pupbld.sql
exit
EOF

## 删库
1. dbca删库
2. 手动删库
select status from v$instance;  # 发现为open状态，需要改为mount状态
alter database close;
alter system enable restricted session;
drop database;

find /opt/oracle/ -iregex '.*your_db.*' -exec rm -fr {} \;
/etc/oratab 删除记录
</script></code></pre>

## rman备份工具
<pre><code class="language-bash line-numbers"><script type="text/plain">mkdir -p /opt/oracle/backup/control
chown oracle:oinstall ...
chmod 775 ...

rman target sys/123456@orcl
configure channel device type disk format'/opt/oracle/backup/DB_%U';
configure controlfile autobackup on;
configure controlfile autobackup format for device type disk to '/opt/oracle/backup/control/cf_%F';
configure retention policy to recovery window of 7 days;
show all;

## 全库压缩备份
backup as compressed backupset full database format '/opt/oracle/backup/full_bk1_%u%p%s.rmn' include current controlfile plus archivelog format '/opt/oracle/backup/arch_bk1_%u%p%s.rmn' delete all input;

## 全库非压缩备份
backup full database format '/opt/oracle/backup/full_bk1_%u%p%s.rmn' include current controlfile plus archivelog format '/opt/oracle/backup/arch_bk1_%u%p%s.rmn' delete all input;

## 全库使用默认通道默认配置备份，同时删除备份过的归档日志
backup as compressed backupset full database include current controlfile plus archivelog delete all input;

一般选择压缩或非压缩

## 全库恢复
cd /opt/oracle/oradata/orcl/ && mv xxx xxx.bak

sqlplus sys/123456 as sysdba
shutdown abort;
startup; # 报错

rman target sys/123456
list backupset;
restore database;
recover database;
alter database open;

## 表空间备份
backup tablespace zfl;

## 表空间恢复
rman target sys/123456
restore tablespace zfl;
recover tablespace zfl;

## 异常处理
# oracle datafile 数据文件损坏或丢失
shutdown abort;
startup mount;
alter database datafile '/opt/oracle/oradata/orcl/xcboa1.dbf' offline drop;
alter database open;
drop tablespace xcboa including contents;
重建表空间
</script></code></pre>
