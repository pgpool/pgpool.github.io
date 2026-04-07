---
title: "Pgpool-II Yum Repository"
permalink: /download/yum/
sidebar:
  nav: "download"
excerpt: "Pgpool-II yum repository."
last_modified_at: 2026-03-26
toc: true
layout: single
---

PgPool Global Development Group provides the [Ppgpool-II Yum Repository](/yum/). 

With this repository, you can install pgpool-II by using yum.
Please visit [documentation](https://www.pgpool.net/docs/latest/en/html/install-rpm.html)
for more details of RPM installation.

## RPM Packaging Policy

### Supported Pgpool-II Versions

RPM packages are provided for the stable versions of Pgpool-II ([EOL information](/eol/).
Typically, the latest five major versions are supported.)

### Supported Platforms (Operating Systems)

RPM packages are provided for the following Red Hat–based distributions (including compatible distributions such as Rocky Linux).

* RHEL / Rocky Linux 8
* RHEL / Rocky Linux 9
* RHEL / Rocky Linux 10

### Supported PostgreSQL Versions

For each release of Pgpool-II (major and minor), RPM packages are provided for all
[https://www.postgresql.org/support/versioning/ supported major versions of PostgreSQL].

For example, as of the release of Pgpool-II 4.6.4 on Nov 25, 2025, PostgreSQL versions 14 through 18 are supported. Therefore, the following RPMs are available for each
corresponding PostgreSQL major version: 
* pgpool-II-pg14-4.6.4  
* pgpool-II-pg15-4.6.4  
* pgpool-II-pg16-4.6.4  
* pgpool-II-pg17-4.6.4
* pgpool-II-pg18-4.6.4

## Install

### 1. Install the repository RPM
To use the yum repository, you must first install the repository RPM. To do this, download the correct RPM from the repository RPM listing, and install it with commands like:

<div class="tabs">

<input type="radio" name="os" id="rhel10" checked>
<label for="rhel10">RHEL / Rocky 10.x</label>

<input type="radio" name="os" id="rhel9">
<label for="rhel9">RHEL / Rocky 9.x</label>

<input type="radio" name="os" id="rhel8">
<label for="rhel8">RHEL / Rocky 8.x</label>

<div class="tab-content" id="content10">
<h4 class="version-label">Pgpool-II 4.7</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.7/redhat/rhel-10-x86_64/pgpool-II-release-4.7-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.6</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.6/redhat/rhel-10-x86_64/pgpool-II-release-4.6-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.5</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.5/redhat/rhel-10-x86_64/pgpool-II-release-4.5-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.4</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.4/redhat/rhel-10-x86_64/pgpool-II-release-4.4-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.3</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.3/redhat/rhel-10-x86_64/pgpool-II-release-4.3-1.noarch.rpm
{% endhighlight %}
</div>

<div class="tab-content" id="content9">
<h4 class="version-label">Pgpool-II 4.7</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.7/redhat/rhel-9-x86_64/pgpool-II-release-4.7-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.6</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.6/redhat/rhel-9-x86_64/pgpool-II-release-4.6-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.5</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.5/redhat/rhel-9-x86_64/pgpool-II-release-4.5-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.4</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.4/redhat/rhel-9-x86_64/pgpool-II-release-4.4-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.3</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.3/redhat/rhel-9-x86_64/pgpool-II-release-4.3-1.noarch.rpm
{% endhighlight %}
</div>

<div class="tab-content" id="content8">
<h4 class="version-label">Pgpool-II 4.7</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.7/redhat/rhel-8-x86_64/pgpool-II-release-4.7-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.6</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.6/redhat/rhel-8-x86_64/pgpool-II-release-4.6-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.5</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.5/redhat/rhel-8-x86_64/pgpool-II-release-4.5-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.4</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.4/redhat/rhel-8-x86_64/pgpool-II-release-4.4-1.noarch.rpm
{% endhighlight %}
<h4 class="version-label">Pgpool-II 4.3</h4>
{% highlight bash %}
sudo dnf install https://www.pgpool.net/yum/rpms/4.3/redhat/rhel-8-x86_64/pgpool-II-release-4.3-2.noarch.rpm
{% endhighlight %}
</div>

</div>


### 2. Install Pgpool-II

Once installing the repository RPM is done, you can proceed to install and update
the Pgpool-II packages from the offcial repository.

Package names in the Pgpool-II yum epository includes the version number of
PostgreSQL to use with pgpool-II, such as: 

```bash
sudo dnf install pgpool-II-pg18 pgpool-II-pg18-extensions
```
