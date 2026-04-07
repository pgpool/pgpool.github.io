---
title: "<i class='fas fa-book-open'></i> Introduction"
permalink: /about/
sidebar:
  nav: "about"
excerpt: "Middleware for PostgreSQL with connection pooling, load balancing, and high availability."
last_modified_at: 2026-03-26
toc: true
layout: single
---

## What is Pgpool-II

Pgpool-II is a middleware that works between [PostgreSQL](https://postgresql.org/) servers and a PostgreSQL database client. Pgpool-II speaks PostgreSQL's backend and frontend protocol, and relays messages between a backend and a frontend. Therefore, a database application (frontend) thinks that Pgpool-II is the actual PostgreSQL server, and the server (backend) sees Pgpool-II as one of its clients. Because Pgpool-II is transparent to both the server and the client, an existing database application can be used with Pgpool-II almost without a change to its sources.

It is distributed under a [license](/license/) similar to BSD and MIT.

## Features

Pgpool-II provides the following features. 

- **High Availability**

Pgpool-II provides a high availability (HA) feature by using multiple PostgreSQL servers so that it automatically removes broken server from the server pool to continue the database task. This is called automatic failover. Also Pgpool-II offers an HA feature for Pgpool-II itself, called Watchdog (see Chapter 4 for more details). Moreover Pgpool-II hires sophisticated quorum algorithm to avoid false positive errors and split brain problem to make the whole HA system highly reliable. See Section 5.15.6 for more details. 

- **Connection Pooling**

Pgpool-II saves connections to the PostgreSQL servers, and reuse them whenever a new connection with the same properties (i.e. username, database, protocol version) comes in. It reduces connection overhead, and improves system's overall throughput.

- **Load Balancing**

If a database is replicated, executing a SELECT query on any server will return the same result.

Pgpool-II takes an advantage of the replication feature to reduce the load on each PostgreSQL server by distributing SELECT queries among multiple servers, improving system's overall throughput. At best, performance improves proportionally to the number of PostgreSQL servers. Load balance works best in a situation where there are a lot of users executing many queries at the same time.

Besides these essential features, Pgpool-II also provides useful features such as: 

- In Memory Query Cache
- Online Recovery
- Watchdog
- Limiting Exceeding Connections
- Replication

For more details, please refer to the [documentation](https://www.pgpool.net/docs/latest/en/html/intro-whatis.html).

