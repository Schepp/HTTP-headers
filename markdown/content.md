<!-- .slide: data-background="" data-state="inverted" -->

# HTTP Headers
---
<!-- .slide: data-background="" data-state="inverted" -->

## Data Saving Modes & Transform Proxies

---

A `Save-Data: on` request header Signals that the user wants less data to be consumed

<p class="fragment">*HTTP-served* resources then get transformed by data saving proxies with certain browsers: HTML, CSS and JS get minified, media gets heavily compressed</p>

---

Data Saving Mode can also be detected via JavaScript:

```
if (
  navigator.connection && 
  navigator.connection.saveData === true
) {
  // Implement data saving operations here.
}
```

---

When running through a transform proxy, the original IP address of the client will appear in the `X-Forwarded-For` request header.  

---

Should you react to the header, don't forget to set a `Vary: Save-Data` header in your response! <span class="fragment">(more on that later)</span>

---
Set a `Cache-Control: no-transform` response header to protect certain assets from being transformed / optimized on the fly

<p class="fragment">Protects your HTML and JS from code injections, e.g. by mobile carriers</p>

---
<!-- .slide: data-background="" data-state="inverted" -->

## Client Side Caching

---

Don't use the `Expires` response header. It's old and weird.

It needs the expiration day's weekday:<br>`Expires: Sun, 03 Feb 2019 16:25:41 GMT`

<p class="fragment">Use `Cache-Control` instead</p>

---

`Cache-Control: max-age=31536000, public, immutable`

<p class="fragment">`max-age=31536000`: Resource expires one year from now</p>
<p class="fragment">`public`: Proxies are allowed to cache the resource</p>
<p class="fragment">`immutable`: Resource shall never be checked again</p>

---

What if you messed up client side caching and need a resource uncached?

---

`Clear-Site-Data: "cache", "cookies", "storage", "executionContexts"`

or 

`Clear-Site-Data: "*"`

or 

`Clear-Site-Data: "cache"`

all clear a client's (cached) data
---
<!-- .slide: data-background="" data-state="inverted" -->

## Client Hints

---

Client Hints allow for proactive content negotiation. They can tell the server about a device's...

<ul>
  <li class="fragment">`DPR: 1.5`: Device pixel ratio (Screen pixels per CSS pixel)</li>
  <li class="fragment">`Viewport-Width: 1024`: Width of the viewport in pixels</li>
  <li class="fragment">`Width: 400`: Actual rendered width of an image in CSS pixels</li>
  <li class="fragment">`Device-Memory: 0.5`: The devices's main memory in GiB</li>
</ul>

---

The following additional Client Hints are currently being specified...

<ul>
  <li class="fragment">`ECT: 3g`: Current overall effective network type</li>
  <li class="fragment">`RTT: 300`: Current effective round-trip time in milliseconds</li>
  <li class="fragment">`Downlink: 1`: Current effective bandwidth in Mb/s</li>
</ul>

---

Client Hints use must be activated/opted in by the server:

```
Accept-CH: DPR, Width, Viewport-Width
```

<p class="fragment">Which means that the first call to a server will always be w/o Client Hints</p>
---

Servers may also tell the client for how long it wants there hints sent over (as a time delta from now):

```
Accept-CH: DPR, Width, Viewport-Width
Accept-CH-Lifetime: 86400
```

In this case the server wants to get sent Client Hints for the next 24 hours.
---

After activating Client Hints, this is what the browser will start sending:

```
GET image.jpg
Accept: image/webp,image/*,*/*;q=0.8
DPR: 2
Viewport-Width: 1024
Width: 508
```
---

When replying with the requested image, the server adds the DPR of the resource so that the result is not interfering with the browser's layout mechanics:

```
Content-DPR: 2
```
---
<!-- .slide: data-background="" data-state="inverted" -->

## Controlling Cache Proxies
---

The `Vary` response header is solely meant for proxies (and CDNs) and tells them how to act on caching.

Sadly it's easy to get it wrong and any mistake hard to spot!
---

