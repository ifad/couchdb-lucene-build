= IFAD build of CouchDB Lucene

This repository contains a Java bytecode build of
[couchdb-lucene](https://github.com/rnewson/couchdb-lucene) at version
[bc1bd8f](https://github.com/rnewson/couchdb-lucene/tree/bc1bd8f).

It is used by internal [IFAD](http://www.ifad.org) projects, but it can be
used by anyone finds this binary build useful - of course! :-).


= Issue Tracking

Issue tracking at [github](http://github.com/ifad/couchdb-lucene-build/issues)


= Minimum System Requirements

Java 1.5 (or above) is required; the <strike>Sun</strike> Oracle version is
recommended as it's regularly tested against.

= The Big Picture

couchdb-lucene parses `conf/couchdb-lucene.ini`, listens onto the configured
TCP socket and connects to the configured couchdb servers, in order to use
CouchDB's `_changes` facility to keep the lucene index up to date.

CouchDB spawns the `tools/couchdb-external-hook.py` script that connects to
the `couchdb-lucene` socket and provides full-text indexing services under
an HTTP path - `_fti` by default.

= Development: how to get up and running quickly

== Download the couchdb lucene daemon

    git clone https://github.com/ifad/couchdb-lucene-build

== Configure CouchDB

Copy `tools/etc/couchdb/local.d/lucene.ini` to your `couchdb/local.d`
directory, updating the paths in the `fti` directive with the correct
ones for your system.

If couchdb-lucene and couchdb run on different machines, you'll need to add
some options to the fti hook. See the aforementioned config file and the
`conf/couchdb-lucene.ini` file for details.

By default couchdb-lucene does not attempt to authenticate to CouchDB. If your
CouchDB's requires it, then set the credentials in the `couchdb-lucene.ini`
configuration file:

    [local]
    url=http://foo:bar@localhost:5984/

Then, restart CouchDB and you're set.

== Run it

Enter the cloned repository directory and execute `./bin/run`

    cd couchdb-lucene-build
    ./bin/run

The standalone lucene JVM will remain in the foreground, kill it with a `^C`
or via a `TERM` signal.

For actual configuration and usage please refer to the upstream README [on
GitHub](https://github.com/rnewson/couchdb-lucene). It all boils down to a
view creation under the `_fti` namespace and then querying it as usual via
HTTP.

= Deployment

TODO

= Maintenance

For optimal query speed you can optimize your indexes. This causes the index
to be rewritten into a single segment.

    curl -X POST http://localhost:5984/<databasename>/_fti/_design/foo/<indexname>/_optimize

If you just want to expunge pending deletes:

    curl -X POST http://localhost:5984/<databasename>/_fti/_design/foo/<indexname>/_expunge

If you recreate databases or frequently change your fulltext functions, you
will probably have old indexes lying around on disk. To remove all of them:

    curl -X POST http://localhost:5984/<databasename>/_fti/_cleanup
