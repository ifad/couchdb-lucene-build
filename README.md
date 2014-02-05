IFAD build of CouchDB Lucene
============================

This repository contains a Java bytecode build of
[ifad/couchdb-lucene](https://github.com/ifad/couchdb-lucene) at version
[4cbeef84](https://github.com/ifad/couchdb-lucene/tree/4cbeef84).

It is used by internal [IFAD](http://www.ifad.org) projects, but it can be
used by anyone finds this binary build useful - of course! :-).

If you use OpenSuSE, we've built an RPM package on [the OpenSuSE Build
Service](https://build.opensuse.org/package/show/home:vjt:ifad/couchdb-lucene).

Minimum System Requirements
===========================

Java 1.6 (or above) is required; the <strike>Sun</strike> Oracle version is
recommended as it's regularly tested against. If you're on OSX &gt;= 10.5
you're set, if you're on Debian install `openjdk-7-jre`, if you're on SuSE
install `java-1_7_0-openjdk`.

The Big Picture
===============

couchdb-lucene parses `conf/couchdb-lucene.ini`, listens onto the configured
TCP socket and connects to the configured couchdb servers, in order to use
CouchDB's `_changes` facility to keep the lucene index up to date.

CouchDB uses a `proxy_handler` to connect to `couchdb-lucene`'s socket and
provides full-text indexing services under an HTTP path - `/_fti` by default.

Development: how to get up and running quickly
==============================================

Download the couchdb lucene build
---------------------------------

    git clone https://github.com/ifad/couchdb-lucene-build.git

Configure CouchDB
-----------------

Copy `vendor/couchdb/local.d/lucene.ini` to your `couchdb/local.d`
directory. Restart CouchDB. All set.

Configure couchdb-lucene
------------------------

Copy `conf/couchdb-lucene.ini.example` into `conf/couchdb-lucene.ini` and edit
as will. The default configuration listens on localhost, port 5985 and assumes
that a CouchDB instance is running on localhost, without requiring a password.

Run couchdb-lucene
------------------

Enter the cloned repository directory and execute `./bin/run`

    cd couchdb-lucene-build
    ./bin/run

The standalone lucene JVM will remain in the foreground, kill it with a `^C`
or via a `TERM` signal.

For actual configuration and usage please refer to the upstream README [on
GitHub](https://github.com/rnewson/couchdb-lucene). It all boils down to a
view creation under the `_fti` namespace and then querying it as usual via
HTTP.


Distribution and authentication
-------------------------------

If couchdb-lucene and couchdb run on different machines, just specify the
CouchDB URL in the `conf/couchdb-lucene.ini` file, and CouchDB-Lucene URL
in the `local.d/lucene.ini` file.

By default couchdb-lucene does not attempt to authenticate to CouchDB. If your
CouchDB's requires it, then set the credentials in the `couchdb-lucene.ini`
configuration file:

    [local]
    url=http://foo:bar@localhost:5984/


Deployment
==========

For systemd units, have a look at [the prebuilt OpenSuSE package](
https://build.opensuse.org/package/show/home:vjt:ifad/couchdb-lucene).

For sysvinit, there are two scripts in the `vendor/init.d directory`, one
for Debian systems and one for SuSE ones.

Both of them read an `/etc/default/couchdb-lucene` file if it exists.
If you checked out this repository in directories other than
`/opt/couchdb-lucene-build`, then you'll need to adjust the `DAEMON`
variable and make it point to the location of the `bin/run` script.

The daemon is started as user `daemon` by default, you can change it
by setting the `USER` variable in the `/etc/default/couchdb-lucene`
file. Make sure that the user has write access to the `indexes` and
`logs` directories.

You can configure `indexes` directory in the `conf/couchdb-lucene.ini`
configuration file, while the location of the log file is configured
in the `conf/log4j.xml` file.

Of course, the easiest way is to use symlinks:

    mkdir -p /var/{lib,log}/couchdb-lucene
    chown daemon: /var/{lib,log}/couchdb-lucene
    ln -s /var/lib/couchdb-lucene indexes
    ln -s /var/log/couchdb-lucene logs

Maintenance
===========

For optimal query speed you can optimize your indexes. This causes the index
to be rewritten into a single segment.

    curl -X POST http://localhost:5984/<databasename>/_fti/_design/foo/<indexname>/_optimize

If you just want to expunge pending deletes:

    curl -X POST http://localhost:5984/<databasename>/_fti/_design/foo/<indexname>/_expunge

If you recreate databases or frequently change your fulltext functions, you
will probably have old indexes lying around on disk. To remove all of them:

    curl -X POST http://localhost:5984/<databasename>/_fti/_cleanup


Issue Tracking
==============

Issue tracking at [github](http://github.com/ifad/couchdb-lucene-build/issues)


Acknowledgments
===============

* [Robert Newson](https://github.com/rnewson) for the excellent couchdb-lucene
  software
* [Amedeo Paglione](https://github.com/amedeo) for the endless CouchDB pushing
  here at IFAD
* [Simone Carletti](https://github.com/weppos) for the endless CouchDB bashing:
  you always need a second opinion! :-)

`EOF`
