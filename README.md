![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Use SQL To Programmatically Restore And Rename Databases
**Post Date: May 30, 2014**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>This logic will automatically restore a database with a new name. All you need to do is provide the database name you want and the path for the database backup file. There's some extra notations in the logic so you can see where you will need to uncomment a line in case to avoid physical name conflicts.
If you want to include some extra logic to make the backup before the restore logic runs, create/add something like this.</p>      


## SQL-Logic
```SQL
declare @cmd varchar(max)
set @cmd = 'backup database ['' +  @database_name + ''] to disk = ''' + @backup_file_name  + ''' with format'
exec    (@cmd)
```

<p>Here's the restore logic…</p>      


## SQL-Logic
```SQL
use master;
set nocount on
declare     @database_name          varchar (255)
declare     @backup_file_name       varchar (255)
declare     @create_restore_logic       varchar (max)
set     @database_name          = 'MyDatabaseName' -- Enter your database name here
set     @backup_file_name       = '\\MyNetworkShareLocation\MyDatabaseBackup.bak'  -- enter your backup file here
set     @create_restore_logic       = ''
select      @create_restore_logic       = @create_restore_logic + 
'
use master;
set nocount on
go
 
declare       @filelistonly        table
(
              logicalname          nvarchar     (128)
,             physicalname         nvarchar     (260)
,             [type]               char         (1)
,             filegroupname        nvarchar     (128)
,             size                 numeric      (20,0)
,             maxsize              numeric      (20,0)
,             fileid               bigint
,             createlsn            numeric      (25,0)
,             droplsn              numeric      (25,0)
,             uniqueid             uniqueidentifier
,             readonlylsn          numeric      (25,0)
,             readwritelsn         numeric      (25,0)
,             backupsizeinbytes    bigint
,             sourceblocksize      int
,             filegroupid          int
,             loggroupguid         uniqueidentifier
,             differentialbaselsn  numeric      (25,0)
,             differentialbaseguid uniqueidentifier
,             isreadonl            bit
,             ispresent            bit
,             tdethumbprint        varbinary    (32)
)
insert into 
              @filelistonly        exec         (''restore filelistonly from disk = ''''' + @backup_file_name + ''''''')
declare       @restore_line0       varchar      (255)
declare       @restore_line1       varchar      (255)
declare       @restore_line2       varchar      (255)
declare       @stats               varchar      (255)
declare       @move_files          varchar      (max)
set           @restore_line0       = (''use master; '')
set           @restore_line1       = (''exec master..sp_killallprocessindb ''''' + @database_name + ''''';'')
set           @restore_line2       = (''restore database [' + @database_name + '] from disk = ''''' + @backup_file_name + ''''' with replace, recovery, '')
set           @stats               = (''stats = 20;'')
set           @move_files          = ''''
select        @move_files          = @move_files + ''move '''''' + logicalname + '''''' to '''''' + physicalname + '''''','' + char(10) from @filelistonly order by fileid asc
--  if you are renaming the database you will need to comment out the line above, and uncomment the line below to avoid physicalname conflicts.
--  select        @move_files          = @move_files + ''move '''''' + logicalname + '''''' to '''''' + reverse(right(reverse(physicalname), len(physicalname) - charindex(''\'',reverse(physicalname),1) + 1)) + ' + @database_name + ' + ''_'' + fileid +  right(physicalname, 4)  + '''''','' + char(10) from @filelistonly order by fileid asc
exec
(
              @restore_line0
+             @restore_line1
+             @restore_line2
+             @move_files
+             @stats
)
go
'
 
--  uncomment the execution line below to run this logic.
--  exec    (@create_restore_logic) 
select      (@create_restore_logic) for xml path (''), type
```

<p>By the way… Just found a bug… So here's the fix.
This will correct the 'name' & 'integer' error you might get when running this logic.
Replace the old segment with this new segment: </p>    


## SQL-Logic
```SQL
1
2
3	--  if you are renaming the database you will need to comment out the line above, and uncomment the line below to avoid physicalname conflicts.
-- select        @move_files          = @move_files + ''move '''''' + logicalname + '''''' to '''''' + reverse(right(reverse(physicalname), len(physicalname) - charindex(''\'',reverse(physicalname),1) + 1)) + 
''' + @database_name + ''' + ''_'' + cast(fileid as varchar(4)) +  right(physicalname, 4)  + '''''','' + char(10) from @filelistonly order by fileid asc
```

Hope this is useful. 


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

     
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

