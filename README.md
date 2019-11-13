![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 自动为SQL Server添加防火墙规则
### Automatically Add FireWall Rules For SQL Server
**发布-日期:  2016年11月28日 (评论)**


![Add Firewall Rules With SQL](images/automatically_add_firewall_rules_for_sql.jpg?raw=true "Automatically Add Firewall Rules")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
自动添加SQL防火墙规则：
这是一些用于检查创建的最新SQL instance，并自动添加端口防火墙规则的SQL逻辑。这不是很难实现自动化，任何有助于自动化的事情总是有很大的帮助，当然也鼓励设置环境的一致性。
这将会运行一般删除语句，因此你不会遇到因为防火墙规则名称重复的问题，然后运行addrule语句。
创建规则名称请按下列规则：
SQL `[MyInstanceName]` Port `[MyPort]`
就是如此简单。
请看如下例子：



## English
Automatically Add SQL Firewall Rules
Here’s some SQL logic that checks for the most recent SQL instance created, and automatically adds a port firewall rule. This isn’t too terribly hard to do on it’s own, but anything to help automate is always a big help, and of course also encourages consistency, and uniformity in setting up environments.
This will run a general remove statement so you don’t run into duplicate issues for your firewall rule names, then it runs the addrule statement.
The rule name will be created with the following Convention.
SQL `[MyInstanceName]` Port `[MyPort]`
Thats it. 


---
## Logic
```SQL
use master;
set nocount on
   
declare @sql_instances  table
(   
    [rootkey]   varchar(255)
,   [value]     varchar(255)
)
insert into @sql_instances 
exec master.dbo.xp_instance_regenumvalues 
    @rootkey    = N'HKEY_LOCAL_MACHINE'
,   @key        = N'SOFTWARE\\Microsoft\\Microsoft SQL Server\\Instance Names\\SQL';
   
declare db_cursor   cursor for select upper([rootkey]), upper([value]) from @sql_instances
declare @instance_name  varchar(255)
declare @instance_path  varchar(255)
open    db_cursor;
fetch next from db_cursor into @instance_name, @instance_path
    while @@fetch_status = 0  
        begin
            declare @port_table table
            (
        [id]        int identity(1,1)
            ,   [Instance]  varchar(255)
            ,   [Port]      int
            )
            declare @port   varchar(50)
            declare @key    varchar(255) = 'software\microsoft\microsoft sql server\' + @instance_path + '\mssqlserver\supersocketnetlib\tcp\ipall'
            exec master..xp_regread
                @rootkey    = 'hkey_local_machine'
            ,   @key        = @key
            ,   @value_name = 'tcpdynamicports'
            ,   @value      = @port output
              
            insert into @port_table ([instance], [port])
            select
                'Instance'  = @instance_name
            ,   'Port'  = isnull(convert(varchar(10), @port), 1433)
            fetch next from db_cursor into @instance_name, @instance_path
        end;
    close db_cursor
deallocate db_cursor;
 
declare @add_port_rule  varchar(255) = (select top 1'netsh advfirewall firewall add rule name="SQL '        + [instance] + '  Port ' + cast([port] as varchar) + '" dir=in action=allow protocol=tcp localport=' + cast([port] as varchar) + '' from @port_table order by [id] desc)
declare @rem_port_rule  varchar(255) = (select top 1'netsh advfirewall firewall delete rule name="SQL '     + [instance] + '  Port ' + cast([port] as varchar) + '"' from @port_table order by [id] desc)
 
exec    master..xp_cmdshell @rem_port_rule  -- if already exists it will remove port rule before creating the new rule.
exec    master..xp_cmdshell @add_port_rule


```

> If you need to verify; here’s a quick Powershell script to return all the Firewall rules with the letters “SQL”


```Powershell
get-netfirewallrule `
| where { $_.enabled -eq 'true' -and $_.direction -eq 'inbound' -and $_.'displayname' -like "*sql*"}`
| select displayname, enabled, action, direction `
| out-string -width 98

```

[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

