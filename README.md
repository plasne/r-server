# RServer on HDInsight

## Connect to SQL Server

You can use the ScaleR Functions: https://msdn.microsoft.com/en-us/library/5f3c9864-9c75-4688-947d-0940045b2671?f=255&MSPPError=-2147217396#Functions.
 
However, in order for those to work with R Server in HDInsight you must deploy the Microsoft ODBC Driver for SQL Server (https://docs.microsoft.com/en-us/sql/connect/odbc/linux/installing-the-microsoft-odbc-driver-for-sql-server-on-linux) on the nodes. The way I was able to get that to work was to deploy the following script as a Script Action:
 
```sh
curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
curl https://packages.microsoft.com/config/ubuntu/16.04/prod.list > /etc/apt/sources.list.d/mssql-release.list
sudo apt-get update
sudo ACCEPT_EULA=Y apt-get --assume-yes install msodbcsql
sudo ACCEPT_EULA=Y apt-get --assume-yes install mssql-tools
sudo ACCEPT_EULA=y apt-get --assume-yes install unixodbc-dev
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bash_profile
echo 'export PATH="$PATH:/opt/mssql-tools/bin"' >> ~/.bashrc
source ~/.bashrc
```

Once I had done that, I was able to do this:

```R
conn <- "DRIVER=ODBC Driver 13 for SQL Server;SERVER=tcp:plasne-db01.database.windows.net,1433;DATABASE=pelasne-database;UID=plasne;PWD=***;"
data <- RxSqlServerData(sqlQuery = “SELECT * FROM dbo.dim_colors”, connectionString = conn)
head(data)
```
