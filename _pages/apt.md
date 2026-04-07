---
title: "Apt Repository"
permalink: /download/apt/
sidebar:
  nav: "download"
excerpt: "Pgopol-II apt repository."
last_modified_at: 2026-03-26
toc: false
layout: single
---

Pgpool-II Debian/Ubuntu packages are available through the [PostgreSQL apt repository](https://wiki.postgresql.org/wiki/Apt). 

To use the apt repository, follow these steps: 

* Create the file repository configuration:

```bash
 sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
```

* Import the repository signing key:

```bash
 wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
```

* Update the package lists:

```bash
 sudo apt-get update
```

* Install the latest version of Pgpool-II. Specify the specific version of PostgreSQL which you are using, e.g. postgresql-15-pgpool2:

```bash
 sudo apt-get -y install pgpool2 libpgpool2 postgresql-15-pgpool2
```

