<!-- .slide: data-background="images/backgrounds/security.jpg" data-state="inverted faded" -->

# SECURITY
---

## Starting easy: CORS

---

When is CORS relevant?

<ul>
  <li class="fragment">Making cross-origin XHR requests</li>
  <li class="fragment">Making cross-origin Fetch requests</li>
  <li class="fragment">Using cross-origin images in a `<canvas>`</li>
</ul>

<p class="fragment">Otherwise you don't need to bother about CORS.</p>

---

Once you **do want** to make a reading cross-origin Fetch request, then the receiving server must send a header allowing this:

```
Access-Control-Allow-Origin: https://your-site.com
```

or simply

```
Access-Control-Allow-Origin: *
```

---

Another use case would be if you need to paint an image from another host on a `<canvas>`.

Then you need to add a `crossorigin` attribute to the image...

```html
<img src="https://other-origin.com">
``` 

... and set `Access-Control-Allow-Origin: *` on the server.
 
---

If you need to read from a server that...

* that has basic auth
* that has JWT auth
* that has session based auth (cookies)

...then the wildcard is not allowed anymore.

```
Access-Control-Allow-Origin: https://your-site.com
```

---

This has nothing to do with

```
Access-Control-Allow-Credentials: true
```

<p class="fragment">This just tells the browser that it may expose authentication cookies or tokens to JavaScript.</p>

---

If you want to modify data be sending e.g. a POST, PUT, or DELETE request, then the server explicitely needs to allow those methods:

```
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Origin: *
```

<p class="fragment">Don't forget to also set `GET` and `OPTIONS`.</p>

---

Since the browser will only see the `Access-Control-Allow-Methods` upon its first request to the server, and since you don't want the `POST` to be sent from the wrong origin, what it will do first is do a so-called...

<br>

> Preflight Request

---

## Preflight Request

The Preflight Request is a very light-weight `OPTIONS` request to the target URL.

It will check the answer for suited CORS header.

Only if they match will it follow up with the real `POST` request.

---

## Encryption

---
Force browsers to load your site over HTTPS only via HTTP Strict Transport Security (HSTS) header:

```
strict-transport-security: max-age=31536000
```

<p class="fragment">The `max-age` directive specifies the time, in seconds, that the browser must now use only HTTPS to access the site that issued the policy.</p>

<p class="fragment">Better start off with a low value!</p>

---
And you can include subdomains as well:

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```
---
Or you use one of the HTTPS-only TLDs, like `.dev` or `.app`<br>(there is 45 of them right now)
---
If you want your domain to be included in that internal browser list then add the `preload` flag like so

```
Strict-Transport-Security: max-age=31536000; 
  includeSubDomains; 
  preload
```

and afterwards submit your site to the list managers: [https://hstspreload.appspot.com/](https://hstspreload.appspot.com/) 
---
On top of that, Content Security Policy (CSP) helps you keep your security model intact:

```
Content-Security-Policy: block-all-mixed-content; 
  form-action https:
```

or

```
Content-Security-Policy: default-src https:;
  form-action https:
```
---
What if you need to migrate a legacy site from HTTP to HTTPS?

<p class="fragment">I mean like a super ugly one</p>

<p class="fragment">Stuffed with HTTP-non-S-resources</p>
---
Content Security Policy will help us again!

```
Content-Security-Policy: upgrade-insecure-requests;
```

Transparently upgrades all embedded resources to be requested via HTTPS.
---
This: 

`<img src="http://.../logo.png">`

<p class="fragment">now <em>virtually</em> becomes this:<br> 
<br>
`<img src="https://.../logo.png">`</p>
---
## Protecting yourself from XSS attacks via CSP
---
Limit the amount of damage an attacker can cause by injecting a manipulated `<base>` element:

```
Content-Security-Policy: base-uri 'self'
```
---
Whitelist the sources from which subresources can be included, e.g. stylesheets, fonts, scripts...  

```
Content-Security-Policy: default-src http://*.mydomain.com;
```
---
Or, more finegrained:
```
Content-Security-Policy: 
  font-src http://fonts.googleapis.com; 
  style-src 'self';
  script-src http://*.mydomain.com;
  img-src http://images.mydomain.com;
  frame-src 'self';
  worker-src 'self';
  connect-src http://*.mydomain.com
