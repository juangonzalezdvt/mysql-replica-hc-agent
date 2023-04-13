mysql-replica-hc-agent
=============================

Based on https://github.com/fujiwara/mysql-slave-healthcheck-agent

Description
------------

`mysql-replica-hc-agent` is a HTTP daemon to show an information of "SHOW REPLICA STATUS" in JSON format.

This is useful for HAProxy's health check access instead of `option mysql-check`.

```
# haproxy.cfg
listen mysql
       bind     127.0.0.1:3306
       mode     tcp
       balance  roundrobin
       option   httpchk
       server   slave1 192.168.1.11:3306 check port 5000
       server   slave2 192.168.1.12:3306 check port 5000
```

Usage
------

```
$ mysql-replica-hc-agent
2014/05/21 00:04:28 Listing port 5000
2014/05/21 00:04:28 dsn root:@tcp(127.0.0.1:3306)/?charset=utf8
```

```
$ curl localhost:5000 | jq .
{
  "Connect_Retry": 60,
  "Exec_Source_Log_Pos": 1048,
  "Last_Errno": 0,
  "Last_Error": "",
  "Source_Host": "db-master",
  "Source_Log_File": "mysql-bin.000006",
  "Source_Port": 3306,
  ...
  "Until_Log_File": "",
  "Until_Log_Pos": 0
}
```

* The query "SHOW REPLICA STATUS" was succeeded, return HTTP status 200 and JSON.
* If could not connect to the MySQL or the MySQL is not a replica, return HTTP status 500.
* If replica is not running, return HTTP status 500. When the option -fail-replica-not-running=false is specified, return 200.

Options
-------

* -port : http listen port number. default 5000.
* -dsn : Data Source Name for MySQL. default "root:@tcp(127.0.0.1:3306)/?charset=utf8"
  See also https://github.com/go-sql-driver/mysql#dsn-data-source-name
  mysql's user must have a privilege of "REPLICATION CLIENT".
* --fail-replica-not-running=true: returns 500 if the replica is not running

How to build
------------

    $ go get github.com/juangonzalezdvt/mysql-replica-hc-agent

How to build (cross compile)
------------

Install gox.

    $ go get github.com/mitchellh/gox
    $ gox -build-toolchain

And make.

    $ make

The binary files will be build in `pkg` directory.

```
pkg
├── darwin_amd64
│   └── mysql-replica-hc-agent
└── linux_amd64
    └── mysql-replica-hc-agent
```

License
-------
Apache License 2.0

Original Author
-------
fujiwara.shunichiro@gmail.com
