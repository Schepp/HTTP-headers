<!-- .slide: data-background="images/backgrounds/performance.jpg" data-state="inverted faded" -->

# PERFORMANCE

## Data Saving Modes & Transform Proxies
---

A `Save-Data: on` request header signals that the user wants less data to be consumed

 - *HTTP-served* resources then get transformed by data saving proxies with certain browsers: HTML, CSS and JS get minified, media gets heavily compressed.


Data Saving Mode can also be detected via JavaScript:

```js
if (
  navigator.connection && 
  navigator.connection.saveData === true
) {
  // Implement data saving operations here.
}
```

- When running through a transform proxy, the original IP address of the client will appear in the `X-Forwarded-For` request header.  

- Should you react to the header, don't forget to set a `Vary: Save-Data` header in your response! <span class="fragment">(more on that later)</span>

- Set a `Cache-Control: no-transform` response header to protect certain assets from being transformed / optimized on the fly

 - Protects your HTML and JS from code injections, e.g. by mobile carriers</p>

<br />
<!-- .slide: data-background="" data-state="inverted" -->

## Client Side Caching
---

Don't use the `Expires` response header. It's old and weird.

It needs the expiration day's weekday:<br>`Expires: Sun, 03 Feb 2019 16:25:41 GMT`

- Use `Cache-Control` instead.

```
`Cache-Control: max-age=31536000, public, immutable`
```

- `max-age=31536000`: Resource expires one year from now.
- `public`: Proxies are allowed to cache the resource.
- `immutable`: Resource shall never be checked again.


```
`Cache-Control: stale-while-revalidate`

or

`Cache-Control: stale-while-revalidate=1200`
```

## The client takes a cached version, followed by fetching an update.
---

```
`Cache-Control: stale-if-error`

or 

`Cache-Control: max-age=600, stale-if-error=1200`
```
The client uses a cached version if the server throws an error.

## Applies to 500, 502, 503, or 504 HTTP response status codes.

---

What if you messed up client side caching and need a resource uncached?

```
`Clear-Site-Data: "cache", "cookies", "storage", "executionContexts"`

or 

`Clear-Site-Data: "*"`

or 

`Clear-Site-Data: "cache"`
```

All of the above clear a client's (cached) data.

<br />
<!-- .slide: data-background="" data-state="inverted" -->

## Client Hints

---

Client Hints allow for proactive content negotiation. They can tell the server about a device's...

 - `DPR: 1.5`: Device pixel ratio (Screen pixels per CSS pixel).
  - `Viewport-Width: 1024`: Width of the viewport in pixels.
  - `Width: 400`: Actual rendered width of an image in CSS pixels.
  - `Device-Memory: 0.5`: The devices's main memory in GiB.

_
---

The following additional Client Hints are currently being specified...

  - `ECT: 4g`: Current overall effective network type.
  - `RTT: 300`: Current effective round-trip time in milliseconds.
  - `Downlink: 1`: Current effective bandwidth in Mb/s.

_
---

Client Hints use must be activated/opted in by the server:

```
Accept-CH: DPR, Width, Viewport-Width
```

## Which means that the first call to a server will always be w/o Client Hints.

---

Servers may also tell the client for how long it wants hints to be sent over (as a time delta from now):

```
Accept-CH: DPR, Width, Viewport-Width
Accept-CH-Lifetime: 86400
```

## In this case the server wants to receive Client Hints for the next 24 hours.

---

After activating Client Hints, this is what the browser will start sending:

```
GET image.jpg
Accept: image/webp,image/*,*/*;q=0.8
DPR: 2
Viewport-Width: 1024
Width: 508
```


When replying with the requested image, the server adds the DPR of the resource so that the result is not interfering with the browser's layout mechanics:

```
Content-DPR: 2
```


<!-- .slide: data-background="" data-state="inverted" -->

## Controlling Cache Proxies
---

The `Vary` response header is solely meant for proxies (and CDNs) and tells them how to act on caching.

Sadly it's easy to get it wrong and any mistake hard to spot!
---

The `Vary` response header...

- ...needs to be sent by your server.
- ...always refers to headers sent by the clients.
- ...works like a unique key in a MySQL database.

<span class="fragment">_</span>
---

