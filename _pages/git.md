---
title: "Git Repository"
permalink: /download/git/
sidebar:
  nav: "download"
excerpt: "Pgpool-II git repository."
last_modified_at: 2026-03-26
toc: true
layout: single
---

Pgpool source code repository is managed by [PostgreSQL's git repository](https://git.postgresql.org/gitweb).
* [Pgpool-II git repository](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=summary)
* [pgpool-I git repository](https://git.postgresql.org/gitweb/?p=pgpool1.git;a=summary) (legacy version of pgpool. Not maintained anymore)

## About pgpool-II source code management

### Pgpool-II version policy

We release **major version** each year, which is mostly backward compatible with
the previous release, Exceptions to this rule are listed in the release notes when sometimes some existing feature is dropped, depreciated or a change that brings
certain incompatibilities with the older versions. The versions are numbered in
the form of A.B.C where A.B is the major version number while C represents the
minor version of a release. Currently pgpool-II versions are represented by 
A == 3 i.e 3.B.C where 3.B represents the major version number, So current
releases of pgpool-II is also sometimes referred as "3. X series".

To absolutely identify version number, specifying the "minor version number (part
C of A.B.C version string)" is important. This minor version is incremented for
the bug fix release, which is roughly released every month. For example
pgpool-II 3.5.0 represents the first release of pgpool-II 3.5 series and 3.5.1 is
the second minor (bug fix) release of the same. Unlike major versions the new minor
release is guaranteed to be 100% backward compatible with the previous minor
release.

### Pgpool-II branch policy

Pgpool-II source code is managed by git and hosted at
[PostgreSQL's git repository](https://git.postgresql.org/gitweb).

Currently we have several branches to track each development tree.
The under development branch (the candidate for next major version) is
[master branch](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=summary).

Each time new major version is released, we create a branch to maintain
the previous major version. We call them as **stable branche**.
Currently we have following stable branches.

* [4.7-stable](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=shortlog;h=refs/heads/V4_7_STABLE)
* [4.6-stable](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=shortlog;h=refs/heads/V4_6_STABLE)
* [4.5-stable](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=shortlog;h=refs/heads/V4_5_STABLE)
* [4.4-stable](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=shortlog;h=refs/heads/V4_4_STABLE)
* [4.3-stable](https://git.postgresql.org/gitweb/?p=pgpool2.git;a=shortlog;h=refs/heads/V4_3_STABLE)
