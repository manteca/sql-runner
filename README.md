## LEROI SQL runner

The LEROI SQL runner has three basic functionalities

* executing SQL code in a specific order
```
runner --execute {RUNNER_FILE_1}, {RUNNER_FILE_2} ..
```
* executing SQL code in a specific order, in staging mode (on test schema, 
tables and data)
```
runner --staging {RUNNER_FILE_1}, {RUNNER_FILE_2} ..
```

* quickly testing SQL code through temporary creation of views
```
runner --test {RUNNER_FILE_1}, {RUNNER_FILE_2} ..
```
* plotting of a dependency graph
```
runner --deps
```

An alias for the `runner` command is `sqlrunner`, for legacy purposes.

Using `run_sql` will run in interactive mode. `run_sql /path/to/config.json`

The supported databases are Redshift, Snowflake and Postgres.

### Installation

SQL-Runner has the following optional dependencies that have to be mentioned when needed, during the installation process with pip:
* `azuredwh` - for work with Azure SQL Data Warehouse
* `snowflake` - for working with Snowflake DB
* `redshift` - for working with AWS Redshift
* `s3` - for enabling AWS S3 API access (for saving dependencies SVG graph)

Additionally for Azure DWH, it's required to install the [Microsoft ODBC Driver](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-2017). For Ubuntu 18.04 this is sufficient:
```sh
# In case any of these gest stuck, simply run `sudo su` once, to cache the password, then exit using Ctrl+D
curl https://packages.microsoft.com/keys/microsoft.asc | sudo apt-key add -
curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list | sudo tee /etc/apt/sources.list.d/mssql-release.list > /dev/null
sudo apt-get update
sudo ACCEPT_EULA=Y apt-get install msodbcsql17
sudo apt-get install unixodbc-dev
```

Another dependency is graphviz:
```sh
sudo apt install graphviz
```

It is highly recommend it to install it in a virtual environment.

To create a virtual environment, run this:
```sh
sudo apt-get install python3-virtualenv
python3 -m virtualenv -p python3 venv
```

To install in a virtual environment, run this:
```sh
source venv/bin/activate
# Install with dependencies, ex. s3 and azuredwh
pip install git+https://github.com/leroi-marketing/sql-runner.git#egg=sql-runner[azuredwh]
```

But if you really want to install it globally, run this:
```sh
sudo apt install python3-pip
# Install with dependencies, ex. s3 and azuredwh
sudo pip install git+https://github.com/leroi-marketing/sql-runner.git#egg=sql-runner[azuredwh]
```

### Configuration
Two configuration files are needed to use the sqlrunner.
* A config.json file that specifies all the necessary configuration variables. The default path is `auth/config.json` relative to the directory that this is run from.
```
{
   "sql_path": "{PATH}",
    "database_type": "{SNOWFLAKE} OR {REDSHIFT} OR {POSTGRES}",
    "auth": {
    "user": "{USERNAME}",
    "password": "{PASSWORD}",
    "account": "{SNOWFLAKE_ACCOUNT}",
    "database": "{SNOWFLAKE_DATABASE}",
    "dbname": "{POSTGRES_DATABASE} OR {REDSHIFT_DATABASE}",
    "host": "{POSTGRES_HOSTNAME} OR {REDSHIFT_HOSTNAME}",
    "port": "{POSTGRES_PORT} OR {REDSHIFT_PORT}"
     
   },
    "deps_schema": "{DEPENDENCY_SCHEMA_NAME}",
    "test_schema_prefix" : "{PREFIX_}",
    "exclude_dependencies": "('EXCLUDED_SCHEMA_1', 'EXCLUDED_SCHEMA_2' ..)",
    "graphviz_path": "{GRAPHVIZ_PATH_FOR_WINDOWS}"
}
```
* One or more csv files specifying the name of the the tables and views and their respective schemas.
 ```
 {SCHEMA_1};{SQL_FILENAME_1};e
 {SCHEMA_1};{SQL_FILENAME_2};e
 {SCHEMA_1};{SQL_FILENAME_3};e
 {SCHEMA_2};{SQL_FILENAME_4};e
 {SCHEMA_3};{SQL_FILENAME_5};e
 ..
 ```
Per schema one directory is expected. The name of the SQL files should correspond to thename of the respective table or view. The last columns specifies the desired action.
 ```
 e: execute the query
 t: create table
 v: create view
 m: materialize view
 test: run assertions on query result
 ```

### Development

To set up dependencies locally for development:
```sh
# Install virtualenv (if your default python is python2, specify also `-p python3`)
python3 -m virtualenv -p python3 venv
source venv/bin/activate
pip install -e .[azuredwh] # and other optional dependencies

# Run local (non-build) version:
python debug.py [arg1 arg2 ...]
```
