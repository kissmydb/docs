---
title: TiDB Transaction Isolation Levels
category: user guide
---

# TiDB Transaction Isolation Levels

Transaction isolation is one of the foundations of database transaction processing. Isolation is the I in the acronym ACID (Atomicity, Consistency, Isolation, Durability), which represents the isolation property of database transactions.

The SQL-92 standard defines four levels of transaction isolation: Read Uncommitted, Read Committed, Repeatable Read and Serializable. See the following table for details:

| Isolation Level  | Dirty Read   | Nonrepeatable Read | Phantom Read          | Serialization Anomaly |
| ---------------- | ------------ | ------------------ | --------------------- | --------------------- |
| Read Uncommitted | Possible     | Possible           | Possible              | Possible              |
| Read Committed   | Not possible | Possible           | Possible              | Possible              |
| Repeatable Read  | Not possible | Not possible       | Not possible in  TiDB | Possible              |
| Serializable     | Not possible | Not possible       | Not possible          | Not possible          |

TiDB offers two transaction isolation levels: Read Committed and Repeatable Read.

TiDB uses the [Percolator transaction model](https://research.google.com/pubs/pub36726.html). A global read timestamp is obtained when the transaction is started, and a global commit timestamp is obtained when the transaction is committed. The execution order of transactions is confirmed based on the timestamps. To know more about the implementation of TiDB transaction model, see [MVCC in TiKV](https://pingcap.com/blog/2016-11-17-mvcc-in-tikv/).

Use the following command to set the level of transaction isolation:

```
SET SESSION TRANSACTION ISOLATION LEVEL [read committed|repeatable read]
```

## Repeatable Read

Repeatable Read is the default transaction isolation level in TiDB. The Repeatable Read isolation level only sees data committed before the transaction begins, and it never sees either uncommitted data or changes committed during transaction execution by concurrent transactions. However, the transaction statement does see the effects of previous updates executed within its own transaction, even though they are not yet committed.

For transactions running on different nodes, the start and commit order depends on the order that the timestamp is obtained from PD.

Transactions of the Repeatable Read isolation level cannot concurrently update a same row. When committing, if the transaction finds that the row has been updated by another transaction after it starts, then the transaction rolls back and retries automatically. For example:

```
create table t1(id int);
insert into t1 values(0);

start transaction;              |               start transaction;
select * from t1;               |               select * from t1;
update t1 set id=id+1;          |               update t1 set id=id+1;
commit;                         |
                                |               commit; -- roll back and retry atutomatically
```

### Difference between TiDB and ANSI Repeatable Read

The Repeatable Read isolation level in TiDB differs from ANSI Repeatable Read isolation level, though they sharing the same name. According to the standard described in the [A Critique of ANSI SQL Isolation Levels](https://www.microsoft.com/en-us/research/wp-content/uploads/2016/02/tr-95-51.pdf) paper, TiDB implements the snapshot isolation level, and it does not allow phantom reads but allows write skews. In contrast, the ANSI Repeatable Read isolation level allows phantom reads but does not allow write skews.

### Difference between TiDB and MySQL Repeatable Read

The Repeatable Read isolation level in TiDB differs from that in MySQL. The MySQL Repeatable Read isolation level does not check whether the current version is visible when updating, which means it can continue to update even if the row has been updated after the transaction starts. In contrast, if the row has been updated after the transaction starts, the TiDB transaction is rolled back and retried. Transaction Retries in TiDB might fail, leading to a final failure of the transaction, while in MySQL the updating transaction can be successful.

The MySQL Repeatable Read isolation level is not the snapshot isolation level. The consistency of MySQL Repeatable Read isolation level is weaker than both the snapshot isolation level and TiDB Repeatable Read isolation level.

## Read Committed

The Read Committed isolation level differs from Repeatable Read isolation level. Read Committed only guarantees the uncommitted data cannot be read.

> **Note:**
>
> Because the transaction commit is a dynamic process, the Read Committed isolation level might read the data committed by part of the transaction. It is not recommended to use the Read Committed isolation level in a database that requires strict consistency.

## Transaction retry

For the `insert/delete/update` operation, if the transaction fails and can be retried according to the system, the transaction is automatically retried within the system.

You can control the number of retries by configuring the `retry-limit` parameter:

```
[performance]
...
# The maximum number of retries when commit a transaction.
retry-limit = 10
```
