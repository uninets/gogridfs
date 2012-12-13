gogridfs
===========

A simple HTTP file server frontend to mongoDB GridFS.

It can do nothing but serve files.

Install
-----------

```bash
go get github.com/mugenken/gogridfs
```

Usage
-----------

```
gogridfs -config /path/to/config.json
```

The module is configured with a JSON file. An example may look like this:

```javascript
{
    "servers": [                 // should contain at least one server of your mongoDB cluster
        "localhost:27012",
        "localhost:37012",
        "localhost:47012"
    ],
    "listen": "localhost:4242",  // the host and port to listen on
    "logfile": "gogridfs.log",   // the logfile
    "database": "gofiles",       // the database that contains the GridFS
    "gridfscollection": "fs",    // the GridFS root
    "handlepath": "/gofiles/",   // the path the handler will connect to
                                 // it will be cut from the file name so requests to
                                 // /gofiles/some/path/file.png will serve
                                 // some/path/file.png from GridFS
    "mode": "strong"             // mgo mode of querying
                                 // strong    => safe, reads and writes on master only
                                 // monotonic => faster, distribution of queries across nodes
                                 //              no guaranteed consistency between queries
                                 // eventual  => fastest, no guaranteed consistency at all
                                 // default node is striong
    "debug": true                // log requested file paths
}
```

To proxy with nginx add something like this to your server directive:

```nginx
upstream gogridfs {
        server 127.0.0.1:4242;
}

server {
    # more config

    location /gofiles/ {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        if (!-f $request_filename) {
            proxy_pass http://gogridfs;
            break;
        }
    }

    # more config
}
```