```
---
CSP also allows you to specifically whitelist the use of certain browser features on a given page, via `sandbox` directive:

```
Content-Security-Policy: sandbox <value>;
``` 

<ul class="multicolumn">
  <li class="fragment">`allow-downloads-without-user-activation`</li>
  <li class="fragment">`allow-forms`</li>
  <li class="fragment">`allow-modals`</li>
  <li class="fragment">`allow-orientation-lock`</li>
  <li class="fragment">`allow-pointer-lock`</li>
  <li class="fragment">`allow-popups`</li>
  <li class="fragment">`allow-popups-to-escape-sandbox`</li>
  <li class="fragment">`allow-presentation`</li>
  <li class="fragment">`allow-same-origin`</li>
  <li class="fragment">`allow-scripts`</li>
  <li class="fragment">`allow-top-navigation`</li>
</ul>

<span class="fragment">_</span>
---
## Stopping other pages from framing you (2020 edition)
---
The old way of stopping your site being framed:

```
X-Frame-Options: DENY
```

the new way to do that is, again, via CSP:

```
Content-Security-Policy: frame-ancestors 'self';
```
---
## Simulating CSP
---
If turning on CSP scares you at the beginning, you can still run it as a simulation and log away any potential violations for further inspection.

To do that, switch:

```
Content-Security-Policy: ...
```

out for:

```
Content-Security-Policy-Report-Only: ...; 
  report-uri https://endpoint.com; 
```

...adding a reporting URL to send the logs to. 
---
## Protect your users' privacy via Referrer Policy
---
```
Referrer-Policy: no-referrer
Referrer-Policy: no-referrer-when-downgrade
Referrer-Policy: origin
Referrer-Policy: origin-when-cross-origin
Referrer-Policy: same-origin
Referrer-Policy: strict-origin
Referrer-Policy: strict-origin-when-cross-origin
Referrer-Policy: unsafe-url
```

`no-referrer-when-downgrade` is the default,<br>whereas `unsafe-url` will always send the referrer.
---
## Security "Metadata"
---
Right now a server processes every request in the same way, regardless of context.

This fact is being used in Cross-Site-Request-Forgery (CSRF) attacks.

e.g. an image gets injected that points to<br>`http://fritz.box/setpassword?value=1234`<br>(`fritz.box` being a household's router)
---
The new "Fetch Metadata Headers" want to mitigate this threat. 

They require browsers to send extra metadata depending on how the request came to be.

```html
<img src="http://fritz.box/setpassword?value=1234">
```

```
Sec-Fetch-Dest: image
Sec-Fetch-Mode: no-cors
Sec-Fetch-Site: cross-site
```

<p class="fragment">Now the server at `http://fritz.box` can decide not to execute it.</p>

---

* `Sec-Fetch-Dest: document`: top level navigation & `<iframe>`
* `Sec-Fetch-Dest: image`: `<img>` or `<picture>`
* `Sec-Fetch-Dest: worker`: Web Worker
* `Sec-Fetch-Dest: empty`: `fetch()` API

---

* `Sec-Fetch-Mode: navigate`: top level navigation
* `Sec-Fetch-Mode: nested-navigate`: `<iframe>`
* `Sec-Fetch-Mode: cors`: request is bucketed in the "CORS" requests 
* `Sec-Fetch-Mode: no-cors`: request is bucketed in the no "CORS" requests 
* `Sec-Fetch-Mode: same-origin`: `fetch()` API 

---

* `Sec-Fetch-Site: same-origin`: from `example.com` to `example.com`
* `Sec-Fetch-Site: same-site`: from `example.com` to `assets.example.com`
* `Sec-Fetch-Site: cross-site`: from `example.com` to `assets.xyz.com`
* `Sec-Fetch-Site: none`: user entered the address in the URL bar

---

### Further examples

```
// Top-level navigation https://example.com 
// to https://example.com/ caused
// by a user’s click on an in-page link:
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: same-origin
Sec-Fetch-User: ?1
```

---

```
// Top-level navigation from https://example.com 
// to https://not-example.com/ caused
// by JavaScript or <meta>:
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: cross-site
```

---

```
// <iframe> navigation from https://example.com 
// to https://example.com/ caused
// by JavaScript or <meta>:
Sec-Fetch-Dest: document
Sec-Fetch-Mode: nested-navigate
Sec-Fetch-Site: same-origin
```

---

<!-- .slide: data-background="images/backgrounds/security.jpg" data-state="inverted faded" -->

&lt;/security&gt;