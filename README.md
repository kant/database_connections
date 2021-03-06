Database Connections
================

## Connecting to Databases from R

> Patterned after <http://db.rstudio.com/>

This walkthrough is for MacOS X users. Windows users may be able to
achieve similar results through the use of
[Chocolatey](https://chocolatey.org/), though research into the specific
packages under Windows that match what homebrew installs is required.

These connectivity examples use DBI-compatible interfaces. DBI is a
database interface specification that allows any compliant tool to
communicate with databases without knowledge of the underlying database
technology. Using DBI tooling allows both direct SQL queries through raw
connections as well as tight integration with the tidyverse through the
spiffy `dbplyr` package.

Database drivers must be written with DBI interfaces. Both native
drivers and the generic (DBI-compliant) odbc connections are
demonstrated. For Oracle and MS SQL Server, odbc is the only (free)
connectivity option.

If you are only connecting to a single database type, the driver you
choose is largely a matter of personal preference. If connecting to
multiple different database types, using odbc will provide a more
consistent interface. Connections made using odbc will appear in the
connections tab of the RStudio IDE.

## Install ODBC and Database Drivers

``` shell
# install base odbc framework
brew install unixodbc

# Microsoft SQL Server, enabling it for use with odbc
brew install freetds --with-unixodbc
  
# PostgreSQL ODBC Drivers
brew install psqlodbc

# Maria/MySQL ODBC drivers
brew install mariadb-connector-c
brew install mariadb-connector-odbc

# SQLite ODBC Drivers
brew install sqliteodbc
```

### Oracle Client Install

The Oracle client install is complicated. 😦

Download the Basic client and the ODBC drivers via  
<http://www.oracle.com/technetwork/topics/intel-macsoft-096467.html>. A
(free) Oracle account is required. Place both ZIP files in your
\~/Downloads
directory.

``` bash
sudo unzip -o ~/Downloads/instantclient-basic-macos.x64-12.2.0.1.0-2.zip -d /opt/ora12
sudo unzip -o ~/Downloads/instantclient-odbc-macos.x64-12.2.0.1.0-2.zip -d /opt/ora12

# patch the Oracle ODBC install script to fix an incorrectly specified library
# extension (.so -> .dylib)
sudo patch ./oracle_odbc_update.patch /opt/ora12/instantclient_12_2/odbc_update_ini.sh > patch.diff
sudo /opt/ora12/instantclient_12_2/odbc_update_ini.sh /usr/local

ln -s `pwd`/libclntsh.dylib.12.1 /usr/local/lib
ln -s `pwd`/libclntshcore.dylib.12.1 /usr/local/lib
```

### Configure ODBC Drivers

Set `/usr/local/etc/odbcinst.ini` to the following to allow easy
refernce to the database drivers by name rather than by full file path:

    [SQLite]
    Driver = /usr/local/lib/libsqlite3odbc.dylib
    
    [PostgreSQL]
    Driver = /usr/local/lib/psqlodbcw.so
    
    [MySQL]
    Driver = /usr/local/lib/libmysqlclient.dylib
    
    [MariaSQL]
    Driver = /usr/local/lib/libmaodbc.dylib
    
    [SQL Server]
    Driver = /usr/local/lib/libtdsodbc.so
    
    [Oracle 12c]
    Description     = Oracle ODBC driver for Oracle 12c
    Driver          = /usr/local/lib/libsqora.dylib.12.2
    Setup           =
    FileUsage       =
    CPTimeout       =
    CPReuse         =

## Configure R packages

Install all the relevant R packages. We use DBI as the primary database
interface, with odbc for ODBC connectivity. Native drivers for
PostgreSQL, MariaDB/MySQL, and SQLite are also installed. In production,
you can choose either native or ODBC drivers as you see fit.

``` r
install.packages("DBI")         # DBI framework
install.packages("odbc")        # odbc framework

# Native drivers
install.packages("RPostgres")
install.packages("RMariaDB")    # covers MariaDB and MySQL
install.packages("RSQLite")
```
