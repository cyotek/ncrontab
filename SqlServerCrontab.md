This article shows how to use NCrontab to generate occurrences of a crontab-style schedule as a table in SQL Server (2005 or later), which can then be used in queries and especially joins to do interesting things.



## Writing the Table-Valued Function in C# ##

The `SqlCrontab` class below implements a managed table-value function called `GetOccurrences` that relies NCrontab for its implementation. Given a crontab expression and a range of time, `GetOccurrences` will yield occurrences as a table with a single column typed as `DATETIME`:

```
namespace NCrontab.Samples
{
    #region Imports

    using System;
    using System.Collections;
    using System.Data.SqlTypes;
    using Microsoft.SqlServer.Server;
    using NCrontab;

    #endregion

    public static class SqlCrontab
    {
        [SqlFunction(FillRowMethodName = "FillOccurrenceRow", IsDeterministic = true, IsPrecise = true)]
        public static IEnumerable GetOccurrences(SqlString expression, SqlDateTime start, SqlDateTime end)
        {
            if (expression.IsNull || start.IsNull || end.IsNull)
                return new DateTime[0];

            try
            {
                var schedule = CrontabSchedule.Parse(expression.Value);
                return schedule.GetNextOccurrences(start.Value, end.Value);
            }
            catch (CrontabException)
            {
                return new DateTime[0];
            }
        }

        public static void FillOccurrenceRow(object obj, out SqlDateTime time)
        {
            time = (DateTime) obj;
        }
    }
}
```

For more information on how CLR-based table-value functions are implemented and work, see [CLR Table-Valued Functions](http://msdn.microsoft.com/en-us/library/ms131103.aspx) topic in [SQL Server Books Online](http://msdn.microsoft.com/en-us/library/bb545450.aspx).

Assuming you are in the same directory as where the above sample code resides in a C# source file, compile a library using the C# compiler as shown here:


```
csc /t:library /r:NCrontab.dll /out:NCrontab.Samples.dll *.cs
```

## Registering with SQL Server ##

Next, in a SQL Server database where you wish to use this function, execute the following SQL batch to register the assembly and function.

```
CREATE ASSEMBLY [NCrontab.Samples] 
FROM 'NCrontab.Samples.dll' -- supply the path before the file name here
GO

CREATE FUNCTION CrontabSchedule(
    @Expression NVARCHAR(100), 
    @Start DATETIME, 
    @End DATETIME)
RETURNS TABLE (
    [Occurrence] DATETIME)
AS 
EXTERNAL NAME [NCrontab.Samples].[NCrontab.Samples.SqlCrontab].[GetOccurrences]
GO
```

SQL Server CLR integration is usually turned off by default so the above `CREATE ASSEMBLY` and `CREATE FUNCTION` will work but the function cannot be used until CLR integration is enabled. Unless it is already enabled, use the following batch to accomplish this:

```
sp_configure 'clr enabled', 1;
GO
RECONFIGURE
GO
```

For more information, see [Enabling CLR Integration](http://msdn.microsoft.com/en-us/library/ms131048.aspx) topic in [SQL Server Books Online](http://msdn.microsoft.com/en-us/library/bb545450.aspx).

## Crontab Schedule Occurrences in SQL Queries ##

Now, the new `dbo.CrontabSchedule` can be used in a query. For example, the following query will yield all occurrences of February 29th between 1900 and 2000:

```
SELECT * FROM dbo.CrontabSchedule('0 12 29 feb *', '1900-1-1', '2000-1-1')
GO
```

The result of the above query should look like this:

```
1904-02-29 12:00:00.000
1908-02-29 12:00:00.000
1912-02-29 12:00:00.000
1916-02-29 12:00:00.000
1920-02-29 12:00:00.000
1924-02-29 12:00:00.000
1928-02-29 12:00:00.000
1932-02-29 12:00:00.000
1936-02-29 12:00:00.000
1940-02-29 12:00:00.000
1944-02-29 12:00:00.000
1948-02-29 12:00:00.000
1952-02-29 12:00:00.000
1956-02-29 12:00:00.000
1960-02-29 12:00:00.000
1964-02-29 12:00:00.000
1968-02-29 12:00:00.000
1972-02-29 12:00:00.000
1976-02-29 12:00:00.000
1980-02-29 12:00:00.000
1984-02-29 12:00:00.000
1988-02-29 12:00:00.000
1992-02-29 12:00:00.000
1996-02-29 12:00:00.000
```

## Multiple Schedule Occurrences with Unions ##

Suppose you want to have a schedule that yields midnight on 1st, 2nd and 3d of February and then on 4th, 5th or 6th of March. Unfortunately, you cannot express this in a single crontab expression. What you can do, however, is use two expressions and `UNION` their results as shown in the example below :

```
SELECT * FROM dbo.CrontabSchedule('0 12 1,2,3 feb *', '2000-1-1', '2005-1-1')
UNION
SELECT * FROM dbo.CrontabSchedule('0 12 4,5,6 mar *', '2000-1-1', '2005-1-1')
ORDER BY 1
GO
```

The result of the above query should look like this:

```
2000-02-01 12:00:00.000
2000-02-02 12:00:00.000
2000-02-03 12:00:00.000
2000-03-04 12:00:00.000
2000-03-05 12:00:00.000
2000-03-06 12:00:00.000
2001-02-01 12:00:00.000
2001-02-02 12:00:00.000
2001-02-03 12:00:00.000
2001-03-04 12:00:00.000
2001-03-05 12:00:00.000
2001-03-06 12:00:00.000
2002-02-01 12:00:00.000
2002-02-02 12:00:00.000
2002-02-03 12:00:00.000
2002-03-04 12:00:00.000
2002-03-05 12:00:00.000
2002-03-06 12:00:00.000
2003-02-01 12:00:00.000
2003-02-02 12:00:00.000
2003-02-03 12:00:00.000
2003-03-04 12:00:00.000
2003-03-05 12:00:00.000
2003-03-06 12:00:00.000
2004-02-01 12:00:00.000
2004-02-02 12:00:00.000
2004-02-03 12:00:00.000
2004-03-04 12:00:00.000
2004-03-05 12:00:00.000
2004-03-06 12:00:00.000
```

## Conclusion ##

Now go and get creative. :)