A few examples:

  - `Vary: Save-Data`: stripped down vs. full content</li>
  - `Vary: Accept`: file format variation, e.g. JPEG vs. WebP</li>
  - `Vary: Accept-Encoding`: compression type: gzip vs. brotli</li>
  - `Vary: DPR`: client hint for display density</li>
  - `Vary: Save-Data, Accept, Accept-Encoding, DPR`: look at all four 

_


 - Set `Cache-Control: private` for authenticated users, so that their responses don't get cached by proxies.

- Then only the client is allowed to cache (`private` as opposed to `public`).

---

## Resource Hints & Prefetch

---

Resource Hints help the browser with discovering render critical resources, like CSS, fonts and maybe also JavaScript.

<p class="fragment">
Most of the time you see them in the markup, like so:
<br>
```
<link href="/styles.css" rel="preload" as="style">
```
</p>

<p class="fragment">
But you can also send that info via HTTP header:
<br>
```
Link: <https://domain.com/styles.css>; rel=preload; as=style
```
</p>
---

If the server is HTTP/2 Push enabled it will force-push any (local) resource listed with a Link header. 


If you don't want that to happen, add a `nopush` attribute to it:

```
Link: <https://domain.com/styles.css>; rel=preload; as=style; nopush
```

```
<link href="/potential-next-page.html" rel="prefetch">
```

when Chrome prefetches such a resource, it'll add the following request header:
 
```
Purpose: prefetch
```


## Feature Policy

---

Feature Policy is like CSP (Content Security Policy), but for browser features. It allows web developers to selectively enable, disable, and modify the behavior of certain web features.

It can serve security as well as web performance.

---

This is how you disallow usage of the Geolocation API:

```
Feature-Policy: geolocation 'none'
```

This is how you allow its usage just for pages and iframes served from your host:

```
Feature-Policy: geolocation 'self'
```
---

Real world scenarios where Feature Policies help enforce performance best practices:

  - Marketing department uses Google Tag Manager and you don't want bad programming practices to be introduced onto your site.
  - People with some sort of access to the HTML output add render blocking A/B testing.
  - You have 3rd Party on your site but you want to limit its negative impact.
  - You have advertisement using video on your site and you don't want it to autoplay.

_
---

Out of the many policies existing the following ones are the most interesting ones for web performance:

 - `autoplay`: Allow or disallow autoplaying video.
 - `sync-script`: Allow or disallow synchronous script tags.
 - `document-write`: Allow or disallow `document.write`.
 - `lazyload`: Force all images and iframe to load lazily (as if all    had `lazyload="on"` attribute set).


---

(continued)

  - `image-compression`: restrict images to have a byte size no more than 10x bigger than their pixel count.
  - `maximum-downscaling-image`: restrict images to be downscaled by not more than 2x.
  - `unsized-media`: requires images to have a width & height specified,  otherwise defaults to 300 x 150.
  - `layout-animations`: turns off CSS animation for any property that triggers a re-layout (e.g. `top`, `width`, `max-height`).


_
---

## Server Timing Headers

---

> The Server-Timing header communicates one or more metrics and descriptions for a given request-response cycle. It is used to surface any backend server timing metrics (e.g. database read/write, CPU time, file system access, etc.) in the developer tools in the user's browser or in the PerformanceServerTiming interface.

---

Examples:

```
// Single metric with value
Server-Timing: cpu;dur=2.4

// Single metric with description and value
Server-Timing: cache;desc="Cache Read";dur=23.2

// Two metrics with value
Server-Timing: db;dur=53, app;dur=47.2
```

---
### Server Timings in the Devtools

![Server Timings](/images/server-timings.jpg)

---
Server Timings accessible for JavaScript

```
Server-Timing: cache;desc="Cache Read";dur=23.2
```

```js
const pageEntry = performance.getEntriesByType('navigation')[0]

console.log(pageEntry.serverTiming);
// {name: "cache", duration: 23.2, description: "Cache Read"}
```
---
Messing with the Server-Timing header :)

```
Server-Timing: data;desc="{\"ab-testgroup\": \"b\"}";dur=0
```

```js
const pageEntry = performance.getEntriesByType('navigation')[0];

console.log(pageEntry.serverTiming);
// {
// name: "data", 
// duration: 0, 
// description: "{\"abTestgroup\": \"b\"}"
// }

const data = JSON.parse(pageEntry.serverTiming.description);
// {abTestgroup: "b"}
```
---
<!-- .slide: data-background="images/backgrounds/performance.jpg" data-state="inverted faded" -->

&lt;/performance&gt;