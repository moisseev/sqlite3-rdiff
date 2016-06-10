# sqlite3-rdiff

compute and apply signature-based row differences for SQLite3 databases

## Depends

- Tcl (and lang/tcl-wrapper for FreeBSD)
- SQLite extension for Tcl
- SQLite3
- [sqlite-md5 extension library for SQLite](https://github.com/moisseev/sqlite-md5)

## Description

Signature is computed by md5 algorithm for each full row. Hash collisions are
resolved by using different --rows-per-hash for some replication session.
The result-file can be same as the old-file in patch mode.
 
## Synopsis

~~~
sqlite3-rdiff [--table-name expression] [--rows-per-hash N] signature old-database signature-file

sqlite3-rdiff [--table-name expression] delta signature-file new-database delta-file

sqlite3-rdiff [--table-name expression] [--multimaster] patch old-database delta-file result-database

sqlite3-rdiff [--table-name expression] analyze old-database new-database
~~~

## Options

~~~
--table-name    regular expression for table names, use ? for any single symbol
                and % for any set of symbols.
~~~
~~~
--rows-per-hash how much rows are hashed at once. The default value is 1.
                Increase it for small signature and decrease for small delta.
                Use analyze mode to determine the best value of this parameter.
                For master-slave replication any value will be safe.
                This option is used only in signature building mode.
~~~
~~~
--multimaster   only insert nonexisting records. May produce duplicates when
                a table has no unique column or indices with "replace" or
               "ignore" conflict resolution algorithm and --rows-per-hash
                option is greater than 1.
                Useful for master-master replication on the inserts-only
                populated databases.
~~~

## Usage

The slave.db is an outdated version of master.db database. The commands below will sync slave.db to actual version of master.db.

~~~
./sqlite3-rdiff --rows-per-hash 10 signature slave.db slave.db.sign
./sqlite3-rdiff delta slave.db.sign master.db slave.db.delta
./sqlite3-rdiff patch slave.db slave.db.delta slave.db
~~~

Note: we can patch slave.db on-the-place by setting last parameter for "sqlite3-rdiff patch" to the slave.db or create a new database by setting last parameter different.

## The --rows-per-hash option

See below test results with different --rows-per-hash values on 52 Mb database.

## Calculation optimal --rows-per-hash value

~~~
$ ./sqlite3-rdiff analyze slave.db master.db
analyze slave.db master.db --table-name %
52774   old-database size (kB)
53919   new-database size (kB)
Calculating optimal --rows-per-hash value ...................
18      optimal --rows-per-hash option value
2936    signature+delta size (kB)
~~~

![sizes from --rows_per_hash graph](https://github.com/moisseev/sqlite3-rdiff/raw/master/sizes_from_rows_per_hash.png)

## License

This code in Public Domain

## Acknowledgements

The code is based heavily on the [now deprecated](http://sqlite.mobigroup.ru/finfo?name=util/sqlite3-rdiff) [MBG SQLite's](http://sqlite.mobigroup.ru) [sqlite3-rdiff](http://sqlite.mobigroup.ru/wiki?name=sqlite3-rdiff) utility by Alexey Pechnikov <pechnikov@mobigroup.ru>.

## See also

[The sqlite3-rdiff web page at MBG SQLite project wiki](http://sqlite.mobigroup.ru/wiki?name=sqlite3-rdiff)

[sqlite3-rdiff: master-slave replication for SQLite](http://geomapx.blogspot.ru/2009/12/sqlite3-rdiff-master-slave-replication.html)

[The small signature for sqlite3-rdiff](http://geomapx.blogspot.com/2009/12/small-signature-for-sqlite3-rdiff.html)

[MurmurHash 2.0 for SQLite](http://geomapx.blogspot.com/2009/12/murmurhash-20.html)
