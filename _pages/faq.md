---
title: "<i class='fas fa-circle-question'></i> FAQ"
permalink: /faq/
sidebar:
  nav: "about"
excerpt: "FAQ"
last_modified_at: 2026-03-26
toc: true
layout: single
---

## Pgpool-II Frequently Asked Questions

### Why configure fails by "pg_config not found" on my Ubuntu box?

`pg_config` is in libpq-dev package. You need to install it before running configure.

### Why records inserted on the primary node do not appear on the standby nodes?

Are you using streaming replication and a hash index on the table?
Then it's a known limitation of streaming replication. The inserted record is there.
But if you SELECT the record using the hash index, it will not appear. Hash index
changes do not produce WAL record thus they are not reflected to the standby nodes.

Solutions are: 1) use btree index instead 2) use pgpool-II native replication.

### Can I mix different versions of PostgreSQL as pgpool-II backends?

You cannot mix different major versions of PostgreSQL, for example 8.4.x and 9.0.x.
On the other hand you can mix different minor versions of PostgreSQL, for example 9.0.3
and 9.0.4. Pgpool-II assumes messages from PostgreSQL to pgpool-II are identical
anytime. Different major version of PostgreSQL may send out different messages and this
would cause trouble for Pgpool-II.

### Can I mix different platforms of PostgreSQL as pgpool-II backends, for example Linux and Windows?

In streaming replication mode, no. Because streaming replication requires that primary
and standby platforms are phsyically identical. On the other hand, pgpool-II's
replication mode only requires logically database clusters identical. Beware, however,
that online recovery script does not use rsync or some such, which do phical copying
among database clusters. You want to use `pg_dumpall` instead.

### It seems my pgpool-II does not do load balancing. Why?

First of all, pgpool-II' load balancing is "session base", not "statement base".
That means, DB node selection for load balancing is decided at the beginning of session.
So all SQL statements are sent to the same DB node until the session ends.

Another point is, whether statement is in an explicit transaction or not. If the
statement is in a transaction, it will not be load balanced in the replication mode.
In pgpool-II 3.0 or later, SELECT will be load balanced even in a transaction if operated in the master/slave mode.

Note the method to choose DB node is not LRU or some such. Pgpool-II chooses DB node
randomly considering the "weight" parameter in pgpool.conf. This means that the chosen
DB node is not uniformly distributed among DB nodes in short term. You might want to
inspect the effect of load balancing after ~100 queries have been sent.

Also cursor statements are not load balanced in replication mode. i.e.:DECLARE..FETCH
are sent to all DB nodes in replication mode. This is because the SELECT might come
with `FOR UPDATE/FOR SHARE`. Note that some applications including psql could use CURSOR for SELECT. For example, from PostgreSQL 8.2, if `"\set FETCH_COUNT n"` is executed, psql unconditionaly uses a curor named `"_psql_cursor"`.

### How can I observe the effect of load balancing?

We recommend to enable `log_per_node_statement` directive in pgpool.conf for this.
Here is an example of the log:
```
2011-05-07 08:42:42 LOG:   pid 22382: DB node id: 1 backend pid: 22409 statement: SELECT abalance FROM pgbench_accounts WHERE aid = 62797;
```

The "DB node id: 1" shows which DB node was chosen for this loadbalancing session.

Please make sure that you start pgpool-II with "-n" option to get Pgpool-II log.
(or you can use syslog in pgpool-II 3.1 or later)

### Why am I getting "ProcessFrontendResponse: failed to read kind from frontend. frontend abnormally exited" in my pgool log?

Well, your clients might be ill-behaved:-) PostgreSQL's protocol requires clients to send particular packet before they disconnect the connection. pgpool-II complains that clients disconnect without sending the packet. You could reprodcude the problem by using psql. Connect to pgpool using psql. Kill -9 psql. You will silimar message in the log. The message will not appear if you quit psql normaly. Another possibility is unstable network connection between your client machine and pgpool-II. Check the cable and network interface card.


