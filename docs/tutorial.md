# Table of Contents

- [Table of Contents](#table-of-contents)
  - [CREATE WEB1](#create-web1)
    - [Upstream](#upstream)
    - [Route](#route)
    - [Test Call](#test-call)
  - [CREATE WEB2](#create-web2)
    - [Upstream](#upstream-1)
    - [Route](#route-1)
    - [Test Call](#test-call-1)
  - [CREATE LOAD BALANCER](#create-load-balancer)
    - [Upstream](#upstream-2)
    - [Route](#route-2)
    - [Test Call](#test-call-2)
  - [CREATE RATE LIMIT (On Web2)](#create-rate-limit-on-web2)
    - [Route](#route-3)
    - [Test Call](#test-call-3)
  - [CREATE API KEY AUTHEN (On Web1)](#create-api-key-authen-on-web1)
    - [Add API Key via Consumers](#add-api-key-via-consumers)
    - [Route](#route-4)
    - [Test Call](#test-call-4)
  - [CREATE JWT AUTHEN (On Web1)](#create-jwt-authen-on-web1)
    - [Add API Key via Consumers](#add-api-key-via-consumers-1)
    - [Route](#route-5)
    - [Test Call](#test-call-5)
  - [CREATE gRPC](#create-grpc)
    - [Run gRPC Server](#run-grpc-server)
    - [Upstream](#upstream-3)
    - [Route](#route-6)
    - [Test Call (directory /grpc_server_example)](#test-call-directory-grpc_server_example)
  - [CREATE PROXY CACHE](#create-proxy-cache)
    - [Setting APISIX at apisix_conf/config.yaml](#setting-apisix-at-apisix_confconfigyaml)
    - [Upstream \&\& Route](#upstream--route)
  - [Test Normal Called](#test-normal-called)
  - [Test Cache Called](#test-cache-called)
  - [CREATE GO PLUGINS](#create-go-plugins)
    - [Upstream \&\& Route](#upstream--route-1)
    - [Test Call](#test-call-6)
  - [LIST ROUTES](#list-routes)
    - [List all routes in APISIX](#list-all-routes-in-apisix)
  - [DOCKER COMPOSE](#docker-compose)
    - [Docker Compose Down](#docker-compose-down)
    - [Docker Compose Up](#docker-compose-up)
    - [Reload Nginx config](#reload-nginx-config)
  - [NOTE](#note)

## CREATE WEB1

### Upstream

```sh
curl "http://127.0.0.1:9180/apisix/admin/upstreams/web1" \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
  "type": "roundrobin",
  "nodes": {
    "web1:80": 1
  }
}'
```

### Route

- for anypath will exactly go to the web1/

```sh
curl "http://127.0.0.1:9180/apisix/admin/routes/web1" \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
  "methods": ["GET"],
  "uri": "/web1/*",
  "upstream_id": "web1"
}'
```

- for rewrite exact path (/web1) and use foo for web1/uri

```sh
curl "http://127.0.0.1:9180/apisix/admin/routes/web1" \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '{
  "methods": ["GET"],
  "uri": "/web1",
  "upstream_id": "web1",
  "plugins": {
    "proxy-rewrite": {
      "uri": "/foo",
      "host":"web1",
      "headers":{
            "Content-Type":"application/json"
         }
    }
  }
}'
```

- for rewrite dynamic path and use trimmed path for web1/rewritten-uri

```sh
curl "http://127.0.0.1:9180/apisix/admin/routes/web1" \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '{
  "methods": ["GET"],
  "uri": "/web1/*",
  "upstream_id": "web1",
  "plugins": {
    "proxy-rewrite": {
      "regex_uri": ["^/web1/(.*)", "/$1"],
      "headers":{
            "Content-Type":"application/json"
         }
    }
  }
}'
```

### Test Call

```sh
curl -i -X GET "http://127.0.0.1:80/web1/"
```

## CREATE WEB2

### Upstream

```sh
curl "http://127.0.0.1:9180/apisix/admin/upstreams/web2" \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
  "type": "roundrobin",
  "nodes": {
    "web2:80": 1
  }
}'
```

### Route

```sh
curl "http://127.0.0.1:9180/apisix/admin/routes/web2" \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
  "methods": ["GET"],
  "uri": "/web2/*",
  "upstream_id": "web2"
}'
```

### Test Call

```sh
curl -i -X GET "http://127.0.0.1:80/web2/"
```

## CREATE LOAD BALANCER

### Upstream

```sh
curl http://127.0.0.1:9180/apisix/admin/upstreams/web_backend \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
  "type": "roundrobin",
  "nodes": {
    "web1:80": 1,
    "web2:80": 3
  }
}'
```

### Route

```sh
curl http://127.0.0.1:9180/apisix/admin/routes/app_route \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
  "methods": ["GET"],
  "uri": "/app/*",
  "upstream_id": "web_backend"
}'
```

### Test Call

```sh
curl -i -X GET "http://127.0.0.1:80/app/"
```

## CREATE RATE LIMIT (On Web2)

### Route

```sh
curl http://127.0.0.1:9180/apisix/admin/routes/limit-count \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
  "methods": ["GET"],
  "uri": "/limit-web2/*",
  "plugins": {
      "limit-count": {
          "count": 2,
          "time_window": 60,
          "rejected_code": 503,
          "key_type": "var",
          "key": "remote_addr"
      }
    },
  "upstream_id": "web2"
}'
```

### Test Call

```sh
curl -i -X GET "http://127.0.0.1:80/limit-web2/"
```

## CREATE API KEY AUTHEN (On Web1)

### Add API Key via Consumers

```sh
curl http://127.0.0.1:9180/apisix/admin/consumers \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '{
  "username": "auth_one",
  "plugins": {
    "key-auth": {
      "key": "my-secret-key"
    }
  }
}'
```

### Route

```sh
curl "http://127.0.0.1:9180/apisix/admin/routes/auth-web1" \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '{
  "methods": ["GET"],
  "uri": "/auth/web1/*",
  "upstream_id": "web1",
  "plugins": {
    "proxy-rewrite": {
      "regex_uri": ["^/auth/web1/(.*)", "/$1"],
      "headers": {
        "Content-Type": "application/json"
      }
    },
    "key-auth": {}
  }
}'
```

### Test Call

```sh
curl -i -X GET "http://127.0.0.1:80/auth/web1/foo" -H "apikey: my-secret-key"
```

## CREATE JWT AUTHEN (On Web1)

### Add API Key via Consumers

```sh
curl http://127.0.0.1:9180/apisix/admin/consumers \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '{
  "username": "jwt_user",
  "plugins": {
    "jwt-auth": {
      "key": "jwt_user_key",
      "secret": "my_super_secret",
      "algorithm": "HS256"
    }
  }
}'
```

### Route

```sh
curl "http://127.0.0.1:9180/apisix/admin/routes/auth-jwt-web1" \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '{
  "methods": ["GET"],
  "uri": "/auth-jwt/web1/*",
  "upstream_id": "web1",
  "plugins": {
    "proxy-rewrite": {
      "regex_uri": ["^/auth-jwt/web1/(.*)", "/$1"],
      "headers": {
        "Content-Type": "application/json"
      }
    },
    "jwt-auth": {}
  }
}'
```

### Test Call

```sh
curl -i -X GET "http://127.0.0.1:80/auth-jwt/web1/foo" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJrZXkiOiJqd3RfdXNlcl9rZXkifQ.qxQgEclS22YLzrlVmm1E8ByJBS_R0J9RB3zJgSM3JXk"
```

## CREATE gRPC

### Run gRPC Server

```sh
cd grpc_server_example && go run main.go
```

### Upstream

```sh
curl "http://127.0.0.1:9180/apisix/admin/upstreams/grpc1" \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
  "scheme": "grpc",
  "type": "roundrobin",
  "tls": false,
  "nodes": {
    "127.0.0.1:50051": 1
  }
}'
```

### Route

```sh
curl "http://127.0.0.1:9180/apisix/admin/routes/grpc1" \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
  "methods": ["POST", "GET"],
  "uri": "/helloworld.Greeter/SayHello",
  "upstream_id": "grpc1"
}'
```

### Test Call (directory /grpc_server_example)

```sh
grpcurl -plaintext -import-path ./proto -proto helloworld.proto \
    -d '{"name":"apisix"}' 127.0.0.1:50051 helloworld.Greeter.SayHello
```

## CREATE PROXY CACHE

### Setting APISIX at apisix_conf/config.yaml

```sh
apisix:
  proxy_cache:
    cache_ttl: 10s  # default cache TTL for caching on disk
    zones:
      - name: memory_cache
        memory_size: 50m
```

### Upstream && Route

```sh
curl http://127.0.0.1:9180/apisix/admin/routes/1 \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
    "uri": "/ip",
    "plugins": {
        "proxy-cache": {
            "cache_strategy": "memory",
            "cache_zone": "memory_cache",
            "cache_ttl": 10
        }
    },
    "upstream": {
        "nodes": {
            "httpbin.org": 1
        },
        "type": "roundrobin"
    }
}'
```

## Test Normal Called

Run the following command:

```sh
curl http://127.0.0.1:80/ip -i
```

**Expected Response:**

```
Apisix-Cache-Status: MISS | EXPIRED
```

---

## Test Cache Called

Run the same command again:

```sh
curl http://127.0.0.1:80/ip -i
```

**Expected Response:**

```
Apisix-Cache-Status: HIT
```

## CREATE GO PLUGINS

### Upstream && Route

```sh
curl http://127.0.0.1:9180/apisix/admin/routes/go-plug \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
    "uri": "/webplug",
    "plugins": {
        "ext-plugin-pre-req": {
            "conf": [
                {
                    "name": "say",
                    "value": "{\"body\": \"hello-from-golang\"}"
                }
            ]
        }
    },
    "upstream": {
        "nodes": {
            "httpbin.org": 1
        },
        "type": "roundrobin"
    }
}'

curl http://127.0.0.1:9180/apisix/admin/routes/go-plug \
-H "X-API-KEY: newsupersecurekey" -X PUT -d '
{
    "uri": "/webplug",
    "plugins": {
        "ext-plugin-pre-req": {
            "conf": [
                {
                    "name": "say",
                    "value": "{\"body\": \"hello-from-golang\"}"
                }
            ]
        }
    }
}'
```

### Test Call

```sh
curl -i -X GET "http://127.0.0.1:80/webplug"
```

## LIST ROUTES

### List all routes in APISIX

```sh
curl "http://127.0.0.1:9180/apisix/admin/routes" \
-H "X-API-KEY: newsupersecurekey" -X GET
```

## DOCKER COMPOSE

### Docker Compose Down

```sh
docker compose -f example/docker-compose.yml down
```

### Docker Compose Up

```sh
docker compose -f example/docker-compose.yml up -d
```

### Reload Nginx config

```sh
docker exec d7998f563077 nginx -s reload
```

## NOTE

- Host represents the intended destination hostname as specified by the client.
