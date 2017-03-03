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

## Publishing Web APIs of R code

1. You must enable Operationalization as per the instructions here: https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-hadoop-r-server-get-started; specifically the bottom part of the article entitled “Using Microsoft R Server Operationalization” and the following section to enable across the worker nodes.

2. The 12800 port doesn’t appear to be open through the public gateway, so you have to connect to the RServer from a system on the same VNET (deployed in Azure or use Point-to-Site VPN) or use SSH port forward tunneling as described here: https://docs.microsoft.com/en-us/azure/hdinsight/hdinsight-hadoop-r-server-get-started; specifically the “RServer Cluster not set up on virtual network” portion.

3. You can test with the service as described here: https://msdn.microsoft.com/en-us/microsoft-r/operationalize/data-scientist-get-started?f=255&MSPPError=-2147217396.

  * Note what it says near the bottom of the article when working with a remote R session you must pause() and resume() in order to publishService(). That was my experience as well, but for some reason it only said that at the very bottom of this page and not on any of the other samples.
