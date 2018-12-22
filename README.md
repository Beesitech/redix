Redix
=======
> a fast persistent real-time key-value store, that uses the same [RESP](https://redis.io/topics/protocol) protocol and capable to store terabytes of data, also it integrates with your mobile/web apps to add real-time features, soon you can use it as a document store cause it should become a multi-model db. `Redix` is used in production, you can use it in your apps with no worries.


Features
=========
- Fast on-disk store
- Pluggable Storage Engine (`badger`, `bolt`)
- Multi Core via `-workers=<num workers here>`
- Very easy and simple
- Very compatible with any `redis-client`
- Not only basic redis channels subscriptions, but also there is `webhook` and `websocket`, so you can easily use integrate it directly to any web/mobile app.

Why
===
> I started this software to learn more about data modeling, data structures and how to map any data to pure key value, I don't need to build a redis clone, but I need to build something with my own concepts in my own style. I decided to use RESP (redis protocol) so you can use `Redix` with any redis client out there.

Production
==========
> Yep, This software is ready for use in production, and I'm already using it in production as a drop-in-replacement for redis in some softwares in [uFlare](https://uflare.io) for our solutions.

Install
=======
- from source: `go get github.com/alash3al/redix`.
- from binaries: go [there](https://github.com/alash3al/redix/releases) and choose your platform based binary, then download and execute from the command line with `-h` flag to see the help text.

Configurations
============
> It is so easy to configure `Redix`, there is no configuration files, it is all about running `./redix` after you download it from the [releases](https://github.com/alash3al/redix/releases), if you downloaded i.e 'redix_linux_amd64' and unziped it.

```bash
$ ./redix_linux_amd64 -h

  -engine string
        the storage engine to be used, available (default "badger")
  -http-addr string
        the address of the http server (default "localhost:7090")
  -resp-addr string
        the address of resp server (default "localhost:6380")
  -storage string
        the storage directory (default "./redix-data")
  -verbose
        verbose or not
  -workers int
        the default workers number (default ...)
```

Examples
=========

```bash

# i.e: $mykey1 = "this is my value"
$ redis-cli -p 6380 set mykey1 "this is my value"

# i.e: $mykey1 = "this is my value" and expire it after 10 seconds
$ redis-cli -p 6380 set mykey1 "this is my value" 10000

# i.e: echo $mykey1
$ redis-cli -p 6380 get mykey1

# i.e: $mymap1[x] = y
$ redis-cli -p 6380 hset mymap1 x y

# i.e: $mymap1[x] = y and expires it after 10 seconds
$ redis-cli -p 6380 hset mymap1 x y 10000

# i.e: sha512 of "test"
$ redis-cli -p 6380 encode sha512 test

# you want to notify an endpoint i.e: "http://localhost:800/new-data" that there is new data available, in other words, you want to subscribe a webhook to channel updates.
$ redis-cli -p 6380 webhookset testchan http://localhost:800/new-data

# add data to a list
# i.e: [].push(....)
$ redis-cli -p 6380 lpush mylist1 "I'm Mohammed" "I like to Go using Go" "I love coding"

# search in the list
$ redis-cli -p 6380 lsrch mylist1 "mo(.*)"

```

DB Engines
===========
- `Redix` supports two engines called `badger` and `bolt`
- `badger` is the default, it is inspired by Facebook [`RocksDB`](https://rocksdb.org/), it meant to be fast on-disk engine, [read more](https://github.com/dgraph-io/badger)
- `bolt` is our alternate engine, it is inspired by [`LMDB`](http://symas.com/mdb/), [read more](https://github.com/etcd-io/bbolt)

Supported Commands
===================
> `Redix` doesn't implement all redis commands, but instead it supports the core concepts that will help you to build any type of data models on top of it, there are more commands and features in all next releases.

## # Basic
- `PING`
- `QUIT`
- `SELECT`

## # Strings
- `SET <key> <value> [<TTL "millisecond">]`
- `MSET <key1> <value1> [<key2> <value2> ...]`
- `GET <key> [<default value>]`
- `MGET <key1> [<key2> ...]`
- `DEL <key1> [<key2> ...]`
- `EXISTS <key>`
- `INCR <key> [<by>]`
- `TTL <key>` returns `-1` if key will never expire, `-2` if it doesn't exists (expired), other wise will returns the `seconds` remain before the key will expire.

## # HASHES
> I enhanced the HASH MAP implementation and added some features like TTL per nested key,
> also you can check whether the hash map itself exists or not using `HEXISTS <hashmapname>` or a nested key 
> exists using `HEXISTS <hashmapname> <keyname>`.  

- `HSET <HASHMAP> <KEY> <VALUE> [<TTL "millesecond">]`
- `HMSET <HASHMAP> <key1> <value1> [<key2> <value2> ...]`
- `HGET <HASHMAP> <KEY>`
- `HDEL <HASHMAP> [<key1> <key2> ...]` (deletes the map itself or keys in the map)
- `HGETALL <HASHMAP>`
- `HMSET <HASHMAP> <key1> <val1> [<key2> <val2> ...]`
- `HEXISTS <HASHMAP> [<key>]`.
- `HINCR <HASHMAP> <key> [<by>]`
- `HTTL <HASHMAP> <key>`, the same as `TTL` but for `HASHMAP`

## # LIST
> I applied a new concept, you can push or push-unique values into the list,
>  based on that I don't need to implement two different data structures, 
> as well as, you can quickly iterate over a list in a high performance way,
> every push will return the internal offset of the value, also, the iterator `lrange`
> will tell you the next offset you can start from.  

- `LPUSH <LIST> <val1> [<val2> ...]` (push the item into the list "it doesn't check for uniqueness, it will append anyway (duplicate)")
- `LPUSHU <LIST> <val1> [<val2> ...]` (push the item into the list only if it isn't exists)
- `LGETALL <LIST> [<offset> <size>]`
- `LREM <LIST> [<val1> <val2> <val3> ...]` (deletes the list itself or values in the list)
- `LCOUNT <LIST>` (get the list members count)
- `LSUM <LIST>` (sum the members of the list "in case they were numbers")
- `LAVG <LIST>` (get the avg of the members of the list "in case they were numbers")
- `LMIN <LIST>` (get the minimum of the members of the list "in case they were numbers")
- `LMAX <LIST>` (get the maximum of the members of the list "in case they were numbers")
- `LSRCH <LIST> <NEEDLE>` (text-search using (string search or regex) in the list)
- `LSRCHCOUNT <LIST> <NEEDLE>` (size of text-search result using (string search or regex) in the list)

## # Pub/Sub
> `Redix` has very simple pub/sub functionality, you can subscribe to internal logs on the `*` channel or any custom defined channel, and publish to any custom channel.

- `SUBSCRIBE [<channel1> <channel2>]`, if there is no channel specified, it will be set to `*`
- `PUBLISH <channel> <payload>`
- `WEBHOOKSET <channel> <httpurl>`, register a http endpoint so it can be notified through `JSON POST` request with the channel updates, this command will return a reference ID so you can manage it later.
- `WEBHOOKDEL <ID>`, stops listening on a channel using the above reference ID.
- `WEBSOCKETOPEN <channel>`, opens a websocket endpoint and returns its id, so you can receive updates through `ws://server.address:port/stream/ws/{generated_id_here}`
- `WEBSOCKETCLOSE <ID>`, closes the specified websocket endpoint using the above generated id. 

## # Utils
> a helpers commands

- `ENCODE <method> <payload>`, encode the specified `<payload>` using the specified `<method>` (`md5`, `sha1`, `sha256`, `sha512`, `hex`)
- `UUIDV4`, generates a uuid-v4 string, i.e `0b98aa17-eb06-42b8-b39f-fd7ba6aba7cd`.
- `UNIQID`, generates a unique string.
- `RANDSTR [<size>, default size is 10]`, generates a random string using the specified length. 
- `RANDINT <min> <max>`, generates a random string between the specified `<min>` and `<max>`.
- `TIME`, returns the current time in `utc`, `seconds` and `nanoseconds`
- `DBSIZE`, returns the database size in bytes.
- `GC`, runs the Garbage Collector.

TODO
=====
- [x] Basic Commands
- [x] Strings Commands
- [x] Hashmap Commands
- [x] List Commands
- [x] PubSub Commands
- [x] Utils Commands
- [ ] Document/JSON Commands
- [ ] GIS Commands
