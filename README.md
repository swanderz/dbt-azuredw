# dbt-azuredw

dbt-azuredw is a custom adapter for [dbt](https://github.com/fishtown-analytics/dbt) that adds support for Azure SQL Data Warehouse.. pyodbc is used as the connection driver as that is what is [suggested by Microsoft](https://docs.microsoft.com/en-us/sql/connect/python/python-driver-for-sql-server). The adapter supports both windows auth, and specified user accounts.

dbt-azuredw is currently in a beta release.

## Connecting to Azure SQL Data Warehouse

Your user profile (located in `~/.dbt/profile`) will need an appropriate entry for your package. 

Required parameters are:

- driver
- host
- database
- schema
- one of the login options:
  - SQL Data Warehouse authentication
    - username
    - password
  - Windows Login
    - windows_login: true

**Example profile:**

The example below configures a seperate dev and prod environment for the package, _foo_. You will likely need to alter the `driver` variable to match whatever is installed on your system. In this example, I'm using version 17, which is the newest on my system. If you have something else on your system, it should work as well.

```yaml
foo:
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
```
