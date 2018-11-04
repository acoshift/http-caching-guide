# http-caching-guide

Short Guide for Modern HTTP Caching

## Cache-Control

Controls how responses should be cached.

### Directives

#### no-store

Do NOT cache the response.

```text
Cache-Control: no-store
```

#### no-cache

Revalidate cache before use (see [Cache Revalidation](#cache-revalidation) for more info).

Browser will send a request to validate if the cached data can be used.
This can reduce bandwidth but still requires a roundtrip to validate the cached response.

```text
Cache-Control: no-cache
```

#### max-age

Set the maximum amount of time (in seconds) that the resource will be cached.

Browser will use cached response without sending any request (from disk cache).

```text
Cache-Control: max-age=3600
```

#### public

Response can be cached in shared cache (Reverse Proxy, CDN, etc.)

> Note: most of the time, you don’t need to use `public` as it can be inferred by other caching directives, such as `max-age`. But check with CDN document first.

```text
Cache-Control: max-age=3600
```

equals to

```text
Cache-Control: public, max-age=3600
```

#### private

Response is intended for one user CAN NOT be cached in shared cache.

```text
Cache-Control: private, max-age=3600
```

#### immutable

Prevents cache revalidation when clicking Refresh button (which normally causes revalidation on all resources).

```text
Cache-Control: max-age=31536000, immutable
```

### Cache-Control Summary

#### Static files with unique file name (ex. with hash in file name)

```text
Cache-Control: public, max-age=31536000, immutable
```

#### MPA (generate HTML)

```text
Cache-Control: no-store
```

Why not `no-cache` ?

Because MPA server usually send `Set-Cookie` to rolling cookie expiration time,
and `304 Not Modified` should not send other header than cache information.
[RFC 7372 - Section 4.1](https://tools.ietf.org/html/rfc7232#section-4.1)

#### Web Pages (static HTML)

```text
Cache-Control: no-cache
```

#### SPA index.html

```text
Cache-Control: no-cache
```

## Cache Revalidation

> Last-Modified and ETag can be used for conditional requests, but not included in this article.

When `no-cache` directive is used,
at least one of these headers must be set to trigger cache revalidation.

### Last-Modified

Time that the resource has been last modified.

Normally used for static files.

```text
Last-Modified: <day-name>, <day> <month> <year> <hour>:<minute>:<second> GMT
```

ex.

```text
Last-Modified: Tue, 18 Sep 2018 11:12:03 GMT
```

If browser has cached the response, it will send a request to validate cache with

```text
If-Modified-Since: Tue, 18 Sep 2018 11:12:03 GMT
```

If the server has a newer file, then it sends the new file contents and new `Last-Modified` header

```text
HTTP/1.1 200 OK
Date: Tue, 18 Sep 2018 22:00:00 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 7
Last-Modified: Tue, 18 Sep 2018 20:00:00 GMT

Content
```

If file has not been modified since, then it returns 304 status code without body.

```text
HTTP/1.1 304 Not Modified
Date: Tue, 18 Sep 2018 22:00:00 GMT

```

### ETag

Like `Last-Modified` but `ETag` has more accuracy.

Used when server does not know resource's last modified date or when a resource could change more than once in a second.

ETag is like a fingerprint of resource, and a server must generate a new ETag when the resource’s contents changed.

ETag can be generated using hash of response body, UUID, or anything that results in a unique string when resource contents changed.

Browser will send `If-None-Match` to server to validate cache.

ETag has `Weak` and `Strong` validator.

#### Weak

A weak ETag is used to validate only **body** of response, when response body changed, ETag value will change.

```text
ETag: W/"value"
```

Example

```text
ETag: W/"abcde"
```

#### Strong

A strong ETag is used to validate **both** response **body** and **headers**.

Even if body is not changed but header is changed, a strong ETag must be changed to a new value.

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

Ex. you don't care about

- `Content-Encoding`
- `Content-Language`
- `Range`
- `Set-Cookie`
- `Vary`
- etc.

then you can use **Weak** ETag
