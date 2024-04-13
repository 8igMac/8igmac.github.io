---
title: "POST vs. GET ... When to use which?"
date: 2023-12-15
description: "POST and GET are both HTTP method that can be use to send data, but when to use which?"
summary: "POST and GET are both HTTP method that can be use to send data, but when to use which?"
---

<!-- Problem statement -->
POST is often use to transmit data from API caller to the 
backend server. But GET can also used to transmit data by encoding
them in the query string parameter. So the problem arise: when to use which?
In this article, we are going to compare the difference between
POST and GET request and tell you when to use which when you 
want to transmit data to the backend server.

Before we dive into the topic, we should talk about 2 important concept
in HTTP - Safe and Idempotent.
- **Idempotent**: An HTTP method is **idempotent** if the intended effect on the 
server of making a single request is the same as the effect of making identical requests.
- **Safe**: An HTTP method is **safe** if it doesn't alter the state of the server (i.e., read-only operation). All safe method are also idempotent. Web crawlers also rely on calling safe method.

## GET method
- **Safe and idempotent.**
- **Use case: retrieving resource from the server (e.g., viewing a web page.)**
- Can be cached by CDN, bookmarked by browser.
- Search engine should only crawl GET request.
- Supply data in the query parameters in the URL.
  - **Less secure** becase the URL can be cached, bookmarked, or logged by the server. (Web servers tend to log the entire requested URL in plain text in their access logs; so sending sensitive information over GET is not a good idea. This applies irrespective of whether HTTP or HTTPS is used.)
  - **URL can only accept ASCII.**
  - **Lenth limit is 2048 characters.**
- Ideal for reloading (i.e., pressing F5), or back buttoned.
- Additional functionality:
  - conditional GET: only fetch if the data expired (to save network bandwidth)
  - partial GET: only fetch data that is not held by the client (to save network bandwidth)

## POST method
- **Neither save nor idempotent.** (Note: PUT method is not safe but idempotent. e.g., editing or creating new if not exist)
- **Use case: Destructive actions such as creation, editing, deleting data. (commonly used in form submition.)**
- Supply data in the request body.
  - Also allow binary data.
  - No length limit.
- POST is more **secure** than GET, because you aren't sticking information into a URL and you can't hit a POST request in address bar of your browser.
- POST make sure all your call in made, GET might use cached version.
  - back button/reload web page (pressing F5) behavior: 
    - Pressing F5 in your browser will resend the last HTTP request (including POST on form data).
    - GET request are re-executed but may not be resubmitted to server if the HTML is stored in the browser cache.
    - POST: the browser will usually alert the user the data will be resubmitted.

## POST or GET ? When to use which?
Rule of thumb: use GET method for retrieving data and action that doesn't
alter the state of the server. Use POST for sending information to the server and actions that might alter the state of the server.

## Reference
- [Safe (HTTP Methods)](https://developer.mozilla.org/en-US/docs/Glossary/Safe/HTTP)
- [Idempotent](https://developer.mozilla.org/en-US/docs/Glossary/Idempotent)


<!-- Info collected from the internet -->





