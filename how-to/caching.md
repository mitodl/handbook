---
parent: How To
---
# Caching

### When to consider caching

- Frequently requested data
- Expensive data computations
- Upstream resource contention (e.g. high server load)
- Slow client-facing responses
- Assets (e.g. css, js, images)

### How to cache

Caching isn't a one-size-fits-all problem. Consideration needs to be made for the nature of the data being cached to determine the best way in which to cache. Some questions to ask about the nature of the data:

- What is the lifecycle of this data?
  - How often does it change?
  - Do we know when it changes?
    - Can we automate cache purge/eviction?
    - Will need to force a cache purge/eviction?
- How tolerant is the application or user experience to stale data?
  - What problems could caching this data cause (race conditions, decisions made on stale data)?
  - Can it be cached indefinitely?
  - Can it be cached for a shorter interval (hours, minutes, seconds)?
  - Can it be cached for the duration of the request?

### Types of caching

These types could in theory be layered, but such an approach should be taken with caution as it becomes more difficult to reason about a system with multiple caching layers, particularly when we don't control one or more of those layers, such as a CDN.

#### Application level caching

Application level caching should be considered for data that is resource intensive to compute or is requested very frequently. It also typically requires that this data isn't user-dependent. It can be done at any level within the application (view, database query, etc). These values would typically be cached in the configured django cache (typically Redis for us). Because we control the cache storage and retrieval, we can invalidate it ourselves as well.

Data can be cached for hours, minutes, seconds, or even just for the duration of the request. Which is best is highly dependent on the nature of the data and testing and experimentation with values is always a good idea.

Some examples:

- A list of courses available
- Pricing information
-

**Use application level caching for data that is mutable over time and is expensive to compute.**

#### Request level caching

Request level caching instructs the browser and CDNs on how to cache content at the request level, this typically involves a number of HTTP request and response headers working in concert.

- For all content:
  - `ETag`/`Last-Modified` headers on responses and handle `If-None-Match`/`If-Modified-Since` on incoming requests
- Content that never changes
  - _e.g._ css, js, image assets served with a hashed filename
  - Requires a unique url for this content
  - Use `Cache-Control` with a long `max-age` - CDNs and browsers will cache for a long time
- Content that changes, but not that often:
  - _e.g._ a marketing page
  - Use `Cache-Control: no-cache` (the response will still be cached, but the downstream CDN will check the upstream server first before serving the cached version)
- Content that should be cached by the browser, but not the CDN:
  - _e.g._ a profile page
  - Use `Cache-Control: private`
  - Very likely a short-lived `max-age` (i.e. 30s)

### Cache Purging

Caches may occasionally need purging, it's important to know ahead of time how this is done.

- CDNs often have an option to purge the entire cache, this is typically something our DevOps team can perform
- Application level caching can be purged by evicting individual values
  - **NOTE:** currently both celery and the django cache use the same redis instance, so it's not possible to purge everything at the cache level


**Use request level caching for content that never or infrequently changes. Support the appropriate headers for the latter.**

### Specifics

#### Wagtail

We use Wagtail in many of our products, some best practices on caching performantly:

- Using the dynamic image serve view ([Example in xPro](https://docs.wagtail.io/en/stable/advanced_topics/images/image_serve_view.html))
- Adding a header to cache the dynamically-served images in a user's browser ([Example in xPro](https://github.com/mitodl/mitxpro/blob/abef74ad530770cc8772620dedae20714fc1a2e5/mitxpro/urls.py#L95-L105))
- Caching image renditions in Redis ([Example in xPro](https://docs.wagtail.io/en/v2.9.3/advanced_topics/performance.html#caching-image-renditions) - requires Wagtail 2.9+)
- Using a custom templatetag which adds a version to each rendered image, allowing us to cache at the CDN level ([Example in xPro](https://github.com/mitodl/mitxpro/blob/e8e5002662f794c3b17a63063af0d1aad293bff3/cms/templatetags/image_version_url.py#L12))


### Resources

- [Mozilla HTTP Caching Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)
- [Caching Best Practices](https://jakearchibald.com/2016/caching-best-practices/)