### I'm running pgpool-II in streaming replication mode. It seems it works but I find following errors in the log. Why?
```
2011-07-19 08:21:59 ERROR: pid 10727: s_do_auth: unknown response "E" before processing BackendKeyData
2011-07-19 08:21:59 ERROR: pid 10727: s_do_auth: unknown response "" before processing BackendKeyData
2011-07-19 08:21:59 ERROR: pid 10727: s_do_auth: unknown response "" before processing BackendKeyData
2011-07-19 08:21:59 ERROR: pid 10727: s_do_auth: unknown response "" before processing BackendKeyData
2011-07-19 08:21:59 ERROR: pid 10727: s_do_auth: unknown response "[" before processing BackendKeyData
2011-07-19 08:21:59 ERROR: pid 10727: pool_read2: EOF encountered with backend
2011-07-19 08:21:59 ERROR: pid 10727: make_persistent_db_connection: s_do_auth failed
2011-07-19 08:21:59 ERROR: pid 10727: find_primary_node: make_persistent_connection failed
```

Pgpool-II tries to connect to PostgreSQL to execute some functions such as
`pg_current_xlog_location()`, which is used for detecting primary server or checking replication
delay. The messages above indicates that Pgpool-II failed to connect with
user = health_check_user and password = health_check_password.
You need to set them properly even if `health_check_period = 0`.

Note that Pgpool-II 3.1 or later will use `sr_check_user` and `sr_check_password` for it instead.

### When I run pgbench to test pgpool-II, pgbench hangs. If I directly run pgbench against PostgreSQL, it works fine. Why?

pgbench creates concurrent connections (the number of connections is specified by
`-c` option) before starting actual transactions. So if the number of concurrent
transactions specified by `-c` exceeds `num_init_children`, pgbench will stuck because
it will wait for pgpool accepting connections forever (remember that pgpool-II accepts
up to `num_init_children` concurrent sessions. If the number of concurrent sessions
reach `num_init_children`, new session will be queued). On the other hand PostgreSQL
does not accept concurrent sessions more than `max_connections`. So in this case you
will just see PostgreSQL errors, rather than connection blocking. If you want to test
pgpool-II's connection queuing, you can use psql instead of pgbench. In the example
session below, `num_init_children = 1`
(this is not a recommended setting in the real world. This is just for simplicity).

```
$ psql test <-- connect to pgpool from terminal #1
psql (9.1.1)
Type "help" for help.
test=# 
$ psql test <-- tries to connect to pgpool from terminal #2 but it is blocked.
test=# SELECT 1; <--- do something from terminal #1 psql
test=# \q <-- quit psql session on terminal #1
psql (9.1.1) <-- now psql on terminal #2 accepts session
Type "help" for help.
test=#
```

### I created pool_hba.conf and pool_passwd to enable md5 authentication through pgpool-II but it does not work. Why?

Probably you made mistake somewhere.
For your help here is a table which describes error patterns depending on the setting of
`pg_hba.conf`, `pool_hba.conf` and `pool_passwd`.

| pg_hba.conf | pool_hba.conf | pool_passwd | result |
| md5         | md5           | yes         | md5 auth |
| md5         | md5           | no          | "MD5" authentication with pgpool failed for user "XX" |
| md5         | trust         | yes/no      | MD5 authentication is unsupported in replication, master-slave and parallel mode |
| trust       | md5           | yes         | no auth |
| trust       | md5           | no          | "MD5" authentication with pgpool failed for user "XX" |
| trust       | trust         | yes/no      | no auth |


### How can I set up SSL for pgpool-II?

SSL support for pgpool-II consists of two parts:

1. between client and pgpool-II
1. pgpool-II and PostgreSQL.

