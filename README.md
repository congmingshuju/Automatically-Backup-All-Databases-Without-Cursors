![CLEVER DATA GIT REPO](https://github.com/congmingshuju/git-resources/blob/master/images/0-clever-data-github.png "李聪明 数据")


# 自动备份没有光标的所有数据库
### Automatically Backup All Databases Without Cursors
**发布-日期:  2014年10月03日 (评论)**

![Automate Backup Jobs](images/process-automation-on-the-metal-gears-l.jpg?raw=true "SQL Automation")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL-Logic](#Logic)
- [Build Quality](#Build-Quality)
- [Author](#Author)
- [License](#License) 


## 中文
以下SQL逻辑将执行以下操作：
1.	在网络共享上自动创建备份文件夹结构（你只需提供共享名称）。
2.	自动删除旧的完整数据库备份（如果存在）（当前保留为2天，可随时修改）。
3.	在所有数据库上自动执行完整数据库备份。
4.	自动删除旧的事务日志备份（如果存在）（当前保留为2天。随意修改）
5.	在所有数据库上自动执行交易日志备份。
6.	自动将交易日志数据文件缩小到最低8kb的增量。
7.	跨所有数据库自动执行DBCC CheckDB。
8.	自动将数据文件缩小到最低8kb的增量。

我建议在以下构造中复制逻辑并创建3个：
### Job1:
备份全部数据库
步骤1.删除旧的完整备份
步骤2.运行完整数据库备份
### Job2:
备份TLOG所有数据库
步骤1.删除旧的事务日志备份
步骤2.运行事务日志备份
步骤3.压缩事务日志数据文件
### Job3:
数据库维护
步骤1.运行DBCC CheckDB
步骤2.压缩数据文件

你需要确保SQL服务帐户在网络共享上具有相应的权限才能创建文件夹和文件。

希望以上可以帮助到你！


## English
The following SQL logic will perform the following actions.
1. Automatically create a backup folder structure on a network share ( you just provide the share name )
2. Automatically delete old Full Database Backups if they exist ( current retention is 2 days. feel free to modify )
3. Automatically perform Full Database Backups on all Databases.
4. Automatically delete old Transaction Log Backups if they exist ( current retention is 2 days. feel free to modify )
5. Automatically perform Transaction Log Backups on all Databases.
6. Automatically shrink Transaction Log Data Files down to the lowest 8kb increment.
7. Automatically perform DBCC CheckDB across all Databases.
8. Automatically shrink Data Files down to the lowest 8kb increment.

> I recommend copying the logic and creating 3  in the following construct:

### Job1:
DATABASE BACKUP FULL ALL DATABASES
Step 1. Delete old full backups.
Step 2. Run Full Database Backups
### Job2:
DATABASE BACKUP TLOG ALL DATABASES
Step 1. Delete old transaction log backups.
Step 2. Run Transaction Log Backups.
Step 3. Shrink Transaction Log Data Files.
### Job3:
DATABASE MAINTENANCE
Step 1. Run DBCC CheckDB
Step 2. Shrink Data Files.

You will need to ensure the SQL Service Account has the appropriate rights on the network share to create Folders and Files.
Hope this is useful.


---
## Logic
```SQL
set nocount on
 
-- create backup folder structure   创建备份文件夹结构
declare @backup_path varchar(255)
declare @server_name varchar(255)
declare @complete_path varchar(255)
declare @create_dbfolders varchar(max)
set @backup_path = '\\MyNetworkShare\SQLBackups\'
set @server_name = ( select replace(cast(serverproperty('servername') as varchar(255)), '\', '--') )
set @complete_path = ( @backup_path + @server_name )
exec master..xp_create_subdir @complete_path
 
-- full database backups to network share  完整数据库备份到网络共享
declare @backup_all_databases varchar(max)
declare @dayname varchar(255)
declare @timestamp varchar(255)
declare @type varchar(255)
set @type = 'FULL'
set @dayname = ( select DATENAME(dw, GETDATE()) )
set @timestamp = ( select replace(replace(replace(REPLACE(CONVERT(char, getdate()), ':', '-'), 'AM', 'am'), 'PM' ,'pm'), ' ', ''))
set @backup_all_databases = ''
select @backup_all_databases = @backup_all_databases + 
'backup database [' + name + '] to disk = ''' + @complete_path + '\' + @dayname + ' ' + @timestamp + ' ' + @type + ' ' + name + '.bak' + ''';' + CHAR(10)
from sys.databases where name not in ('tempdb') and state_desc = 'online'
exec (@backup_all_databases)
go
 
-- create backup folder structure       创建备份文件夹结构
declare @backup_path varchar(255)
declare @server_name varchar(255)
declare @complete_path varchar(255)
declare @create_dbfolders varchar(max)
set @backup_path = '\\MyNetworkShare\SQLBackups\'
set @server_name = ( select replace(cast(serverproperty('servername') as varchar(255)), '\', '--') )
set @complete_path = ( @backup_path + @server_name )
exec master..xp_create_subdir @complete_path
 
-- delete old full backups       删除旧的完整备份
declare @olderthan datetime
declare @deletepath varchar(255)
set @deletepath = @complete_path + '\'
set @olderthan = dateadd(day, -0, getdate());
execute master.dbo.xp_delete_file 0, @deletepath, 'bak', @OlderThan;
go
 
-- create backup folder structure     创建备份文件夹结构
declare @backup_path varchar(255)
declare @server_name varchar(255)
declare @complete_path varchar(255)
declare @create_dbfolders varchar(max)
set @backup_path = '\\MyNetworkShare\SQLBackups\'
set @server_name = ( select replace(cast(serverproperty('servername') as varchar(255)), '\', '--') )
set @complete_path = ( @backup_path + @server_name )
exec master..xp_create_subdir @complete_path
 
-- transaction log backups to network share  事务日志备份到网络共享
```SQL use master;
set nocount on
set quoted_identifier off
declare @backup_all_logs varchar(max)
declare @dayname varchar(255)
declare @timestamp varchar(255)
declare @type varchar(255)
set @type = 'TLOG'
set @dayname = ( select DATENAME(dw, GETDATE()) )
set @timestamp = ( select replace(replace(replace(REPLACE(CONVERT(char, getdate()), ':', '-'), 'AM', 'am'), 'PM' ,'pm'), ' ', ''))
set @backup_all_logs = ''
select @backup_all_logs = @backup_all_logs + 
'backup log [' + name + '] to disk = ''' + @complete_path + '\' + @dayname + ' ' + @timestamp + ' ' + @type + ' ' + name + '.trn' + ''';' + CHAR(10)
from sys.databases where name not in ('master', 'model', 'msdb', 'tempdb') and state_desc = 'online' and recovery_model_desc = 'FULL'
exec (@backup_all_logs)
go
 
-- shrink transaction log backups    压缩事务日志备份
declare @shrink_files varchar(max)
set @shrink_files = ''
select @shrink_files = @shrink_files +
'use [' + name + ']; dbcc shrinkfile(2,8);' + char(10)
from sys.databases where name not in ('tempdb') and state_desc = 'online' and recovery_model_desc = 'FULL'
exec (@shrink_files)
 
-- create backup folder structure   创建备份文件夹结构
declare @backup_path varchar(255)
declare @server_name varchar(255)
declare @complete_path varchar(255)
declare @create_dbfolders varchar(max)
set @backup_path = '\\MyNetworkShare\SQLBackups\'
set @server_name = ( select replace(cast(serverproperty('servername') as varchar(255)), '\', '--') )
set @complete_path = ( @backup_path + @server_name )
exec master..xp_create_subdir @complete_path
 
-- delete old transaction log backups   删除旧的事务日志备份
declare @olderthan datetime
declare @deletepath varchar(255)
set @deletepath = @complete_path + '\'
set @olderthan = dateadd(day, -0, getdate());
execute master.dbo.xp_delete_file 0, @deletepath, 'trn', @OlderThan;
go

```

[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Quality 
| [![Build status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg=true)](https://ci.appveyor.com/project/tygerbytes/resourcefitness) | [![Coveralls](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?branch=master)](https://coveralls.io/github/tygerbytes/ResourceFitness?branch=master) | [![nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](https://www.nuget.org/packages/TW.Resfit.Core/) |
|-|-|-|

>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](https://ci.appveyor.com/project/tygerbytes/resourcefitness/history)


## Author

- **李聪明 数据 Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明数据-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://github.com/congmingshuju/git-resources/blob/master/images/clever-data-gist-z5.png "李聪明 数据")

