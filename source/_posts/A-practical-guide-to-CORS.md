---
title: A practical guide to CORS
date: 2017-10-08 23:47:06
tags: [web, JavaScript]
---

A practical guide for things you need to know about enabling [Cross-Origin Resource Sharing (CORS)](https://www.w3.org/TR/cors/).
<!-- more -->

# Why we need CORS
Web pages are under the restrictions of **same-origin policy**, which means scripts contained in a web page cannot access data in another page with different _origin_. For example, `http://www.example.com:81/dir/other.html` cannot be accessed by `http://www.example.com/dir/page.html` because the port is different. You can find more detailed explanations on [wiki](https://en.wikipedia.org/wiki/Same-origin_policy#Origin_determination_rules).

However, there are some cases we want to enable cross-origin resource sharing.
- Resources you would like to be loaded from separate domains like images, CSS and script files.
- You would like to provide a public API to be consumed by any client, or clients specified by a whitelist.
- When developing, your frontend web app is running on a different port from your backend server.

# Demo setup
So let's get started. I've prepared a simple [KOA](http://koajs.com/) server to demonstrate the idea. You can find the demo on [Github](https://github.com/xg-wang/cors-guide) and follow the tag for each stage.

```bash
git clone git@github.com:xg-wang/cors-guide.git
cd cors-guide
npm i
npm start
```

The server runs at http://localhost:3000/ and frontend runs at http://localhost:8000/

## 01. CORS fail
First we simply retrieve the data through API running on port 3000:

```javascript
fetch('http://localhost:3000/')
  .then(res => res.text())
  .then(res => responseElement.innerHTML = res)
  .catch(err => responseElement.innerHTML = err)
```

Click the button to send the request, you could see the returned response is "TypeError: Failed to fetch".

![fail](/2017/10/A-practical-guide-to-CORS/fail.png)

Now open your dev console, you can find the following error:
> Failed to load http://localhost:3000/: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost:8000' is therefore not allowed access. If an opaque response serves your needs, set the request's mode to 'no-cors' to fetch the resource with CORS disabled.

Because our frontend is hosted on port 3000, and server is running on port 8000, the request from browser is restricted by the **same-origin policy**. To fix it, simply add a header to the response in our server code:·
```
Access-Control-Allow-Origin: http://localhost:8000
```

Now click the button again and you should be able to see the response from server!

![Nails it](/2017/10/A-practical-guide-to-CORS/success.png)

Open up the dev server and check the request and response, you can find the `Origin: http://localhost:8000` header in the request. This is added by the browser automatically.
In the response, we have the header we just added: `Access-Control-Allow-Origin: http://localhost:8000`. This allows our http://localhost:8000 to have access to the API endpoint. You may also specify "*" as a wildcard, thereby allowing any origin to access the resource.

## 02. preflight
This is not the end of story, let's try to add some header to our request as `'Content-Type': 'application/json'`.

Send the request again and it fails. The console has the following error:
> Failed to load http://localhost:3000/: Request header field Content-Type is not allowed by Access-Control-Allow-Headers in preflight response.

### What is a preflighted request/response?
Open up the dev tools network tab, the request sent by the browser is not the request we created by JavaScript. It is called a preflighted request and has the following Headers:
```
Request Method: OPTIONS
Access-Control-Request-Headers: content-type
Access-Control-Request-Method: GET
```
Together with the `Origin: <origin>` header they are all we need for the request. In any access control request, the `Origin` header is always sent.
And `Access-Control-Request-Headers` might also be a comma separated array.

These headers are used to tell the server what the following actual CORS request contains. Here we are asking the server to allow the `Content-Type` header and `GET` method.

After the browser receives the legit preflight response, the actual CORS request is sent.

![MDN example for preflight request](/2017/10/A-practical-guide-to-CORS/prelight.png)
> MDN example for preflight request

### Why there was no preflight request?
In the previous section, our request went well and there is no CORS preflight - it is called a 'Simple Request'.
MDN has a comprehensive [description](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS#Simple_requests) about this.
To summarize, a simple request is a GET|HEAD|POST method, and only contains '[CORS-safelisted request-header](https://fetch.spec.whatwg.org/#cors-safelisted-request-header)'

> A CORS-safelisted request-header is a header whose name is a byte-case-insensitive match for one of
> - `Accept`
> - `Accept-Language`
> - `Content-Language`
> - `Content-Type` and whose value, once extracted, has a MIME type (ignoring parameters) that is > `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`
>
> or whose name is a byte-case-insensitive match for one of
> - `DPR`
> - `Downlink`
> - `Save-Data`
> - `Viewport-Width`
> - `Width`
>
> and whose value, once extracted, is not failure.

By CORS spec, a simple request won't trigger the preflight request.

### How to fix it?
To grant access to the resource, we need to set corresponding headers in the response for the preflight request.
```
Access-Control-Allow-Origin: http://localhost:8000
Access-Control-Allow-Methods: GET
Access-Control-Allow-Headers: Content-Type
```

Several things to notice:
- `...-Methods` and `...-Headers` can be comma separated array.
- `...-Origin` need to be in both preflight response and actual response.
- You can only allow 1 origin, but you can always extract the actual origin from the `Origin` header and allow it based on your whitelist or simply set a wildcard "*".

There are 3 more access control headers you can set:
- `Access-Control-Expose-Headers` lets a server whitelist headers that browsers are allowed to access.
- `Access-Control-Max-Age: <delta-seconds>` indicates how long the results of a preflight request can be cached.
- `Access-Control-Allow-Credentials` will be discussed in next section.

## 03. credentials
By default, in cross-site [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) or [Fetch](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) invocations, browsers will not send credentials ([HTTP cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies) and [HTTP Authentication information](https://developer.mozilla.org/en-US/docs/Web/HTTP/Authentication)).
But they both have option flag to set. Take Fetch for example, there is a `credentials` option:
> The request credentials you want to use for the request: omit, same-origin, or include. To automatically send cookies for the current domain, this option must be provided. Starting with Chrome 50, this property also takes a FederatedCredential instance or a PasswordCredential instance.

If we add this option and send the request:
```
fetch('http://localhost:3000/', {
  headers: {
    'Content-Type': 'application/json'
  },
  credentials: 'include'
}).then(res => res.json())
  .then(res => responseElement.innerHTML = res)
  .catch(err => responseElement.innerHTML = err)
```
The following error will popup in your dev console:
> Failed to load http://localhost:3000/: Response to preflight request doesn't pass access control check: The value of the 'Access-Control-Allow-Credentials' header in the response is '' which must be 'true' when the request's credentials mode is 'include'. Origin 'http://localhost:8000' is therefore not allowed access.

### fix
Remember the `Access-Control-Allow-Credentials`?
Set this header in both preflight and request: `Access-Control-Allow-Credentials: true`. Refresh, send again and it succeeds.

Some more things to notice:
- For a simple request, response without this header will be rejected.
- `Cookie` header will be added to the request if credentials flag set.
- `Access-Control-Allow-Origin: *` fails in this situation, you have to set the allowed origin explicitly.
- It is a good idea to add `Vary: Origin` response header.
> If the server specifies an origin host rather than "*", then it could also include Origin in the Vary response header to indicate to clients that server responses will differ based on the value of the Origin request header.

A more persuasive reason can be found [here](https://github.com/rs/cors/issues/10).

### one more thing
In the Fetch API, you can set [Request.mode](https://developer.mozilla.org/en-US/docs/Web/API/Request/mode)
> The mode read-only property of the Request interface contains the mode of the request (e.g., cors, no-cors, same-origin, or navigate.) This is used to determine if cross-origin requests lead to valid responses, and which properties of the response are readable.

Set this to 'cors' when sending CORS request, more details can be found in the [spec](https://fetch.spec.whatwg.org/#dom-request).

# Summary
In this post we discussed the use cases of CORS and how we can enable it by building a simple API step by step. You can find the full demo on [Github](https://github.com/xg-wang/cors-guide).

For conclusion, to enable CORS for a general CORS request, you need to add the following headers:
- For preflight request which can be filtered by check method is 'OPTIONS':
  - `Access-Control-Allow-Origin`
  - `Access-Control-Allow-Credentials`
- For actual CORS request which is not `OPTIONS` and has a `Access-Control-Request-Method` header:
  - `Access-Control-Allow-Origin`
  - `Access-Control-Allow-Methods`
  - `Access-Control-Allow-Headers`
  - `Access-Control-Allow-Credentials`
  - _(Optional)_ `Access-Control-Max-Age`
  - _(Optional)_ `Access-Control-Expose-Headers`
- Extract origin from `Origin` header and set it together with `Vary: Origin` for each response.