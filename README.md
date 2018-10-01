# http-caching-guide

Short Guide for Modern HTTP Caching

## Cache-Control

Controls how to cache response

### Directives

#### no-store

Do NOT cache the response.

```text
Cache-Control: no-store
```

#### no-cache

Revalidate cache before use (see [Cache Revalidation](#cache-revalidation) for more info).

Client must send request to validate is cached data can be use.
This can reduce bandwidth but still use a roundtrip to validate cache response.

```text
Cache-Control: no-cache
```

#### max-age

Set the maximum amount of time (in seconds) that resource will be cached.

Client will use cached response without send request.

> Note: max-age can explicit set public directive.

```text
Cache-Control: max-age=3600
```

equals to

```text
Cache-Control: public, max-age=3600
```

#### public

Response can be cached in shared cache (Reverse Proxy, CDN, etc.)

Normally just set max-age.

#### private

Response CAN NOT be cached in shared cache.

```text
Cache-Control: private, max-age=3600
```

## Cache Revalidation

> TODO

### Last-Modified

> TODO

### ETag

> TODO
