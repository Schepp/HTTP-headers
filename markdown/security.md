<!-- .slide: data-background="images/backgrounds/security.jpg" data-state="inverted faded" -->

# SECURITY
---
## Encryption
---
Force browsers to load you site over HTTPS only via HTTP Strict Transport Security (HSTS) header:

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
Or you use on of the HTTPS-only TLDs, like `.dev` or `.app`<br>(there is 45 of them right now)
---
If you want you domain to be included in that internal browser list then add the `preload` flag like so

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
Content Security Policy will hep us again!

```
Content-Security-Policy: upgrade-insecure-requests;
```

Upgrades all embedded resources to be requested via HTTPS.
---
This: `<img src="http://.../logo.png">`

<p class="fragment">now becomes this: `<img src="https://.../logo.png">`</p>

<p class="fragment">before the request is ever issued.</p>
---
## Protecting yourself from XSS attacks via CSP
---
Limit the amount of damage an attacker can cause by setting a `<base>` element:

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
Content-Security-Policy: sandbox allow-same-origin;
``` 

<ul class="multicolumn">
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
## Stopping other pages from framing you (2019 edition)
---
The old way of stopping your site being framed:

```
X-Frame-Options: DENY
```

the new way to do that is again via CSP:

```
Content-Security-Policy: frame-ancestors 'self';
```
---
## Simulating CSP
---
If turning on CSP scares you at the beginning, you can still run it as a simulation and log away any potential violations for futher inspection.

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

e.g. a image gets injected that points to<br>`http://fritz.box/setpassword?value=1234`<br>(`fritz.box` being a household's router)
---
The new `Sec-Metadata` header wants to mitigate this threat. 

It requires browsers to send extra metadata depending on how the request came to be.

```
// <picture>
Sec-Metadata: initiator=imageset, destination=image, 
  site=cross-site, target=subresource

// Top-level navigation
Sec-Metadata: initiator="", destination=document, 
  site=cross-site, target=top-level, cause=user-activation

// <iframe> navigation
Sec-Metadata: initiator="", destination=document, 
  site=same-site, target=nested, cause=forced
``` 

<p class="fragment">Now the server can decide what to do with it.</p>
---
<!-- .slide: data-background="images/backgrounds/security.jpg" data-state="inverted faded" -->
