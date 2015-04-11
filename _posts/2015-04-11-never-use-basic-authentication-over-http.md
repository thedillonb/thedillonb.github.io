---
layout: post
published: false
title: Never Use Basic Authentication over HTTP
categories: Programming
tags: 
  - http
  - api
  - authentication
---

Basic authentication is an extremely popular means of authentication due to it's simplicity and ease at which it can be implemented. Unfortunately, over HTTP, it's also the **absolute worst way you could have your clients authenticate**!  Basic authentication utilizes the `Authentication` HTTP header to transmit the clients username and password credentials. The client simply takes the given username and password, base64 encodes them both, and inserts them into the HTTP header when making a request to a server.

```http
GET /my-cats HTTP/1.1
Authentication: Basic ZXhhbXBsZTpleGFtcGxl
Host: cat-collection.com
```

The server can simply base64 decode the `Authentication` header to retrieve the username and password to authenticate against. The result is a list of my cats! Unfortunately, as I indicated above, basic authentication should **never** be used over the HTTP protocol. 

The HTTP protocol, unlike it's smarter sibling, HTTPS, is not encrypted. Which means you might as well assume everyone on the internet can see what you send. While my collection of cats is not sensitive information, I did happen to stick my credentials in the `Authentication` header when I made the request, which means I might as well be giving my credentials to malicious hackers since it's a cake walk to retrieve them from my un-encrypted request. 

Whether it's someone in the same coffee shop as you, sniffing Wifi packets, or someone who is half way around the world that happened to jump in as the middle of your request, your credentials will be completely visible outside of HTTPS as well as easily decoded since base64 is **not** a means of encryption! For this reason, you should **never** use basic authentication of HTTP! Basic authentication should only be considered over HTTPS.


 



