LevelDB [![Build Status](https://img.shields.io/travis/snowyu/node-nosql-leveldb/master.svg)](http://travis-ci.org/snowyu/node-nosql-leveldb) 
=================


[![npm](https://img.shields.io/npm/v/nosql-leveldb.svg)](https://npmjs.org/package/nosql-leveldb) [![downloads](https://img.shields.io/npm/dm/nosql-leveldb.svg)](https://npmjs.org/package/nosql-leveldb) [![license](https://img.shields.io/npm/l/nosql-leveldb.svg)](https://npmjs.org/package/nosql-leveldb) 

![LevelDB Logo](https://camo.githubusercontent.com/35c59b1270df115851d8f50b57c7eb2d916f61d4/68747470733a2f2f302e67726176617461722e636f6d2f6176617461722f6134393862313232616563623736373834393061333862623539336363313264)

A Low-level Node.js LevelDB binding
-------------------------

Leveldown was extracted from [LevelUP](https://github.com/rvagg/node-levelup) and now serves as a stand-alone binding for LevelDB.

nosql-leveldb was modified from [LevelDown](https://github.com/rvagg/node-leveldown). see changes below for differencement list.

It is **strongly recommended** that you use LevelUP-sync in preference to LevelDB unless you have measurable performance reasons to do so. LevelUP is optimised for usability and safety. Although we are working to improve the safety of the LevelDB interface it is still easy to crash your Node process if you don't do things in just the right way.

See the section on <a href="#safety">safety</a> below for details of known unsafe operations with LevelDB.

<a name="platforms"></a>
Tested & supported platforms
----------------------------

  * **Linux** (including ARM platforms such as Raspberry Pi *and Kindle!*)
  * **Mac OS**
  * **Solaris** (SmartOS & Nodejitsu)
  * **FreeBSD**
  * **Windows**
    * Node 0.10 and above only, see [issue #5](https://github.com/rvagg/node-levelup/issues/5) for more info
    * See installation instructions for *node-gyp* dependencies [here](https://github.com/TooTallNate/node-gyp#installation), you'll need these (free) components from Microsoft to compile and run any native Node add-on in Windows.

## Chnages

### v2.x.x

+ the modularization(feature plugin) with [abstract-nosql](https://github.com/snowyu/node-abstract-nosql)
- (`broken changes`) remove the streamable feature from buildin. this is a plugin now.
- (`broken change`) defaults to disable asBuffer option.
  * pls use the `getBuffer` method to get as buffer.

### v1.x.x

+ Add the AbstractError and error code supports.
+ Add the isExists supports to test the key whether exists.
+ Add the synchronous methods supports for open, close, put, get, del, batch, approximateSize.
  * the methods will be executed as sync method if no callback argument passed.
  * if any error occurs it will throw exception.
  * Or call openSync, putSync, getSync, delSync, batchSync, approximateSizeSync directly.
+ Add iterator.nextSync, iterator.endSync synchronous methods
* Optimize the iterator performance and fix memory leaks.
* Upgrade the LevelDB to 1.18 and Snappy to 1.1.2
* Add mGetSync to multi get keys for better performance.
  * see [abstract-nosql](https://github.com/snowyu/node-abstract-nosql) for more details
+ Add GetBufferSync to get as buffer.

## Performance

run bench/bench.js to see a simple performance compare:

the original Leveldown bench result:

```bash

  benchmarking with 12,000 records, 24 chars each

          put :    29,411 w/s in    408ms

        batch :   101,694 w/s in    118ms

          get :    35,398 r/s in    339ms        (asBuffer)
          get :    44,444 r/s in    270ms

     iterator :    58,823 r/s in    204ms        (asBuffer)
     iterator :   127,659 r/s in     94ms
     iterator :   148,148 r/s in     81ms        (directly)

```

The new nosql-leveldb bench result:

```bash

benchmarking with 12,000 records, 24 chars each

          put :    30,000 w/s in    400ms
      putSync :    74,074 w/s in    162ms
      putSync :    77,922 w/s in    154ms        (directly)

        batch :   121,212 w/s in     99ms
    batchSync :   179,104 w/s in     67ms
    batchSync :   200,000 w/s in     60ms        (directly)

          get :    36,036 r/s in    333ms        (asBuffer)
          get :    56,338 r/s in    213ms
      getSync :    79,470 r/s in    151ms        (asBuffer)
      getSync :   255,319 r/s in     47ms
      getSync :   260,869 r/s in     46ms        (directly)

     iterator :    94,488 r/s in    127ms        (asBuffer)
     iterator :   153,846 r/s in     78ms
     iterator :   342,857 r/s in     35ms        (directly)
 iteratorSync :   292,682 r/s in     41ms
 iteratorSync :   300,000 r/s in     40ms        (directly)

```


<a name="api"></a>
## API

  * <a href="#ctor"><code><b>LevelDB()</b></code></a>
  * <a href="#LevelDB_open"><code><b>LevelDB#open()</b></code></a>
  * <a href="#LevelDB_close"><code><b>LevelDB#close()</b></code></a>
  * <a href="#LevelDB_put"><code><b>LevelDB#put()</b></code></a>
  * <a href="#LevelDB_get"><code><b>LevelDB#get()</b></code></a>
  * <a href="#LevelDB_get"><code><b>LevelDB#getBuffer()</b></code></a>
  * <a href="#LevelDB_get"><code><b>LevelDB#mGet()</b></code></a>
  * <a href="#LevelDB_del"><code><b>LevelDB#del()</b></code></a>
  * <a href="#LevelDB_batch"><code><b>LevelDB#batch()</b></code></a>
  * <a href="#LevelDB_approximateSize"><code><b>LevelDB#approximateSize()</b></code></a>
  * <a href="#LevelDB_getProperty"><code><b>LevelDB#getProperty()</b></code></a>
  * <a href="#LevelDB_iterator"><code><b>LevelDB#iterator()</b></code></a>
  * <a href="#iterator_next"><code><b>iterator#next()</b></code></a>
  * <a href="#iterator_end"><code><b>iterator#end()</b></code></a>
  * <a href="#LevelDB_destroy"><code><b>LevelDB.destroy()</b></code></a>
  * <a href="#LevelDB_repair"><code><b>LevelDB.repair()</b></code></a>


--------------------------------------------------------
<a name="ctor"></a>
### LevelDB(location)
<code>LevelDB()</code> returns a new **LevelDB** instance. `location` is a String pointing to the LevelDB location to be opened.


--------------------------------------------------------
<a name="LevelDB_open"></a>
### LevelDB#open([options, ]callback)
<code>open()</code> is an instance method on an existing database object.

The `callback` function will be called with no arguments when the database has been successfully opened, or with a single `error` argument if the open operation failed for any reason.

#### `options`

The optional `options` argument may contain:

* `'createIfMissing'` *(boolean, default: `true`)*: If `true`, will initialise an empty database at the specified location if one doesn't already exist. If `false` and a database doesn't exist you will receive an error in your `open()` callback and your database won't open.

* `'errorIfExists'` *(boolean, default: `false`)*: If `true`, you will receive an error in your `open()` callback if the database exists at the specified location.

* `'compression'` *(boolean, default: `true`)*: If `true`, all *compressible* data will be run through the Snappy compression algorithm before being stored. Snappy is very fast and shouldn't gain much speed by disabling so leave this on unless you have good reason to turn it off.

* `'cacheSize'` *(number, default: `8 * 1024 * 1024` = 8MB)*: The size (in bytes) of the in-memory [LRU](http://en.wikipedia.org/wiki/Cache_algorithms#Least_Recently_Used) cache with frequently used uncompressed block contents. 

**Advanced options**

The following options are for advanced performance tuning. Modify them only if you can prove actual benefit for your particular application.

* `'writeBufferSize'` *(number, default: `4 * 1024 * 1024` = 4MB)*: The maximum size (in bytes) of the log (in memory and stored in the .log file on disk). Beyond this size, LevelDB will convert the log data to the first level of sorted table files. From the LevelDB documentation:

> Larger values increase performance, especially during bulk loads. Up to two write buffers may be held in memory at the same time, so you may wish to adjust this parameter to control memory usage. Also, a larger write buffer will result in a longer recovery time the next time the database is opened.

* `'blockSize'` *(number, default `4096` = 4K)*: The *approximate* size of the blocks that make up the table files. The size related to uncompressed data (hence "approximate"). Blocks are indexed in the table file and entry-lookups involve reading an entire block and parsing to discover the required entry.

* `'maxOpenFiles'` *(number, default: `1000`)*: The maximum number of files that LevelDB is allowed to have open at a time. If your data store is likely to have a large working set, you may increase this value to prevent file descriptor churn. To calculate the number of files required for your working set, divide your total data by 2MB, as each table file is a maximum of 2MB. 

* `'blockRestartInterval'` *(number, default: `16`)*: The number of entries before restarting the "delta encoding" of keys within blocks. Each "restart" point stores the full key for the entry, between restarts, the common prefix of the keys for those entries is omitted. Restarts are similar to the concept of keyframs in video encoding and are used to minimise the amount of space required to store keys. This is particularly helpful when using deep namespacing / prefixing in your keys.


--------------------------------------------------------
<a name="LevelDB_close"></a>
### LevelDB#close(callback)
<code>close()</code> is an instance method on an existing database object. The underlying LevelDB database will be closed and the `callback` function will be called with no arguments if the operation is successful or with a single `error` argument if the operation failed for any reason.


--------------------------------------------------------
<a name="LevelDB_put"></a>
### LevelDB#put(key, value[, options], callback)
<code>put()</code> is an instance method on an existing database object, used to store new entries, or overwrite existing entries in the LevelDB store.

The `key` and `value` objects may either be `String`s or Node.js `Buffer` objects. Other object types are converted to JavaScript `String`s with the `toString()` method. Keys may not be `null` or `undefined` and objects converted with `toString()` should not result in an empty-string. Values of `null`, `undefined`, `''`, `[]` and `new Buffer(0)` (and any object resulting in a `toString()` of one of these) will be stored as a zero-length character array and will therefore be retrieved as either `''` or `new Buffer(0)` depending on the type requested.

A richer set of data-types are catered for in LevelUP.

#### `options`

The only property currently available on the `options` object is `'sync'` *(boolean, default: `false`)*. If you provide a `'sync'` value of `true` in your `options` object, LevelDB will perform a synchronous write of the data; although the operation will be asynchronous as far as Node is concerned. Normally, LevelDB passes the data to the operating system for writing and returns immediately, however a synchronous write will use `fsync()` or equivalent so your callback won't be triggered until the data is actually on disk. Synchronous filesystem writes are **significantly** slower than asynchronous writes but if you want to be absolutely sure that the data is flushed then you can use `'sync': true`.

The `callback` function will be called with no arguments if the operation is successful or with a single `error` argument if the operation failed for any reason.


--------------------------------------------------------
<a name="LevelDB_get"></a>
### LevelDB#get(key[, options], callback)
<code>get()</code> is an instance method on an existing database object, used to fetch individual entries from the LevelDB store.

The `key` object may either be a `String` or a Node.js `Buffer` object and cannot be `undefined` or `null`. Other object types are converted to JavaScript `String`s with the `toString()` method and the resulting `String` *may not* be a zero-length. A richer set of data-types are catered for in LevelUP.

Values fetched via `get()` that are stored as zero-length character arrays (`null`, `undefined`, `''`, `[]`, `new Buffer(0)`) will return as empty-`String` (`''`) or `new Buffer(0)` when fetched with `asBuffer: true` (see below).

#### `options`

The optional `options` object may contain:

* `'fillCache'` *(boolean, default: `true`)*: LevelDB will by default fill the in-memory LRU Cache with data from a call to get. Disabling this is done by setting `fillCache` to `false`.

* `'asBuffer'` *(boolean, default: `true`)*: Used to determine whether to return the `value` of the entry as a `String` or a Node.js `Buffer` object. Note that converting from a `Buffer` to a `String` incurs a cost so if you need a `String` (and the `value` can legitimately become a UFT8 string) then you should fetch it as one with `asBuffer: true` and you'll avoid this conversion cost.

The `callback` function will be called with a single `error` if the operation failed for any reason. If successful the first argument will be `null` and the second argument will be the `value` as a `String` or `Buffer` depending on the `asBuffer` option.


--------------------------------------------------------
<a name="LevelDB_del"></a>
### LevelDB#del(key[, options], callback)
<code>del()</code> is an instance method on an existing database object, used to delete entries from the LevelDB store.

The `key` object may either be a `String` or a Node.js `Buffer` object and cannot be `undefined` or `null`. Other object types are converted to JavaScript `String`s with the `toString()` method and the resulting `String` *may not* be a zero-length. A richer set of data-types are catered for in LevelUP.

#### `options`

The only property currently available on the `options` object is `'sync'` *(boolean, default: `false`)*. See <a href="#LevelDB_put">LevelDB#put()</a> for details about this option.

The `callback` function will be called with no arguments if the operation is successful or with a single `error` argument if the operation failed for any reason.


--------------------------------------------------------
<a name="LevelDB_batch"></a>
### LevelDB#batch(operations[, options], callback)
<code>batch()</code> is an instance method on an existing database object. Used for very fast bulk-write operations (both *put* and *delete*). The `operations` argument should be an `Array` containing a list of operations to be executed sequentially, although as a whole they are performed as an atomic operation inside LevelDB. Each operation is contained in an object having the following properties: `type`, `key`, `value`, where the *type* is either `'put'` or `'del'`. In the case of `'del'` the `'value'` property is ignored. Any entries with a `'key'` of `null` or `undefined` will cause an error to be returned on the `callback`. Any entries where the *type* is `'put'` that have a `'value'` of `undefined`, `null`, `[]`, `''` or `new Buffer(0)` will be stored as a zero-length character array and therefore be fetched during reads as either `''` or `new Buffer(0)` depending on how they are requested.

See [LevelUP](https://github.com/rvagg/node-levelup#batch) for full documentation on how this works in practice.

#### `options`

The only property currently available on the `options` object is `'sync'` *(boolean, default: `false`)*. See <a href="#LevelDB_put">LevelDB#put()</a> for details about this option.

The `callback` function will be called with no arguments if the operation is successful or with a single `error` argument if the operation failed for any reason.


--------------------------------------------------------
<a name="LevelDB_approximateSize"></a>
### LevelDB#approximateSize(start, end, callback)
<code>approximateSize()</code> is an instance method on an existing database object. Used to get the approximate number of bytes of file system space used by the range `[start..end)`. The result may not include recently written data.

The `start` and `end` parameters may be either `String` or Node.js `Buffer` objects representing keys in the LevelDB store.

The `callback` function will be called with no arguments if the operation is successful or with a single `error` argument if the operation failed for any reason.


--------------------------------------------------------
<a name="LevelDB_getProperty"></a>
### LevelDB#getProperty(property)
<code>getProperty</code> can be used to get internal details from LevelDB. When issued with a valid property string, a readable string will be returned (this method is synchronous).

Currently, the only valid properties are:

* <b><code>'leveldb.num-files-at-levelN'</code></b>: return the number of files at level *N*, where N is an integer representing a valid level (e.g. "0").

* <b><code>'leveldb.stats'</code></b>: returns a multi-line string describing statistics about LevelDB's internal operation.

* <b><code>'leveldb.sstables'</code></b>: returns a multi-line string describing all of the *sstables* that make up contents of the current database.


--------------------------------------------------------
<a name="LevelDB_iterator"></a>
### LevelDB#iterator([options])
<code>iterator()</code> is an instance method on an existing database object. It returns a new **Iterator** instance.

#### `options`

The optional `options` object may contain:

* `'gt'` (greater than), `'gte'` (greater than or equal) define the lower bound of the values to be fetched and will determine the starting point where `'reverse'` is not `true`. Only records where the key is greater than (or equal to) this option will be included in the range. When `'reverse'` is 'true` the order will be reversed, but the records returned will be the same.

* `'lt'` (less than), `'lte'` (less than or equal) define the higher bound of the range to be fetched and will determine the starting poitn where `'reverse'` is *not* `true`. Only key / value pairs where the key is less than (or equal to) this option will be included in the range. When `'reverse'` is `true` the order will be reversed, but the records returned will be the same.

* `'start', 'end'` legacy ranges - instead use `'gte', 'lte'`

* `'reverse'` *(boolean, default: `false`)*: a boolean, set to true if you want the stream to go in reverse order. Beware that due to the way LevelDB works, a reverse seek will be slower than a forward seek.

* `'keys'` *(boolean, default: `true`)*: whether the callback to the `next()` method should receive a non-null `key`. There is a small efficiency gain if you ultimately don't care what the keys are as they don't need to be converted and copied into JavaScript.

* `'values'` *(boolean, default: `true`)*: whether the callback to the `next()` method should receive a non-null `value`. There is a small efficiency gain if you ultimately don't care what the values are as they don't need to be converted and copied into JavaScript.

* `'limit'` *(number, default: `-1`)*: limit the number of results collected by this iterator. This number represents a *maximum* number of results and may not be reached if you get to the end of the store or your `'end'` value first. A value of `-1` means there is no limit.

* `'fillCache'` *(boolean, default: `false`)*: wheather LevelDB's LRU-cache should be filled with data read.

* `'keyAsBuffer'` *(boolean, default: `true`)*: Used to determine whether to return the `key` of each entry as a `String` or a Node.js `Buffer` object. Note that converting from a `Buffer` to a `String` incurs a cost so if you need a `String` (and the `value` can legitimately become a UFT8 string) then you should fetch it as one.

* `'valueAsBuffer'` *(boolean, default: `true`)*: Used to determine whether to return the `value` of each entry as a `String` or a Node.js `Buffer` object.


When all three simultaneously, their priority is as follows:

* start < lte < lt
* end < gte < gt

--------------------------------------------------------
<a name="iterator_next"></a>
### iterator#next(callback)
<code>next()</code> is an instance method on an existing iterator object, used to increment the underlying LevelDB iterator and return the entry at that location.

the `callback` function will be called with no arguments in any of the following situations:

* the iterator comes to the end of the store
* the `end` key has been reached; or
* the `limit` has been reached

Otherwise, the `callback` function will be called with the following 3 arguments:

* `error` - any error that occurs while incrementing the iterator.
* `key` - either a `String` or a Node.js `Buffer` object depending on the `keyAsBuffer` argument when the `iterator()` was called.
* `value` - either a `String` or a Node.js `Buffer` object depending on the `valueAsBuffer` argument when the `iterator()` was called.


--------------------------------------------------------
<a name="iterator_end"></a>
### iterator#end(callback)
<code>end()</code> is an instance method on an existing iterator object. The underlying LevelDB iterator will be deleted and the `callback` function will be called with no arguments if the operation is successful or with a single `error` argument if the operation failed for any reason.


--------------------------------------------------------
<a name="LevelDB_destroy"></a>
### LevelDB.destroy(location, callback)
<code>destroy()</code> is used to completely remove an existing LevelDB database directory. You can use this function in place of a full directory *rm* if you want to be sure to only remove LevelDB-related files. If the directory only contains LevelDB files, the directory itself will be removed as well. If there are additional, non-LevelDB files in the directory, those files, and the directory, will be left alone.

The callback will be called when the destroy operation is complete, with a possible `error` argument.

<a name="LevelDB_repair"></a>
### LevelDB.repair(location, callback)
<code>repair()</code> can be used to attempt a restoration of a damaged LevelDB store. From the LevelDB documentation:

> If a DB cannot be opened, you may attempt to call this method to resurrect as much of the contents of the database as possible. Some data may be lost, so be careful when calling this function on a database that contains important information.

You will find information on the *repair* operation in the *LOG* file inside the store directory. 

A `repair()` can also be used to perform a compaction of the LevelDB log into table files.

The callback will be called when the repair operation is complete, with a possible `error` argument.


<a name="safety"></a>
Safety
------

### Database state

Currently LevelDB does not track the state of the underlying LevelDB instance. This means that calling `open()` on an already open database may result in an error. Likewise, calling any other operation on a non-open database may result in an error.

LevelUP currently tracks and manages state and will prevent out-of-state operations from being send to LevelDB. If you use LevelDB directly then you must track and manage state for yourself.

<a name="support"></a>
Getting support
---------------

There are multiple ways you can find help in using LevelDB in Node.js:

 * **IRC:** you'll find an active group of LevelUP users in the **##leveldb** channel on Freenode, including most of the contributors to this project.
 * **Mailing list:** there is an active [Node.js LevelDB](https://groups.google.com/forum/#!forum/node-levelup) Google Group.
 * **GitHub:** you're welcome to open an issue here on this GitHub repository if you have a question.

<a name="contributing"></a>
Contributing
------------

LevelDB is an **OPEN Open Source Project**. This means that:

> Individuals making significant and valuable contributions are given commit-access to the project to contribute as they see fit. This project is more like an open wiki than a standard guarded open source project.

See the [CONTRIBUTING.md](https://github.com/rvagg/node-LevelDB/blob/master/CONTRIBUTING.md) file for more details.

### Contributors

LevelDB is only possible due to the excellent work of the following contributors:

<table><tbody>
<tr><th align="left">Riceball LEE</th><td><a href="https://github.com/snowyu">GitHub/snowyu</a></td><td></td></tr>
<tr><th align="left">Rod Vagg</th><td><a href="https://github.com/rvagg">GitHub/rvagg</a></td><td><a href="http://twitter.com/rvagg">Twitter/@rvagg</a></td></tr>
<tr><th align="left">John Chesley</th><td><a href="https://github.com/chesles/">GitHub/chesles</a></td><td><a href="http://twitter.com/chesles">Twitter/@chesles</a></td></tr>
<tr><th align="left">Jake Verbaten</th><td><a href="https://github.com/raynos">GitHub/raynos</a></td><td><a href="http://twitter.com/raynos2">Twitter/@raynos2</a></td></tr>
<tr><th align="left">Dominic Tarr</th><td><a href="https://github.com/dominictarr">GitHub/dominictarr</a></td><td><a href="http://twitter.com/dominictarr">Twitter/@dominictarr</a></td></tr>
<tr><th align="left">Max Ogden</th><td><a href="https://github.com/maxogden">GitHub/maxogden</a></td><td><a href="http://twitter.com/maxogden">Twitter/@maxogden</a></td></tr>
<tr><th align="left">Lars-Magnus Skog</th><td><a href="https://github.com/ralphtheninja">GitHub/ralphtheninja</a></td><td><a href="http://twitter.com/ralphtheninja">Twitter/@ralphtheninja</a></td></tr>
<tr><th align="left">David Björklund</th><td><a href="https://github.com/kesla">GitHub/kesla</a></td><td><a href="http://twitter.com/david_bjorklund">Twitter/@david_bjorklund</a></td></tr>
<tr><th align="left">Julian Gruber</th><td><a href="https://github.com/juliangruber">GitHub/juliangruber</a></td><td><a href="http://twitter.com/juliangruber">Twitter/@juliangruber</a></td></tr>
<tr><th align="left">Paolo Fragomeni</th><td><a href="https://github.com/hij1nx">GitHub/hij1nx</a></td><td><a href="http://twitter.com/hij1nx">Twitter/@hij1nx</a></td></tr>
<tr><th align="left">Anton Whalley</th><td><a href="https://github.com/No9">GitHub/No9</a></td><td><a href="https://twitter.com/antonwhalley">Twitter/@antonwhalley</a></td></tr>
<tr><th align="left">Matteo Collina</th><td><a href="https://github.com/mcollina">GitHub/mcollina</a></td><td><a href="https://twitter.com/matteocollina">Twitter/@matteocollina</a></td></tr>
<tr><th align="left">Pedro Teixeira</th><td><a href="https://github.com/pgte">GitHub/pgte</a></td><td><a href="https://twitter.com/pgte">Twitter/@pgte</a></td></tr>
<tr><th align="left">James Halliday</th><td><a href="https://github.com/substack">GitHub/substack</a></td><td><a href="https://twitter.com/substack">Twitter/@substack</a></td></tr>
</tbody></table>

### Windows

A large portion of the Windows support comes from code by [Krzysztof Kowalczyk](http://blog.kowalczyk.info/) [@kjk](https://twitter.com/kjk), see his Windows LevelDB port [here](http://code.google.com/r/kkowalczyk-leveldb/). If you're using LevelUP on Windows, you should give him your thanks!


<a name="license"></a>
License &amp; copyright
-------------------

Copyright (c) 2012-2014 LevelDB contributors (listed above).

LevelDB is licensed under the MIT license. All rights not explicitly granted in the MIT license are reserved. See the included LICENSE.md file for more details.

*LevelDB builds on the excellent work of the LevelDB and Snappy teams from Google and additional contributors. LevelDB and Snappy are both issued under the [New BSD Licence](http://opensource.org/licenses/BSD-3-Clause).*
