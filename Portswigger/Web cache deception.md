## Web caches

A web cache is a system that sits between the origin server and the user. When a client requests a static resource, the request is first directed to the cache. If the cache doesn't contain a copy of the resource (known as a cache miss), the request is forwarded to the origin server, which processes and responds to the request. The response is then sent to the cache before being sent to the user. The cache uses a preconfigured set of rules to determine whether to store the response.

When a request for the same static resource is made in the future, the cache serves the stored copy of the response directly to the user (known as a cache hit).

![[Pasted image 20260319210053.png]]

Caching has become a common and crucial aspect of delivering web content, particularly with the widespread use of Content Delivery Networks (CDNs), which use caching to store copies of content on distributed servers all over the world. CDNs speed up delivery by serving content from the server closest to the user, reducing load times by minimizing the distance data travels.

---

## Cache keys

When the cache receives an HTTP request, it must decide whether there is a cached response that it can serve directly, or whether it has to forward the request to the origin server. The cache makes this decision by generating a 'cache key' from elements of the HTTP request. Typically, this includes the URL path and query parameters, but it can also include a variety of other elements like headers and content type.

If the incoming request's cache key matches that of a previous request, the cache considers them to be equivalent and serves a copy of the cached response.


---

## Cache rules

Cache rules determine what can be cached and for how long it should be cached it basically used on static content becuz dynamic ones may contain sensitive info and those are directly got from the server.
The web cache deception attack exploits how this cache rules are applied.

Types of rules :
- Static file extension rules - These rules match the file extension of the requested resource, for example `.css` for stylesheets or `.js` for JavaScript files.
- Static directory rules - These rules match all URL paths that start with a specific prefix. These are often used to target specific directories that contain only static resources, for example `/static` or `/assets`.
- File name rules - These rules match specific file names to target files that are universally required for web operations and change rarely, such as `robots.txt` and `favicon.ico`.
- Caches may also implement custom rules based on other criteria, such as URL parameters or dynamic analysis.

---

## Constructing a web cache deception attack

**Step 1 — Find a target endpoint**

You're looking for pages that:

- Return **your session's data** (profile, account info, tokens, etc.)
- Use `GET` — because POST/PUT requests modify state and caches don't touch those
- The sensitive stuff might not even be visible rendered in browser — check the **raw response in Burp** (could be hidden fields, tokens in JSON, etc.)



**Step 2 — Find the discrepancy**

This is the core research phase. You're looking for a mismatch between how the **cache** and **origin server** interpret the same URL. Three main types:

**URL-to-resource mapping** — origin ignores extra path segments:

```
/profile/legit → serves profile
/profile/legit/anything.css → STILL serves profile (origin doesn't care)
```

**Delimiter confusion** — characters like `;`, `?`, `#` mean different things to different systems:

```
/profile;fake.css  ← cache sees .css, origin sees /profile
```

**Path normalization** — encoded characters get decoded differently:

```
/profile%2Ffake.css ← cache and origin decode this differently
```



**Step 3 — Craft + deliver the malicious URL**

You build a URL that exploits the discrepancy, send it to the victim. When they load it:

1. Cache has no stored response → passes to origin
2. Origin serves their real dynamic data (ignores the junk suffix)
3. Cache sees the URL looks static → **stores it** ✅ (for you)

You then hit the same URL in **Burp Repeater** — look for `X-Cache: HIT` in the response headers. That confirms the cache served you the stored version = victim's data 💀

**Why Burp and not browser** — browser might get redirected if you're not logged in, or the app nukes local state, which masks the vuln entirely

---

