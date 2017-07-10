---
title: Loader Instructions
category: advanced
---

# Loader instructions

## What is Loader?

Developed by PingCAP, Loader is a data import tool and it imports data to TiDB and MySQL.

[Download the Binary](http://download.pingcap.org/tidb-enterprise-tools-latest-linux-amd64.tar.gz)

## Why did we develop Loader?

Since tools like mysqldump will take us days to migrate massive amounts of data, we used the mydumper/myloader suite of Percona to multi-thread export and import data. During the process, we found that mydumper works well. However, as myloader lacks functions of error retry and savepoint, it is inconvenient for us to use. Therefore, we developed loader, which reads the output data files of mydumper and imports data to TiDB/MySQL through mysql protocol.

## What can Loader do?

+ Multi-thread import data

+ Support mydumper data format

+ Support error retry

+ Support savepoint

+ Improve the speed of importing data through system variable

## Usage

### Parameter description
```
  -L string: the log level setting. It can be set as debug, info, warn, error, fatal (default: "info")
  
  -P int: the port of TiDB/MySQL (default: 4000)
  
  -d string: the storage directory of data that need to import (default: "./")
  
  -h string: the host of TiDB/MySQL (default: "127.0.0.1")
  
  -checkpoint-schema string: the database name of checkpoint. In the execution process, loader will constantly update this database. After recovering from an interruption, loader will get the process of the last run through this database. (default: "tidb_loader")
  
  -skip-unique-check: whether to skip the unique index check, 0 means no while 1 means yes (can improve the speed of importing data).
  Note: Only when you import data to TiDB can you open this option (default: 1)
  
  -p string: the account and password of TiDB/MySQL
  
  -pprof-addr string: the pprof address of Loader. It tunes the perfomance of Loader (default: ":10084")
  
  -q int: the number of insert statement that included in each transaction during the import process (default: 1. By default, the size of each insert statement of sql exported by mydumper is 1MB, including many rows of data.)
  
  -t int: the number of thread (default: 4)
  
  -u string: the user name of TiDB/MySQL (default: "root")
```

### Configuration file

Apart from command line parameters, you can also use configuration files. The format is shown as below:

```toml
# Loader log level
log-level = "info"

# Loader log file
log-file = ""

# Directory of the dump to import
dir = "./"

# Loader pprof addr
pprof-addr = ":10084"

# We saved checkpoint data to tidb, which schema name is defined here.
checkpoint-schema = "tidb_loader"

# Number of threads restoring concurrently for worker pool. Each worker restore one file at a time, increase this as TiKV nodes increase
pool-size = 16
 
# Skip unique index check
skip-unique-check = 0

# An alternative database to restore into

#alternative-db = ""
# Database to restore
#source-db = ""

# DB config
[db]
host = "127.0.0.1"
user = "root"
password = ""
port = 4000

# [[route-rules]]
# pattern-schema = "shard_db_*"
# pattern-table = "shard_table_*"
# target-schema = "shard_db"
# target-table = "shard_table"

```

### Usage

Command line parameter:

    ./bin/loader -d ./test -h 127.0.0.1 -u root -P 4000

Or use configuration file "config.toml":

    ./bin/loader -c=config.toml
    
### Note

+ If you use the default checkpoint-schema, after importing the data of a database, drop the tidb_loader database before you begin to import the next database. 
+ It is recommended to specify the `checkpoint-schema = "tidb_loader"` parameter when importing data.