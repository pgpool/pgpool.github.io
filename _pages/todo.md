---
title: "<i class='fas fa-list-check'></i> TODO"
permalink: /todo/
sidebar:
  nav: "community"
excerpt: "Pgpool-II community TODO."
last_modified_at: 2026-03-26
toc: true
layout: single
---


## Pgpool-II TODO list

### Allow to use multiple pgpool-II instances with in-memory query cache enabled

For this purpose we not only use memcached but also we need to store the oid map info on it to share the info among pgpool-II instances.

According to [https://www.pgpool.net/pipermail/pgpool-hackers/2018-November/003143.html](https://www.pgpool.net/pipermail/pgpool-hackers/2018-November/003143.html),attempt to put oid map into memcached was failed due to reliability and performance reason.

Maybe we should try with more reliable in memory storage engine, such as Redis.

### Allow to use pg_rewind in online recovery

`pg_rewind` could speed up online recovery. However it only works when the target node was normally shut down. Can we recognize that?

Probably yes by looking at `pg_controldata`.

### Support peer auth

Apparently pool_hba.conf should recognize it if we are going to support it. Also pgpool-II should forward it to PostgreSQL.
We need think the case if pg_hba.conf does not use peer auth.

### Allow to use client encoding

It would be nice if pgpool client could use encoding which different from PostgreSQL server encoding.

To implement this, the parser should be able to handle "unsafe" encodings such as Shift_JIS. psql replaces second byte of each multibyte character to fool the parser. We could hire similar strategy.

### Cursor statements are not load balanced, sent to all DB nodes in replication mode

`DECLARE..FETCH` are sent to all DB nodes in replication mode. This is because the `SELECT` might come with `FOR UPDATE/FOR SHARE`.

It would be nice if pgpool-II checks if the `SELECT` uses `FOR UPDATE/FOR SHARE` and if not, enable load balance (or only sends to the master node if load balance is disabled).

Note that some applications including psql could use `CURSOR` for `SELECT`. For example, from PostgreSQL 8.2, if `\set FETCH_COUNT n` is executed, psql unconditionaly uses a curor named "_psql_cursor".

### Handle abnormal down of virtual IP interface when watchdog enabled

When virtual IP interface is dropped abnormally by manual ifconfig etc., there are no one holding VIP, and clients aren't able to connect pgpool-II. Watchdog of active pgpool should monitor the interface or VIP, and handle its down.

### Do not invalidate query cache created in a transaction in some cases

Currently new query cache for table t1 created in a transaction is removed at commit if there's DMLs which touch t1 in the same transaction. Apparently this is overkill for same cases:

```
BEGIN;
INSERT INTO t1 VALUES(1);
SELECT * FROM t1;
COMMIT;
```

To enhance this, we need to teach pgpool-II about "order of SELECTs and DMLs.".

### Fix memory leak in pool_config.c

The module in charge of parsing pgpool.conf has memory leak problem. Usually pgpool reads pgpool.conf just once at the start up time, it is not a big problem.

However reloading pgpool.conf will leak memory and definitely a problem. Also using memory leak check tools like valgrind emit lots of error messages and very annoying. So it would be nice to fix the problem in the future.

### Put together a definition of error codes into a single header file

Currently most error codes used by pool_send_{error,fatal}_message() etc (e.g. "XX000", "XX001", "57000") are hard-coded in different sources. They should be defined as constants in a single header together.

### Import PostgreSQL's latch module

Pgpool already has similar module but PostgreSQL's one seems more sophiscated and reliable.

### Implement "log_timezone"

(From pgpool-genera: 5215)

> I'd like to propose that an addition be made to pgpool to allow for a log timestamp to be written to the log with a timezone other than the locally defined timezone. Where this > is helpful is when we use an external tool like logstashforwarder, where we want the logs to be absorbed with a timestamp with a UTC timezone. Postgres offers this feature
> ('log_timezone'), which we use, and it would be nice to allow pgpool to behave in the same way.

### Allow to reset in memory query cache in the shared memory without restarting Pgpool-II

See discussion: [https://www.pgpool.net/pipermail/pgpool-general/2017-May/005565.html](https://www.pgpool.net/pipermail/pgpool-general/2017-May/005565.html)

### Do not prevent load balancing in explicit transactions in certain cases

If write queries are issued in an explicit transaction, Following `SELECT`s are not load balanced, rather sent to primary node. This is intended to allow `SELECT`s to retrieve the latest data regardless the replication delay. Currently "write query" includes anything other than `SELECT`s. This is overkill for some class of queries: for example, Since `SET` command are sent to both primary and standby nodes, sending `SELECT`s to any of DB nodes could retrieve the latest data.

### Add black/white table list for load balancing control

If a table is in the black list, always send queries to primary server. Probably database.schema.table notion is preferable.

### Support Cert authentication between Pgpool-II and PostgreSQL

Pgpool-II 4.0 added support for Cert authentication between frontend and Pgpool-II, but between Pgpool-II and backend is not yet supported.

### Detach the standby node with large replication lag

Now no loadbalance to the standby node with large replication lag. But if due to some reason of online-recovery the recoveroed standby node can't connect to primary node, the standby node should be detached.

### Allow to get primary node info in failback_command script

Now we can get master node info in `failback_command` script, it will be more useful to get hostname, port and database cluster directory of new primary node.

### Allow to specify whether a relcahe entery is for a global table

Currently relcache is defined for each database. However some of relcache entry does not depend on databases: for example shared catalogs and misc info including PostgreSQL version. For such info having per-database relcache entry is not only waste of resource, but less efficient. It is desirable to be able to specify if a relcache entry does not depend on databases.

### Grouping config requires duplicate codings

pool_config_variable.c manages each config variables along with its group to belong to. However each group definition also requires which config variable belongs to the group. This is redundant and should be avoided.

### Duplicate functionality: "show pool_status" and "pgpool show all"

Both commands produce almost same output except that `show pool_status` lacks some variables because certain config variables were forgotten to be added. Probably we should keep `pgpool show all` only because it does not require to maintaining pool_process_reporting.c. To keep backward compatibility, if `show pool_status` is requested, `pgpool show all` could be called.

### Add PCP command to invalidate particular query cache

Sometimes it is not possible to invalidate cache entry because Pgpool-II fails to detect the table modification ebent; e.g. table modified by functions, triggers and rules. There should be some ways for administrators to invalidate such query cache entries.

### Use a hash table for the relation cache

Currently we use a simple array for the relation cache. Apparently it will not scale if there are many cache entries. Using a hash table should provide quicker lookup.

### Support GSSAPI

Pgpool-II does not support it yet. Moreover if client sends request with "gssencmode=prefer" Pgpool-II fails.

### Allow to reset statistics counters without restarting Pgpool-II

[https://www.pgpool.net/pipermail/pgpool-general/2021-February/007478.html](https://www.pgpool.net/pipermail/pgpool-general/2021-February/007478.html)

### Allow to exec a command when quorum state is changed

Possible use case is, when quorum is lost, admin wants to prevent applications to send queries through pgpool or directory to PostgreSQL. In this case a registered command is executed to shutdown the primary PostgreSQL.

### Do not disturb sessions in failover of standby servers when load_balance_mode is off

In streaming replication mode if `load_balance_mode` is off, it would be desirable to not disconnect sessions in failover of standby servers. Currently Pgpool-II connects to all backend even if `load_balance_mode` is off. But it is actually unnecessary to connect to standby servers if `load_balance_mode` is off. If pgpool only connects to primary server, it does not need to disconnect session in failover of standby servers.

### Pgpool-II leader node switchover

Currently, we need to shutdown the leader pgpool services to change the leader. Also we dont have an option to change the leader to a desired node untill we shutdown the services. the leader node switch should we possible with simple command such as a PCP command, instead of requiring a service restart.

## TODOs already done

### Support IPv6 network
As of 4.5, it is allowed to use IPv6 address for PostgreSQL backend server, listening addresses of pgpool-II itself and listening addresses of pcp process.

However, watchdog process only listens to IPv4 and UNIX domain socket. Following modules need to be updated.
* watchdog communication port (`wd_port`)
  * wd_create_recv_socket/wd_create_client_socket
* heartbeat port (`heartbeat_port`)
  * wd_create_hb_recv_socket/wd_create_hb_send_socket
This has been implemented in 4.6.

### Support multiple pcp_socket_dir
Pgpool-II has supported multiple unix_socket_directories in 4.4 release. I think `pcp_socket_dir` should also support multiple directories.

This has been implemented in 4.5.

### Allow load balance for PREPARE/EXECUTE/DEALLOCATE
Pgpool-II does not load balance these queries even if it's processing read only `SELECT`.

This has been implemented in 4.5.

### Recognize multi statemnet queries
As stated in the document, pgpool-II does not recognize multi statement queries correctly (`BEGIN;SELECT 1;END`). Pgpool-II only parses the first element of the query ("`BEGIN`" in this case) and decides how to behave.

Of course this will bring various problems. It would be nice if pgpool-II could understand the each part of the multi statement queries.

Problem is, how PostgreSQL backend handles the multi statement queries. For example, when client sends `BEGIN;SELECT 1;END`, backend returns "Command Complete" respectively and "Ready for query" is returned only once. Thus, trying to split multi statement queries to non multi statement queries like what psql is doing will not work.

Simon Riggs suggested that if Pgpool-II cannot process multi-statement query properly, then it should have an option to prohibit the multi stattement queries in the developer unconference held in PGConf.ASIA 2016 on December 1st 2016 in Tokyo. (or maybe we could disregard the 2nd or later queires instead).

---------------------------------------------------------------------------------------

This has been implemented in from master (to be 4.5) to 4.1 (as of 2023/4/19).

Now Pgpool-II correctly recognizes multi-statements and distributes the query to proper PostgreSQL nodes.

Basically Pgpool-II forwards multi-statement queries to primary node (or all nodes in replication/snapshot isolation mode). A few SQL like `BEGIN/END/SAVEPOINT/DEALLOCATE` need special handling. This part is implemented in:

[https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=fd32f5ef996cad36d5b1554e92a33ea7a815419a](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=fd32f5ef996cad36d5b1554e92a33ea7a815419a)

Another challenge is how to deal with minimal parser. The minimal parser happily gives up parsing multi-statement query once it finds `UPDATE/INSERT/DELETE` in streaming replication mode and pgpool fails to recognize the query is multi-statement. To fix this, "psqlscan" is imported from PostgreSQL, which precisely detects multi-statement with lower cost than SQL parser. Still it is expensive than simple string comparison, and we use psqlscan only when the query string is large (currently defined as 10kB). As a result, the minimal parser is only used when query is larger than 10kB and the query is not multi-statement.

This part is implemented in:

[https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=48da8715bf403965507eef0321c0ab10054ac71c](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=48da8715bf403965507eef0321c0ab10054ac71c) [https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=64f670ca4abae749e1a95cc57b6a508a8611e44d](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=64f670ca4abae749e1a95cc57b6a508a8611e44d)

As for handling each query in a multi-statement query, it is technically impossible as stated above. So we don't need to worry about this.

We can now safely and correctly handle multi-statement queries.

### Allow to use a custom script to communicate with trusted servers
A hard coded timeout for ping (3 seconds) is not always appropriate.

It also allows to use alternative command which is more suitable than ping in certain system configuration.

This has been implemented in 4.4.

commit:
[https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=b7bcf0d7b833559962cde8c5f4dfe3f5c07dda3c](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=b7bcf0d7b833559962cde8c5f4dfe3f5c07dda3c)

### Support multiple unix_socket_directories and related parameters
PostgreSQL already does this.

Also unix_socket_group and unix_socket_permissions are needed to be supported.

These have been implemented in 4.4.

commit:
[https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=bc03514b124de01176d5ded220f33cabff742ade](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=bc03514b124de01176d5ded220f33cabff742ade)

### Allow SSL in health check etc.
Health check process connects to PostgreSQL backend without using SSL. This means password for connecting PostgreSQL database is flying on the wire in plain text. Same thing can be said to the streaming replication delay check worker process. It would be nice if health check and streaming replication delay check worker process use SSL if requested by backend.

This has been implemented in 2.3.2 (released on 2010/2/7) since SSL was intrdoduced. We usually list newer entries first but it was discovered quite recently that the item had been implemented, and we decided to list the item here.

Note that the streaming replication delay check worker process was introduced in 3.0 (released on 2010/9/10). SSL was already supported in the streaming replication delay check worker.

### Support IPv6 network
As of 3.4, it is allowed to use IPv6 address for PostgreSQL backend server and bind address of pgpool-II itself.

However, PCP process still only binds to IPv4 and UNIX domain socket.

This has been implemented in Pgpool-II 4.4.

### Allow to use comma separated IP address or host names in listen_addresses
Currently only single IP or host name or '*' is allowed. PostgreSQL already allows multiple listen addresses.

This has been implemented in 4.4.

commit:
[https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=fd0efceae011c8d2c2f7c2b26dc0a738f055972e](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=fd0efceae011c8d2c2f7c2b26dc0a738f055972e)
[https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=e661dd1fd561792500ec0a7a4fc05c33891c2dec](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=e661dd1fd561792500ec0a7a4fc05c33891c2dec)

### Allow to specify replication delay by time
delay_threshold specifies replication delay upper limit in bytes. It will be more intuitive to specify delay by time like '10 seconds'.

This has been implemented in Pgpool-II 4.4.

### Track the point to flush messages
When a flush message is received, all pending messages should be flushed to frontend.

This has been implemented in Pgpool-II 4.4.

### Include other file in pgpool.conf file
Add the feature pgpool.conf can include other file.

This has been implemented in Pgpool-II 4.3.

### Allow to use schema qualifications in black_function_list and white_function_list
Currently schema qualifications are ignored.

This has been implemented in 4.2.

commit:
[https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=fc9fa8dbe21dd561b049ad94377a1c0246aec493](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=fc9fa8dbe21dd561b049ad94377a1c0246aec493)

### Set ps status in extended query
Currently ps status is only set for simple query.

This has been implemented in 4.2.

### Change relative path of ssl_key and ssl_cert to DEFAULT_CONFIGDIR
Currently relative path depends on runtime directory.

This has been implemented in 4.2.

### Add support for an user/password input file to pg_md5
[https://www.pgpool.net/mantisbt/view.php?id=422](https://www.pgpool.net/mantisbt/view.php?id=422)

```
pg_md5 can now accept input file.
```

This has been implemented in 4.2.

### Support for CRL (Certificate Revocation List)
Enhances SSL security.

This has been implemented in 4.2.

### Allow to send relation cache query to other than primary node
Pgpool-II needs to access PostgreSQL's system catalog to obtain meta info. For now the query is always sent to primary.
This is good because it could avoid replication delay for newly created tables. However if primary PostgreSQL is geographically distant, the query could take long time. It would be nice if there's a parameter to allow send such queries to other than primary node.

This has been implemented in 4.1.

### Automatically reattach a node in streaming master/slave configuration
In streaming master/slave configuration there could be an option to automatically reattach a node if it's up-to-date with the master (0 bytes behind). It often happens that due to minor network outage a slave node is dropped off from pgpool and stays down even if the the node has resumed replication with master and is up-to-date.pgpool already knows how much slave is behind master so i guess this wouldn't be too difficult to implement? (from bugtrack #17)

Another concern is whether the standby in question actually connects to the proper primary server or not. It is possible that the standby is up and running but is connected to different primary server. Simon Riggs suggested at the developer unconference on December 1st 2016 held in PGConf.ASiA 2016 in Tokyo that pg_stat_wal_receiver, which is new in PostgreSQL 9.6, can be used to safely judge that the standby in question is actually connected to appropriate primary server.
pg_stat_replication provides ideal information for this purpose. By using it this will be supported in

This has been implemented as `auto_failback` 4.1.

### Move relation cache to shared memory
This will bring less inquiry to the system catalog (thus better performance) and more real-time cache invalidation.

This has been implemented in 4.1 as `enable_shared_relcache`.

### Allow to specify which node is dead when starting up
If we set longer health check timeout and/or many health check retries, starting up pgpool-II will take long time if some of DB nodes have been down because of health checking and retries in creating connection to backend.
pgpool_status should help here but for the very first starting up, we cannot use it.

It would be nice if we could tell pgpool-II about down node info.
As of 3.4, pgpool_status file is changed to a plain ASCII file and users could specify down node by using ordinary text editors.

### Ability to load balance based on Client IP, database, table etc.
From bugid 26: I have recently moved a database from Mysql to postgresql 9.1.5 which is behind a pgpool-II-3.1.4 . Everything went fine until i observed that some "tickets" are not created correctly by the application (OTRS) that populate the database.

After some debugging i found/guess that the problem is the following:
when a cron job wants to create a ticket he has to insert info in abut 10 tables, and i guess that the 2-nd, 3-rd ... inserts depends on the first. The problem was that this operation is not performed transactionally so after the first insert, when the app tries to perform the other inserts, first tries to select "the first insert", but this first insert is still not propagated to all nodes, and the error occurs.

I`m aware of the fact that if this entire operation would be performed transactionally (only on master) the issue is solved, but unfortunately i cannot modify the app.

So i want to know if there is any way that i can tell to pgpool something like :
any request from this ip do not load balance.

PS. temporary i have set the weight factor to 0 to the 2-nd and 3-rd postgresql slaves and it behaves ok, because reads and writes only from master.

P.P.S. there's also different request regarding load balance.
[https://www.pgpool.net/pipermail/pgpool-general/2014-June/003032.html](https://www.pgpool.net/pipermail/pgpool-general/2014-June/003032.html)

This item has been implemented in 3.4 as `database_redirect_preference_list` and `app_name_redirect_preference_list`.

### Import PostgreSQL's exception handling
PostgreSQL's exception handling (elog family) is pretty good tool to make codes simple and robust.
It would be nice if pgpool could use this.

This has been already done in 3.4.

### Allow to print user name in the logging
This will be useful for audit purpose.

This has been done and will appear in pgpool-II-3.4.0.

### Remove on disk query cache
Old on disk query cache has almost 0 user and has severe limitations, including no automatic cache invalidation.
This has been already obsoleted since on memory query cache implemented and will appear in 3.4.0.

### Restart watchdog process when it abnormaly exits
It would be nice for pgpool main to restart watchdog process when it dies abnormally.

### Synchronize backend nodes information with watchdog when standby pgpool starts up
When a certain node is detached from active pgpool and then standby pgpool starts up, the standby pgpool can't recognize that the node is detached.

Standby pgpool should get information about node status from other pgpool.

### Avoid multiple pgpools from executing failover.sh simultaneously
In master-slave mode with watchdog, when a backend DB is down, all pgpools execute `failover.sh`.

It might cause something wrong.

### Add new parameter for searching primary node timeout
pgpool-II uses `recovery_timeout` for searching the primary node timeout after failover.

Since this is an abuse of the parameter, we should add new parameter for searching the primary node.

### Allow to load balance even in an explicit transaction in replication mode
Currently load balance in an explicit transaction is only allowed in master-slave mode.

It should be allowed in the replication mode as well.

### Add testing framework
PostgreSQL has nice regression test suite.

It would be nice if pgpool-II has similar test suite.

Such a suite could be very complex system because it should include not only pgpool-II itself, but also multiple PostgreSQL instances.

Even such a test suite should be able to manage multiple pgpool-II instances.

### Add switch to control select(2) time out in connecting to PostgreSQL
In connect_inet_domain_socket_by_port(), select(2) is issued to watch events on the fd created by non blocking connect(2).

The time out parameter of select(2) is fixed to 1 second, which is not long enough in flakey network environment like AWS ([https://www.pgpool.net/pipermail/pgpool-general/2014-May/002880.html](https://www.pgpool.net/pipermail/pgpool-general/2014-May/002880.html)).

To solve the problem, new switch to control the time out is desired (done for pgpool-II 3.4.0).

### Allow to specify which node is dead when starting up
If we set longer health check timeout and/or many health check retries, starting up pgpool-II will take long time if some of DB nodes have been down because of health checking and retries in creating connection to backend.
pgpool_status should help here but for the very first starting up, we cannot use it.

It would be nice if we could tell pgpool-II about down node info (pgpool-II 3.4.0 changes the pgpool_status format to ASCII. Thus users can edit the file if needed).

### Remove parallel query
Parallel query has severe restrictions such as certain queries cannot be used, nor in extended protocol (i.e. JDBC).

Also it is pain to upgrade to newer version of PostgreSQL's SQL parser (yes, pgpool-II uses PostgreSQL's parser code). In short, parallel query gives us small gain comparing with the work needed to maintain/enhance. So I would like to obsolete parallel query in the future pgpool-II release. (related parameters have been removed from pgpool.conf in 3.4.0. pgpool-II 3.5.0 will remove actual code).

### Enhance pcp commands
There are number of drawbacks in pcp commands including 1) the timeout parameter is not used any more and should be removed 2) error codes returned from the commands are completely useless 3) multiple commands cannot be accepted simultaneously.

This has been already done in 3.5.

### Enhance performance of extended protocol case
When extended protocol (i.e. JDBC etc.) used, pgpool-II's overhead is pretty large compared with simple query. Need to enhance it.

This has been already done in 3.5.

### Import PostgreSQL 9.5's parser
No need to say for this.

This has been already done in 3.5.

### Watchdog feature enhancement
Watchdog is a very important feature of pgpool-II as it is used to eliminate the single point of failure and provide HA. But there are few feature requests and bugs in the existing watchdog that require little more than a simple code fix, and requires the complete revisit of its core architecture.
See the design proposal for watchdog enhancement [here]

This has been already done in 3.5.

### Allow to specify user name, password and database name for health check per backend base
In some environment it is not allowed to access standard database i.e. postgres and template1. So users need to specify them per backend basis.

Maybe we need backend_healthcheck_username0 etc? See https://www.pgpool.net/pipermail/pgpool-hackers/2015-June/000942.html for more details.

This has been already done in 3.5.

### Enhance documents
The current document is plain HTML, which is a real pain to maintain. Like PostgreSQL, is SGML our direction?

Pgpool-II 3.6 is going to change the document format to SGML. This has been already implemented in 3.6. We employ SGML.

### Add SET command
Pgpool specific `SET` command would be useful. For example, using `SET debug = 1` could produce debug info on the fly for particular session.

This is being discussed in pgpool-II 3.6 development. This item has been implemented in 3.6.

### Send read query only to standbys even after failover
We can configure pgpool-II to not send read queries to the primary. However after a failover, the role of the node could be changed.

To solve the problem, we need new flag to specify that read queries always are sent to standbys regardless the failover ([pgpool-general: 1621] backend weight after failover).

This has been already implemented in 3.4 as `database_redirect_preference_list` and `app_name_redirect_preference_list`.

### Do not disconnect clients when a failover happens
At this moment we don't know how to implement it but this is a desirable feature.

This has been already implemented in 3.6.

### Create separate process for health checking
To make main process more stable, it would be better to make separate process which is responsible for health checking.

This has been already implemented in 3.7.

### Health-check timeout for each backend node
In the current, timeout values specified by `health_check_timeout` means the total time for checking all the backend status. Hence, if it takes a long time to succeed to check a backend, when timeout occurs during checking the next backend, this node is regarded as failed and failovered even though this is healthy.To resolve this issue, we need health-check timeout for each backend.

This has been implemented in 3.7.

### Support SCRAM authentication
PostgreSQL 10.0 supports SCRAM authentication. It seems there's fundamental difficulty with this.

See [https://www.pgpool.net/pipermail/pgpool-hackers/2017-May/002331.html](https://www.pgpool.net/pipermail/pgpool-hackers/2017-May/002331.html) for more details.

This has been implemented in 4.0.

### Allow to choose load balance behavior per SQL statement
If a query string matches specified regular expression, send the query to either primary or standby.

This has been implemented in 4.0.

### Allow to specify load balance weight ratio for load balance parameters
Allow to specify load balance weight ratio for `database_redirect_preference_list` and `app_name_redirect_preference_list` like: "postgres:primary(0.3)".

See [https://www.pgpool.net/pipermail/pgpool-hackers/2017-December/002650.html](https://www.pgpool.net/pipermail/pgpool-hackers/2017-December/002650.html)

This has been implemented in 4.0.

### Support for SSL using ECDH
ECDH is encryption algorithm. Our SSL support lacks this (PostgreSQL already has this) and supporting ECDH should make Pgpool-II more secure.

This has been implemented in 4.1.

commit:
[https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=51bc494aaa7fd191e14038204d18effe2efb0ec8](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=51bc494aaa7fd191e14038204d18effe2efb0ec8)

### Allow to set application name in log_line_prefix
Currently, application name (%a) can only be set if startup packet includes it. It would be nice if Pgpool-II traps "`set application_name...`" in the current session and allows `log_line_prefix` to use it.

This has been implemented in 4.2.
commit:
[https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=d8434662d63c115280779941af61252663f9c134](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=d8434662d63c115280779941af61252663f9c134)

### Support for SSL passphrase
Using passphrase to encrypt private key is more secure. PostgreSQL already has this. Pgpool-II should import it.

This has been implemented in 4.2.

commit:
[https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=6ea154454c38d3f1191772f6a3aa01aa60a69c86](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=commit;h=6ea154454c38d3f1191772f6a3aa01aa60a69c86)