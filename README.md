# dbt-azuredw

dbt-azuredw is a custom adapter for [dbt](https://github.com/fishtown-analytics/dbt) that adds support for Azure SQL Data Warehouse.. pyodbc is used as the connection driver as that is what is [suggested by Microsoft](https://docs.microsoft.com/en-us/sql/connect/python/python-driver-for-sql-server). The adapter supports both windows auth, and specified user accounts.

dbt-azuredw is currently in a beta release.

## Connecting to Azure SQL Data Warehouse

## building your `profiles.yml`
Use the profiles.yml file included as a guide, updating with your creds. You can find all the creds you need under _Home > dbname (account/dbname) - Connection strings_ in Azure, along with the username and password for authentication. 

## Getting Started
1. Run this to keep your profiles.yml from tracking:

``` 
git update-index --skip-worktree profiles.yml
```

2. Update profiles.yml with your actual Azure Data Warehouse creds.
3. Build the docker image. From the repo root:

```
docker build . -t dbt-azure-dw
```

4. Run a bash shell in the container:

```
docker run -v $(PWD):/dbt_development/plugins -it dbt-azure-dw /bin/bash
```

you can then jump into `jaffle_shop (mssql)` and work on making it run against your ADW!  


**Sample profiles.yml**

```yaml

default:	
  target: dev	
  outputs:	
    dev:	
      type: azuredw	
      driver: 'ODBC Driver 17 for SQL Server'	
      host: account.database.windows.net	
      database: dbt_test	
      schema: foo	
      username: dbt_user	
      password: super_secret_dbt_password
      authentication: ActiveDirectoryPassword   
```

## Known Issues
- At this time dbt-azuredw supports only `table`, `view` and `incremental` materializations (no `ephemeral`)
- Only top-level (model) CTEs are supported, ie CTEs in macros are not supported (this is a sqlserver thing)



## Jaffle Shop

Fishtown Analytic's [jaffle shop](https://github.com/fishtown-analytics/jaffle_shop) package is currently unsupported by this adapter. At the time of this writing, jaffle shop uses the `using()` join, and `group by [ordinal]` notation which is not supported in T-SQL. An alternative version has been forked by the author of dbt-mssql [here](https://github.com/jacobm001/jaffle_shop_mssql).

## Creating indexes on post-hook

- To create a nonclustered index for a specific model, go to that model's SQL and add a `config` macro with a `pre-hook` and `post-hook` key/value pair.  
- Whenever you _create_nonclustered_index_ on a `post-hook`, we recommend you _drop_all_indexes_on_table_ on a `pre-hook`.  
- You can create more than one index on a model in the `post-hook` by submitting a bracketed list of _create_nonclustered_index_ macros. 
- See examples below.

### Macro Syntax

- _create_clustered_index_ takes one argument:
  - column - a quoted string that refers to the column you want to create a clustered index on
- _drop_all_indexes_on_table_ needs no arguments.
- _create_nonclustered_index_ takes two arguments:
  - columns - a list of quoted strings that refer to the column names you want to create the index on
  - includes - a list of quotes strings that refer to the column names that you want to include in the index look-ups.

### Create one index on a model

```jinja2
{{ 
    config({
      "pre-hook": "{{ drop_all_indexes_on_table() }}",
      "post-hook": "{{ create_nonclustered_index(columns = ['some_column'], includes = ['another_column']) }}"
    }) 
}}
```

### Create many indexes on a model 

```jinja2
{{ 
    config({
      "pre-hook": "{{ drop_all_indexes_on_table() }}",
      "post-hook": [
         "{{ create_clustered_index(column = 'some_table_key'}}",
         "{{ create_nonclustered_index(columns = ['some_column']) }}",
         "{{ create_nonclustered_index(columns = ['a_colmumn', 'the_column']) }}",
         "{{ create_nonclustered_index(columns = ['this_column', 'that_column'], includes = ['my_column', 'your_column']) }}"
	 ]
    }) 
}}
```