The `Vary` response header...

<p class="fragment">...needs to be sent by your server</p>
<p class="fragment">...always refers to headers sent by the clients</p>
---

A few examples...

<ul>
  <li class="fragment">`Vary: Save-Data`: stripped down vs. full content</li>
  <li class="fragment">`Vary: Accept`: file format variation, e.g. JPEG vs. WebP</li>
  <li class="fragment">`Vary: Accept-Encoding`: compression type: gzip vs. brotli</li>
  <li class="fragment">`Vary: DPR`: client hint for display density</li>
  <li class="fragment">`Vary: Save-Data, Accept, Accept-Encoding, DPR`: look at all four</li>
</ul>

---

Set `Cache-Control: private` for authenticated users, so that their responses don't get cached by proxies.

<p class="fragment">Then only the client is allowed to cache (`private` as opposed to `public`)</p>

---

## Resource Hints

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

If the server is HTTP/2 Push enabled it will force push any (local) resource listed with a Link header. 

<p class="fragment">
If you don't want that to happen, add a `nopush` attribute to it:
<br>
```
Link: <https://domain.com/styles.css>; rel=preload; as=style; nopush
```
</p>
---

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

<ul>
  <li class="fragment">Marketing department uses Google Tag Manager and you don't want bad programming practices to be introduced onto your site</li>
  <li class="fragment">People with some sort of access to the HTML output add render blocking A/B testing</li>
  <li class="fragment">These same people inserting badly written code</li>
  <li class="fragment">You have 3rd Party on your site but you want to limit its negative impact</li>
  <li class="fragment">You have advertisement using video on your site and you don't want it to autoplay</li>
</ul>

---

Out of the many policies existing the following ones are the most interesting ones for web performance:

<ul>
  <li class="fragment">`autoplay`: Allow or disallow autoplaying video</li>
  <li class="fragment">`sync-script`: Allow or disallow synchronous script tags</li>
  <li class="fragment">`document-write`: Allow or disallow `document.write`</li>
  <li class="fragment">`lazyload`: Force all images and iframe to load lazily (as if all had lazyload-attribute)</li>
</ul>

---

...continued:

<ul>
  <li class="fragment">`image-compression`: restrict images to have a byte size no more than 10x bigger than their pixel count</li>
  <li class="fragment">`maximum-downscaling-image`: restrict images to be downscaled by more than 2x</li>
  <li class="fragment">`unsized-media`: requires images to have a width & height specified, otherwise defaults to 300 x 150</li>
  <li class="fragment">`layout-animations`: turns off CSS animation for any property that triggers a re-layout (e.g. `top`, `width`, `max-height`)</li>
</ul>

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

![Server Timings](images/server-timings.jpg)

---
<!-- .slide: data-background="images/backgrounds/thankyoupage.jpg" data-state="inverted faded" -->

<br><br><br><br><br><br>
## That's about it. Thank you!

* Slides: [http://schepp.github.io/HTTP-headers](http://schepp.github.io/HTTP-headers)
* Twitter: [@derSchepp](https://twitter.com/derSchepp) (English)
* Podcast: [Working Draft](http://workingdraft.de) (German)
---

## Links

[Delivering Fast and Light Applications with Save-Data](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/save-data/)
[The headers we don't want](https://www.fastly.com/blog/headers-we-dont-want)
[Cache-Control: immutable](http://bitsup.blogspot.com/2016/05/Cache-Control-immutable.html)
[Best Practices for Using the Vary Header](https://www.fastly.com/blog/best-practices-using-vary-header)
[Deploying WebP via Accept Content Negotiation](https://www.igvita.com/2013/05/01/deploying-webp-via-accept-content-negotiation/)
[What Are Client Hints and Are They Worth Implementing](https://www.keycdn.com/blog/client-hints)
[Preload](https://w3c.github.io/preload/)
[Feature Policies List](https://www.chromestatus.com/features/6218263637786624)
[WICG Feature Policy](https://github.com/WICG/feature-policy)
[Server Timing Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Server-Timing)