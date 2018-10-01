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

Browser will send request to validate is cached data can be use.
This can reduce bandwidth but still use a roundtrip to validate cache response.

```text
Cache-Control: no-cache
```

#### max-age

Set the maximum amount of time (in seconds) that resource will be cached.

Browser will use cached response without send request (from disk cache).

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

### Cache-Control Summary

#### Static Files with unique file name (ex. with hash in file name)

```text
Cache-Control: max-age=31536000
```

#### MPA (generate HTML)

```text
Cache-Control: no-store
```

#### Web Pages (static HTML)

```text
Cache-Control: no-cache
```

#### SPA index.html

```text
Cache-Control: no-cache
```

## Cache Revalidation

When `Cache-Control` set to `no-cache`,
at least one of these headers must be set to trigger cache revalidation.

### Last-Modified

Time that resource last modified.

Normally use for static file, file server.

```text
Last-Modified: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT
```

ex.

```text
Last-Modified: Tue, 18 Sep 2018 11:12:03 GMT
```

If browser has cached response, browser will send request to validate cache with

```text
If-Modified-Since: Tue, 18 Sep 2018 11:12:03 GMT
```

If server has newer file, then send new file and new `Last-Modified`

```text
HTTP/1.1 200 OK
Date: Tue, 18 Sep 2018 22:00:00 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 7
If-Modified-Since: Tue, 18 Sep 2018 20:00:00 GMT

Content
```

If file does not modify, then return 304 status code without body.

```text
HTTP/1.1 304 Not Modified
Date: Tue, 18 Sep 2018 22:00:00 GMT

```

### ETag

> ETag can be use for conditional requests, but not include in this article.

Like `Last-Modified` but ETag have more accurate.

Use when server do not know resource last modified or resource change many time more than one in a second.

ETag is like fingerprint of resource, and must generate new value when resource value changed.

ETag can generate using hash of response body or uuid or anything that unique when resource changed.

Browser will send `If-None-Match` to server to validate cache.

ETag has `Weak` and `Strong` validator.

#### Weak

Weak ETag use to validate only **body** of response, when response body changed, ETag value will change.

```text
ETag: W/"value"
```

Example

```text
ETag: W/"abcde"
```

#### Strong

Strong ETag validate **both** response **body** and **headers**.

Even if body does not change but header change, Strong ETag must change to new value.

Use-cases: To cache byte-range response.

```text
ETag: "value"
```

Example

```text
ETag: "abc1" # for gzip
ETag: "abc2" # for br
```

#### Comparison

| ETag 1 | ETag 2 | Strong Comparison | Weak Comparison |
|--------|--------|-------------------|-----------------|
| W/"1"  | W/"1"  | no match          | match           |
| W/"1"  | W/"2"  | no match          | no match        |
| W/"1"  | "1"    | no match          | match           |
| "1"    | "1"    | match             | match           |

(from [RFC-7232](https://tools.ietf.org/html/rfc7232#section-2.3.2))

#### ETag Summary

Use **Strong** ETag unless you know what you're doing 😁