#1 and #2 are independent each other. For example, you can only enable SSL connection
of #1, or #2. Or you can enable both #1 and #2. I explain #1
(for #2, please take a look at PostgreSQL documentation).

Make sure that pgpool is built with openssl. If you build from source code, use
`--with-openssl` option.

1. First create server certificate. In the command below you will be asked PEM pass
phrase (It will be asked when pgpool starts up).
If you want to start pgpool without being asked pass phrase, you can remove it later.
```
openssl req -new -text -out server.req
```
sample server certficate create session:
```
$ openssl req -new -text -out server.req
Generating a 1024 bit RSA private key
.........++++++
..++++++
writing new private key to 'privkey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:JP
JP
State or Province Name (full name) [Some-State]:Tokyo
Tokyo
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Pgpool Global Development Group
Pgpool Global Development Group
Organizational Unit Name (eg, section) []:
Common Name (eg, YOUR name) []:
Email Address []:
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

1. Remove PEM pass phrase if you want.
```
$ openssl rsa -in privkey.pem -out server.key
Enter pass phrase for privkey.pem:
writing RSA key
$ rm privkey.pem
```

1. Turn the certificate into a self-signed certificate.
```
$ openssl req -x509 -in server.req -text -key server.key -out server.crt
```

1. Copy server.key and server.crt to appropreate place. Suppose we copy to `/usr/local/etc`.
Make sure that you use cp -p to retain appropreate permission of server.key.
Alternatively you can set permission later.
```
$ chmod og-rwx /usr/local/etc/server.key
```

1. Set the certificate and key location in `pgpool.conf`.
```
ssl = on
ssl_key = '/usr/local/etc/server.key'
ssl_cert = '/usr/local/etc/server.crt'
```

1. Restart pgpool. To confirm SSL connection between client and pgpool is working, connect to pgpool using psql.
```
psql -h localhost -p 9999 test
psql (9.1.1)
SSL connection (cipher: AES256-SHA, bits: 256)
Type "help" for help.
test=# \q
```

If you see "SSL connection...", SSL connection between client and pgpool is working.
Please make sure that use "-h localhost" option. Because SSL only works with TCP/IP,
with Unix domain socket SSL does not work. 

### I'm using pgpool-II in replication mode. I expected that pgpool-II replaces current_timestamp call with time constants in my INSERT query, but actually it doesn't. Why?

Probably your INSERT query uses schema qualied table name (like public.mytable) and you
did not install pool_regclass function coming pgpool. Without pgpool_reglclass,
pgpool-II only deals with table names without schema qualification.

### Why max_connection must satisfy this formula max_connection >= (num_init_children * max_pool) and not max_connection >= num_init_children?

Probably you need to understand how pgpool uses these variables. Here is internal processing inside pgpool.

1. Wait for connection request from clients.
1. pgpool child receives connection request from a client.
1. The pgpool child looks for existing connection in the pool which
   has requested database/user pair up to `max_pool`.
1. If found, reuse it.
1. If not found, opens a new connection to PostgreSQL and registers to
   the pool.  If the pool has no empty slot, closes the oldest
   connection to PostgreSQL and reuse the slot.
1. Do some query processing until the client sends session close request.
1. Close the connection to client but keeps the connection to
   PostgreSQL for future use.
1. Go to #1

### Is connection pool cache shared among pgpool process?

No, the connection pool cache is in pgpool's process private memory and is not shared
by other pgpool. This is how the connection cache is managed: Suppose pgpool process
12345 has connection cache for database A/user B but process 12346 does not have
connection cache for database A/user B and both 12345 and 12346 are in idle state
(no client is connecting at this point). If client connects to pgpool process 12345
with database A/user B, then the exisiting connection of 12345 is reused. On the other
hand, If client connects to pgpool process 12346, 12346 needs to create new connection.
Whether 12345 or 12346 is chosen, is not under control of pgpool. However in the long
run, each pgpool child process will be equally chosen and it is expected that each
process's pool will be resued equally.

### Why my SELECTs are not cached?

Certain libraries such as iBatis, MyBatis always rollback transactions if they are not
explicitely committed. Pgpool never caches SELECTs result in a rollbacked transaction
because they might not be inconsistent.

### Can I use # comments or blank lines in pool_passwd?

The answer is simple. No (just like /etc/passwd).

### I cannot use MD5 authentication if start pgpool without -n option. Why?

You must have given -f option as a relative path: i.e. `-f pgpool.conf`, rather than
full path: i.e. `-f /usr/local/etc/pgpool.conf`. Pgpool tries to locate the full path
of pool_passwd (which is neccesary for MD5 auth) from pgpool.conf path. This is fine
with `-n` option. However if pgpool starts without `-n` option, it changes current
directory to `/`, which is neccessary processs for daemonizing. As a result, pgpool
tries to open `/pool_passwd`, which will not successs.

### I see standby servers go down status in steaming replication mode and see PostgreSQL messages "terminating connection due to conflict" Why?

If you see following messages along with those, it is likely vacuum on primary server
removes rows which SELECTs on standby server want to see. Workaround is setting
`hot_standby_feedback = on` in your standby server's `postgresql.conf`.

```
2013-04-07 19:38:10 UTC FATAL:  terminating connection due to conflict with recovery
2013-04-07 19:38:10 UTC DETAIL:  User query might have needed to see row versions that must be removed.
2013-04-07 19:38:10 UTC HINT:  In a moment you should be able to reconnect to the database and repeat your command.
2013-04-07 19:38:10 UTC LOG:  could not send data to client: Connection reset by peer
2013-04-07 19:38:10 UTC ERROR:  canceling statement due to conflict with recovery
2013-04-07 19:38:10 UTC DETAIL:  User query might have needed to see row versions that must be removed.
2013-04-07 19:38:10 UTC LOG:  could not send data to client: Broken pipe
2013-04-07 19:38:10 UTC FATAL:  connection to client lost
```

### Every few minites load of the system which pgpool-II running on gets high as much as 5-10. Why?

Mulptiple users stats that this is observed only Linux kernel 3.0. 2.6 or 3.2 does show
the behavior. We suspect that there is a problem with 3.0 kernel. See more discussions
on "[pgpool-general: 1528] Mysterious Load Spikes".

### When watchdog enabled and the connection number reach the number of num_init_children, VIP switchover occurs. Why?

When the connection number reach the number of `num_init_children`, the watchdog will
be failed because select 1 is failed, and then VIP will be transfer to another pgpool.
Unfortunately, there are no way to discriminate normal client's connections from
watchdog's connection. Larger `num_init_children`, `wd_life_point` and smaller
`wd_interval` may prevent the problem somewhat. 

The next major version, pgpool-II 3.3, will support a new monitoring method which uses
UDP heartbeat packets instead of queries such like `SELECT 1` to resolve the problem.

### Why do I need to install pgpool_regclass?

If you are using PostgreSQL 8.0 or later, installing pgpool_regclass function on all
PostgreSQL to be accessed by pgpool-II is strongly recommended, as it is used internally
by pgpool-II. Without this, handling of duplicate table names in different schema might
cause trouble (temporary tables aren't a problem).

Related FAQ is [here](#I'm using pgpool-II in replication mode. I expected that pgpool-II replaces current_timestamp call with time constants in my INSERT query, but actually it doesn't. Why?)

If you are using PostgreSQL 9.4.0 or later and pgpool-II 3.3.4 or later, 3.4.0 or
later, you don't need to install `pgpool_regclass` since PostgreSQL 9.4 has built-in
`pgpool_regclass` like function `to_regclass`.

### md5 authentication does not work. Please help

There's an excellent summary of various check points to set up md5 authentication.
Please take a look at it.

https://www.pgpool.net/pipermail/pgpool-general/2013-May/001773.html

### I'm running pgpool/PostgreSQL on Amazon AWS and occasionaly I get network errors. Why?

It's a known problem with AWS. We recommend to complain to the Amazon support.
pgpool-II 3.3.4, 3.2.9 or later mitigate the problem by changing timeout value for
connect(actually select system call) from 1 second to 10 seconds.
Also pgpool-II 3.4 or later has a switch to control the timeout value.

### I cannot run pcp command on my Ubuntu box. Why?
pcp commands need libpcp.so. In Ubuntu it is included "libpgpool0" package.

### On line recovery failed. How can I debug this?
`pcp_recovery_node` executes `recovery_1st_stage_command` and/or
`recovery_2nd_stage_command` depending on your configuration. Those scripts are
supposed to be executed on the master PostgreSQL node (the first live node in
replication mode or primary node in streaming replication mode).
"BackendError" means there's something wrong in pgpool and/or PostgreSQL.
To verify this, I recommend followings;

1. start pgpool with debug option
1. execute pcp_recovery_node
1. examin pgpool log and master PostgreSQL log

### Watchdog doesn't start if not all "other" nodes are alive

It's a feature. Watchdog's lifeheck will start after all of the pgpools has started.
Until this, failover of the virtual IP never occurs.

### If I start transaction, pgool-II also starts a transaction on standby nodes. Why?

This is necessary to deal with the case when JDBC driver wants to use cursors.
Pgpool-II takes a liberty of distributing SELECTs to the standby node including cursor
statements. Unfortunately cursor statements need to be executed in an explicit
transaction.

### When I use schema qualified table names, pgpool-II does not invalidate on memory query cache and I got outdated data. Why?

It seems you did not install `pgpool_regclass` function. Without the function,
Pgpool-II ignores the schema name pat of the schema qualified table name and the cache
invalidation fails.

### I periodically get error message like "read_startup_packet: incorrect packet length". What does it mean?

Monitoring tools including Zabbix and Nagios periodically sends a packet or ping to
the port which pgoool is listening on. Unfortunately those packets do not have
correct contents, and pgpool-II complains it. If you are not sure who is sending
such a packet, you could turn on `log_connections` to know the source host and port
info. If they are from such tools, you could stop the monitoring to avoid the
problem or even better, change the monitoring method to send legal query, for
example, `SELECT 1`. 

### I'm getting repeated errors like this every few minutes on Tomcat: "An I/O Error occurred while sending to the backend" Why?

Tomcat creates persistent connections to pgpool. If you set `client_idle_limit` to
non 0, pgpool disconnects the connection and next time when Tomcat tries to send
something to pgpool it breaks with the error message.

One solution is set client_idle_limit to 0. However this will leave lots of idle connections.

Another solution provided by Lachezar Dobrev is:
You might solve that by adding a time-out on the Tomcat side. https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html
What you should set is (AFAIK):
* minIdle (default is 10, set to 0)
* timeBetweenEvictionRunsMillis (default 5000)
* minEvictableIdleTimeMillis    (default 60000)

This will try every 5 seconds and close any connections that were not used in the
last 60 seconds. If you keep the sum of both numbers below the client time-out on
the pgpool size connections should be closed at Tomcat side before they time-out on
the pgpool side.

It is also beneficial to set the:

* testOnBorrow (default false, set to true)
* validationQuery (default none, set to 'SELECT version();' no quotes)

This will help with connections should they expire while waiting, without supplying
a disconnected connection to the application.

### When I check pg_stat_activity view, I see a query like `SELECT count(*) FROM pg_catalog.pg_class AS c WHERE c.oid = pgpool_regclass('pgbench_accounts') AND c.relpersistence = 'u'` in active state for very long time. Why?

It's a limitation of pg_stat_activity. You can safely ignore it.

Pgpool-II issues queries like above for internal use to master node. When user query
runs in extended protocol mode (sent from JDBC driver, for example), pgpool-II's
query also runs in the mode. To make `pg_stat_activity` recognize the query finishes,
pgpool-II needs to send a packet called "Sync", which unfortunately breaks user's
query (more precisely, unnamed portal). Thus pgpool-II sends "Flush" packet instead
but then pg_stat_activity does not recognize the end of the query.

Interesting thing is, if you enable `log_duration`, it logs the query finishes.

### Online recovery always fails after certain minutes. Why?

It is possible that PostgreSQL statement_timeout kills the online recovery process.
The process is executed as a SQL statement and if it's running too long, PostgreSQL
sends signal 2 to the SQL and kills it. Varying by the size of the database, the
online recovery process takes very long time. Make sure to disable 
`statement_timeout` or set it long enough time.

### Why "SET default_transaction_isolation TO DEFAULT" fails ?

```
$ psql -h localhost -p 9999 -c 'SET default_transaction_isolation to DEFAULT;'
ERROR: kind mismatch among backends. Possible last query was: "SET default_transaction_isolation to DEFAULT;" kind details are: 0[N: statement: SET default_transaction_isolation to DEFAULT;] 1[C]
HINT: check data consistency among db nodes
ERROR: kind mismatch among backends. Possible last query was: "SET default_transaction_isolation to DEFAULT;" kind details are: 0[N: statement: SET default_transaction_isolation to DEFAULT;] 1[C]
HINT: check data consistency among db nodes
connection to server was lost
```

Pgpool-II detects that node 0 returns "N" (a NOTICE message comes from PostgreSQL)
while node 1 returns "C" (which means the command finished).

Though pgpool-II expects that both node 0 and 1 returns identical messages,
actually they are not. So pgpool-II threw an error.

Probably certain log/message settings are different in node 0 and 1.
Please check client_min_messages or something like that.

They should be identical.

### How does pgpool-II find the primary node?

pgpool-II issues `SELECT pg_is_in_recovery()` to each DB node. If it returns true,
then the node is a standby node. If one of DB nodes returns false, then the node is
the primary node and done.

Because it is possible that promoting node could return true for the SELECT, if no primary node is found and "search_primary_node_timeout" is greater than 0, pgpool-II sleeps 1 second and contines to issues the SELECT query to each DB node again until total sleep time exceeds search_primary_node_timeout.

### Can I use pg_cancel_backend() or pg_terminate_backend()?

You can safely use pg_cancel_backend().

Be warned that pg_terminate_backend() will cause a fail over because it makes PostgreSQL
emit an identical error code as postmaster shutdown. Pgpool-II 3.6 or greater mitigates
the problem. See [the manual](https://www.pgpool.net/docs/latest/en/html/restrictions.html) for more details.

Remember that pgpool-II manages multiple PostgreSQL servers. To use the function, you not
only need to identify the backend pid but the backend server.

If the query is running on the primary server, you can call the function something like

```
/*NO LOAD BALANCE*/ SELECT pg_cancel_backend(pid)
```

SQL comment is to prevent the SELECT be load balanced to standby. Of course you
could issue the SELECT against directly the primary server.

If the query is running on one of standby servers, you need to issue the SELECT
against directly the standby server.

### Why my client is disconnected to pgpool-II when failover happens?

pgpool-II consists of many process where each process corresponds to client session.
When failover occurs, each process may iterate on a loop for each backend without
knowing a backend goes down. This may result in incorrect processing, or segfault
in the worst case. For this reason, when failover occurs, pgpool-II parent process
interrupt child process using signal to let them exit. Note that switch over using
`pcp_detach_node` has same effect.

From Pgpool-II 3.6 or greater, however, a fail over does not cause the disconnection
in certain conditions. See the manual for more details.

### Why am I getting "LOG:  forked new pcp worker ..," and "LOG:  PCP process with pid: xxxx exit with SUCCESS." messages in pgpool log?

Prior to pgpool-II 3.5, pgpool could only handle single PCP command at a time and
all PCP commands were handled by a single PCP child process which lives throughout
the lifespan of pgpool-II main process. In pgpool II 3.5 the restriction of single
PCP command is removed and pgpool-II can now handle multiple simultaneous PCP
commands. For every PCP command issued to pgpool a new PCP child process is forked
and that process exits after execution of the PCP command is complete. So these log
messages are perfectly normal and are generated whenever a new PCP worker process
is created or completes execution for a PCP command.

### How does pgpool-II handle md5 authentication?

1. PostgreSQL and pgpool store md5(password+username) into pg_authid or pool_password. From now on I denote string md5(password+username) as "S".
1. When md5 auth is requested, pgpool sends a random number salt "s0" to frontend.
1. Frontend replies back to pgpool with md5(S+s0).
1. pgpool extracts S from pgpool_passwd and calculate md5(S+s0). If #3 and #4 matches, goes to next step.
1. Each backend sends salt to pgpool. Suppose we have two backends b1 and b2, and salts are s1 and s2.
1. pgpool extracts S from pgpool_passwd and calculate md5(S+s1) and send it to b1. pgpool extracts S from pgpool_passwd and calculate md5(S+s2) and send it to b2.
1. If b1 and b2 agree with the authentication, the whole md5 auth process succeeds.

### Why Pgpool-II does not automatically recognize a database comes back online?

It would be technically possible but we don't think it's a safe feature.
Consider a streaming replication configuration. When a standby comes back online, it does not necessarily means it connects to the current primary node. It may connect to a different primary node , or even it's not a standby any more. If Pgpool-II automatically recognizes such that standby as online, SELECTs to the standby node will return different result as the primary, which is a disaster for database applications.
Also please note that "pgpool reload" does not do anything for recognizing the standby node as online. It just reloads configuration files.
Please note that in Pgpool-II 4.1 or later, it is possible to automatically make a standby server online if it's safe enough. See configuration parameter "auto_failback" for more information.

### After enabling idle_in_transaction_session_timeout, Pgpool-II sets the DB node status to all down

idle_in_transaction_session_timeout was introduced in PostgreSQL 9.6. It is intended to cancel idle transactions. Unfortunately after the time out occurs, PostgreSQL raises a FATAL error, which triggers failover in Pgpool-II if fail_over_on_backend_error is on.
These are some workarounds to avoid the unwanted failover.
* Disable fail_over_on_backend_error. By this, failover will not happen if the FATAL error occurs, but the session will be terminated.
* Set connection_life_time、child_life_time and client_idle_limit less than idle_in_transaction_session_timeout. This will not terminate the session even if the FATAL error occurs. However even if the FATAL error does not occur, the connection pools are removed if one or more of items satisfy the specified condition, which may affect performance.

### How can I know PostgreSQL backend status connected by Pgpool-II?

The backend status shown in `pg_stat_activity` can be examined by using
`show pool_pools` command. One of the columns `pool_backendpid` shown by
`show pool_pools` is the process id of the corresponding PostgreSQL backend process.
Once it is determined, you can examine the output of `pg_stat_activiy` matching with
its `pid` column.

You can do this automatically by using dblink extension of PostgreSQL.
Here is a sample query:

```
SELECT * FROM dblink('dbname=test host=xxx port=11000 user=t-ishii password=xxx', 'show pool_pools') as t1 (pool_pid int, start_time text, pool_id int, backend_id int, database text, username text, create_time text,majorversion int, minorversion int, pool_counter int, pool_backendpid int, pool_connected int), pg_stat_activity p WHERE p.pid = t1.pool_backendpid;
```

You can execute the SQL above on either PostgreSQL or Pgpool-II. The first argument
of dblink is a connection string to connect to Pgpool-II, not PostgreSQL.

### Where can I get Debian packages for Pgpool-II?

You can get Debian packages here: https://apt.postgresql.org/pub/repos/apt/pool/main/p/pgpool2/
For older releases you can find the packages at: https://atalia.postgresql.org/morgue/p/pgpool2/

### How to run Pgpool-II with non-root user?

If you install Pgpool-II using RPM packages, Pgpool-II will be running by root by default.
You can also run Pgpool-II with a non-root user. But root privilege is required to
control the virtual IP, so you have to copy ip/ifconfig/arping command and add the
setuid flag to them.

Following is an example to run Pgpool-II with postgres user.

1. Edit `pgpool.service` file to use postgres user to start Pgpool-II
```
# cp /usr/lib/systemd/system/pgpool.service /etc/systemd/system/pgpool.service
# vi /etc/systemd/system/pgpool.service
...
User=postgres
Group=postgres
...
```

1. Change owner of /var/{lib,run}/pgpool
```
# chown postgres:postgres /var/{lib,run}/pgpool
# cp /usr/lib/tmpfiles.d/pgpool-II-pgxx.conf /etc/tmpfiles.d
# vi /etc/tmpfiles.d/pgpool-II-pgxx.conf
===
d /var/run/pgpool 0755 postgres postgres -
===
```

1. Change owner of Pgpool-II config files 
```
chown -R postgres:postgres /etc/pgpool-II/
```

1. Copy ip/ifconfig/arping commands to somewhere where the user has access permissions and add setuid flag to them.
```
# mkdir /var/lib/pgsql/sbin
# chown postgres:postgres /var/lib/pgsql/sbin
# chmod 700 /var/lib/pgsql/sbin
# cp /sbin/ifconfig /var/lib/pgsql/sbin
# cp /sbin/arping /var/lib/pgsql/sbin
# cp /sbin/ip /var/lib/pgsql/sbin
# chmod 4755 /var/lib/pgsql/sbin/ip
# chmod 4755 /var/lib/pgsql/sbin/ifconfig
# chmod 4755 /var/lib/pgsql/sbin/arping 
```

### Can I use repmgr with Pgpool-II?

No. These software do not consider each other. You should use Pgpool-II without repmger or use repmgr without Pgpool-II. See this message for more details: https://www.pgpool.net/pipermail/pgpool-general/2019-August/006743.html

### Connection fails in CentOS6

Pgpool-II doesn't support GSSAPI authentication yet, but GSSAPI is requested in
CentOS6. Therefore, the connection attempt will fail in CentOS6. 

A workaround is to set a environment variable to disable GSSAPI encryption in the
client:

```
export PGGSSENCMODE=disable
```

### Watchdog standby does not take over master when the master goes down

If you have even number of watchdog nodes, you need to turn on 
`enable_consensus_with_half_votes` parameter, which is new in 4.1.
The reason why you need this is explained in the 4.1 release note:
> This changes the behavior of the decision of quorum existence and failover consensus on even number (i.e. 2, 4, 6...) of watchdog clusters. Odd number of clusters (3, 5, 7...) are not affected. When this parameter is off (the default), a 2 node watchdog cluster needs to have both 2 nodes are alive to have a quorum. If the quorum does not exist and 1 node goes down, then 1) VIP will be lost, 2) failover srcript is not executed and 3) no watchdog master exists. Especially #2 could be troublesome because no new primary PostgreSQL exists if existing primary goes down. Probably 2 node watchdog cluster users want to turn on this parameter to keep the existing behavior. On the other hand 4 or more even number of watchdog cluster users will benefit from this parameter is off because now it prevents possible split brain when a half of watchdog nodes go down. 

This kind of error could happen for multiple reasons. Here "52" is ASCII code point
in hexa decimal number, that is ASCII 'R'. 'R' is normal response from backend.
"45" is 'E' in ASCII, which means PostgreSQL complains something. In summary backend
0 accepts the connection request normaly, while backend 1 complains. To solve the
problem, you need to look into pgpool.log. For example, If you set `reject` entry
for the connection request in backend 1's `pg_hba.conf`:

```
local	all	foo	reject
```

and try to connect to pgpool, you will get the error.You should be able to find
something like below in pgpool log:

```
 2021-05-23 15:38:38: child pid 375: LOG:  pool_read_kind: error message from 1 th backend:pg_hba.conf rejects connection for host "[local]", user "foo", database "test", no encryption
 2021-05-23 15:38:38: child pid 375: ERROR:  unable to read message kind
 2021-05-23 15:38:38: child pid 375: DETAIL:  kind does not match between main(52) slot[1] (45)
```

You find that you need to fix pg_hba.conf of backend 1.

Other types of errors include:

* Backend 1's pg_hba.conf setting refuses the connection from pgpool
* max_connections parameter of PostgreSQL is not identical among backends

Note that "main" in the error message is "master" in Pgpool-II 4.1 or before.
Also note that the detailed error info ("error message from 1 th backend:...") is
not available in Pgpool-II 3.6 or before.

### I am getting an authentication error when Pgpool-II connects to Azure PostgreSQL. Why?

Auzre PostgreSQL only accepts clear text password. md5 authentication nor
SCRAM-SHA-256 cannot be used. You need to set clear text password in pool_passwd.

related bug track entries:
* <https://www.pgpool.net/mantisbt/view.php?id=737>
* <https://www.pgpool.net/mantisbt/view.php?id=699>

### Why does "show pool_nodes" show replication_delay as "0" even when the standby is not up-to-date? Why doesn't "show pool_nodes" show replication_state and replication_sync_state?

There are two possible reasons.

First one is sr_check_user does not have enough previlege to query against
`pg_stat_replication` view. `sr_check_user` needs to be PostgreSQL superuser or
in `pg_monitor` group. Please consult [PostgreSQL manual](https://www.postgresql.org/docs/current/monitoring-stats.html#MONITORING-STATS-VIEWS) for more details.

Another reason can be the setting of backend_application_name parameter of the
standby node in `pgpool.conf`. It must match with the `application_name` in
`primary_coninfo` parameter in `postgresql.conf`.

### For client authentication I want to avoid maintaining pool_passwd file. What's the recommended way to do that?

See this email thread.

[[pgpool-general: 8897] pgpool forwarding database users/passwords](https://www.pgpool.net/pipermail/pgpool-general/2023-August/008897.html)


### When I enable SSL, Pgpool-II eats CPU lot. Why?

Pgpool-II uses OpenSSL. It has known performance issues with 3.0.2.
Some OS use the version of OpenSSL, including Ubuntu 22.04 and 24.04.
The issue of OpenSSL has been fixed in 3.1 but never backported to 3.0.
In Ubuntu 24.10 the issue has been fixed but 24.10 is not LTS.

This original information is provided [here](https://github.com/pgpool/pgpool2/issues/93#issuecomment-2691037744).

