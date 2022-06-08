# Linux Cluster Monitoring Agent

## Introduction
The Linux Cluster Monitoring Agent is a tool that allows users to monitor nodes connected in a Linux cluster by tracking hardware specifications of 
each node and storing the data in a relational database. 


## Architecture and Design


## Databases and Tables
The database `host_data` consists of two tables, `host_info` and `host_usage`. `host_info` stores hardware specifications and insert it into the database. 
It will be run only once at the installation time., and `host_usage` stores current host usage data and updates every minute by each node.

### host_info
- `id`: a unique identification number and primary key, autoincremented in PGSQL
- `host_name`: a full, unique name for the node 
- `cpu_number`: the number of CPUs
- `cpu_architecture`: a layout of the internals of the CPU (eg. x86_64)
- `cpu_model`: the full model name of the CPU
- `cpu_mhz`: a measurement of the transmission speed of the CPU (in mhz)
- `l2_cache`: level 2 cache memory, separate from the chip core (in KB)
- `total_mem`: total memory usage (in KB)
- `time_stamp`: date and time of data retrieval in UTC

### host_usage
- `host_id`: a unique identification number and foreign key, points to `id` from `host_info`
- `memory_free`: amount of memory free (in MB)
- `cpu_idle`: cpu idle percentage
- `cpu_kernel`: cpu kernel percentage
- `disk_io`: number of disk I/O
- `disk_available`: disk available (in MB)
- `time_stamp`: date and time of data retrieval in UTC


## Script Description

- [host_info.sh](host) collects the host hardware info and insert it into the database. It will be run only once at the installation time.

- [scripts/psql_docker.sh](host) collects the current host usage (CPU and Memory) and then insert into the database. It will be triggered by 
the crontab job every minute.

- [ddl.sql](host) is used to automate the creation of the `host_agent` database and the `host_info` and `host_usage` tables when 
`psql -h localhost -U postgres -d host_agent -f sql/ddl.sql` 

- [queries.sql](host)
contains SQL queries that helps the user to better keep track of/ manage the cluster better and also plan for future recourses.

## Usage
### 1. Database and Table Initialization
Before runnning the bash agent, the PostgreSQL instance has to be provisioned by creating and starting up a Docker container and postgres instance 
that provisioned for data base management.
`./scripts/psql_docker.sh start|stop|create [db_username][db_password]`

### 2. host_info.sh Usage
It will run the script once per node to insert the node's hardware specifications into the `host_info` table.
`./scripts/host_info.sh psql_host psql_port db_name psql_user psql_password`


### 3. host_usage.sh Usage
This script inserts a snapshot of the node's current resource usage into `host_usage` table and can bo run manually by the below code.
`./scripts/host_usage.sh psql_host psql_port db_name psql_user psql_password`

### 4. crontab Setup
A crontab is to create a repeatedly run the host_usage.sh script after a specified interval.

`crontab -e` <br />
`* * * * *  ./[path to host_usage.sh]/host_usage.sh hostname PSQL_PORT db_name db_username db_password > /tmp/host_usage.log`

## Improvements

- The crontab basically add the usage to database and keeps it, but it does not handle or delete the older data.
- The Script does not contains exception handling when connecting to the database.




