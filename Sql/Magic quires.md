### ==هتلي كل الجداول الي فيها returnDate== 



  select * from INFORMATION_SCHEMA.COLUMNS where COLUMN_NAME = 'returnDate'

---
### ==query to know tables==



declare @db_name nvarchar(255); declare @t_name nvarchar(255) = 'ProjectContract'; declare @sql nvarchar(max); declare @record_count int;

create table #db_record_counts (databasename nvarchar(255), recordcount int);

declare db_cursor cursor for select name from sys.databases where state = 0 and name not in ('master', 'tempdb', 'model', 'msdb');

open db_cursor;

fetch next from db_cursor into @db_name;

while @@fetch_status = 0 begin set @sql = 'if exists (select 1 from ' + quotename(@db_name) + '.information_schema.tables where table_name = ''' + @t_name + ''') begin if exists (select 1 from ' + quotename(@db_name) + '.dbo.' + @t_name + ') begin select @record_count = count(*) from ' + quotename(@db_name) + '.dbo.' + @t_name + '; insert into #db_record_counts (databasename, recordcount) values (''' + @db_name + ''', @record_count); end end';

```
begin try
    exec sp_executesql @sql, N'@record_count int output', @record_count output;
end try
begin catch
    print 'error in database ' + @db_name + ': ' + error_message();
end catch;

fetch next from db_cursor into @db_name;

```

end;

close db_cursor; deallocate db_cursor;

declare @final_sql nvarchar(max) = 'select databasename, recordcount from #db_record_counts order by recordcount desc;';

exec sp_executesql @final_sql;

drop table #db_record_counts;