# nginx config

Demo nginx configuration with default load balancing (upstream round-robin)

## Nginx load balancing algorithms

Globally, the following load balancing mechanisms (or methods) are supported in nginx:

- round-robin — requests to the application servers are distributed in a round-robin fashion,
- least-connected — next request is assigned to the server with the least number of active connections,
- ip-hash — a hash-function is used to determine what server should be selected for the next request (based on the client’s IP address).

#### Default load balancing configuration (round-robin)

```nginx
http {
    upstream myapp1 {
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://myapp1;
        }
    }
}
```

Also we can add weight to round-robin strategy:

#### Weighted load balancing

With this configuration, every 5 new requests will be distributed across the application instances as the following: 3 requests will be directed to srv1, one request will go to srv2, and another one — to srv3.

It is similarly possible to use weights with the least-connected and ip-hash load balancing in the recent versions of nginx.

```nginx
upstream myapp1 {
    server srv1.example.com weight=3;
    server srv2.example.com;
    server srv3.example.com;
}
```

#### Least connected load balancing

```nginx
 upstream myapp1 {
        least_conn;
        server srv1.example.com;
        server srv2.example.com;
        server srv3.example.com;
 }
```

#### Session persistence (sticky-session style)

Please note that with round-robin or least-connected load balancing, each subsequent client’s request can be potentially distributed to a different server. There is no guarantee that the same client will be always directed to the same server.

If there is the need to tie a client to a particular application server — in other words, make the client’s session “sticky” or “persistent” in terms of always trying to select a particular server — the ip-hash load balancing mechanism can be used.

With ip-hash, the client’s IP address is used as a hashing key to determine what server in a server group should be selected for the client’s requests. This method ensures that the requests from the same client will always be directed to the same server except when this server is unavailable.

```nginx
upstream myapp1 {
    ip_hash;
    server srv1.example.com;
    server srv2.example.com;
    server srv3.example.com;
}
```