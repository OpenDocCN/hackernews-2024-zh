<!--yml
category: 未分类
date: 2024-05-27 14:31:16
-->

# DNS over HTTPS is not what I thought

> 来源：[https://www.petefreitag.com/blog/dns-over-https/](https://www.petefreitag.com/blog/dns-over-https/)

A few months ago I was on a mission to remove some of the old broken links on my blog. I started blogging back in 2002, so many of the sites that I linked to twenty years ago were no longer active, or no longer under the same ownership. I decided to start this task by weeding out any domains that no longer resolved over DNS.

While I knew of a few traditional methods to query DNS, I thought this was a good chance to explore the *DNS over HTTPS* or *DoH* protocol. It is a fairly *new* standard for accessing DNS records over an encrypted https connection. The RFC (8484) was published in October of 2018.

What is cool about DNS over HTTPS is that, well, it uses HTTPS. Because HTTP clients are plentiful and well understood by many developers, it *should* make for a pretty simple implementation.

### DNS Query using curl

Suppose we want to query the `A` record for example.com, we can make a request using curl like this:

```
curl -H 'Accept: application/dns-json' 'https://cloudflare-dns.com/dns-query?name=example.com&type=A'
```

In this case I'm using CloudFlare's public DNS over HTTPS server `cloudflare-dns.com`, but we can also use Google's public DNS over HTTPS server: `dns.google` with curl like this:

```
curl -H 'Accept: application/dns-json' 'https://dns.google/resolve?name=example.com&type=A'
```

Really the only thing slightly tricky about this, is that you have to pass the http request header:

```
Accept: application/dns-json
```

From there it is just a matter of passing in the name that you want to look up, and the DNS record type using query string parameters.

The response we'll get from the server looks something like this:

```
{
  "Status": 0,
  "TC": false,
  "RD": true,
  "RA": true,
  "AD": true,
  "CD": false,
  "Question": [
    {
      "name": "example.com",
      "type": 1
    }
  ],
  "Answer": [
    {
      "name": "example.com",
      "type": 1,
      "TTL": 85506,
      "data": "93.184.216.34"
    }
  ]
}

```

If you were to make a request for a hostname that does not have an A record, you might get a response like this:

```
{
    "Status": 3,
    "TC": false,
    "RD": true,
    "RA": true,
    "AD": false,
    "CD": false,
    "Question": [
        {
        "name": "invalid-12345-example.com",
        "type": 1
        }
    ],
    "Authority": [
        {
        "name": "com",
        "type": 6,
        "TTL": 900,
        "data": "a.gtld-servers.net. nstld.verisign-grs.com. 1706288745 1800 900 604800 86400"
        }
    ]
    }

```

You'll notice in this case that there is no `Answer` array, and the `Status` is `3` instead of `0`. The status of 3 corresponds to the NXDOMAIN (code 3) response from a DNS server. Here are some of the possible status values:

*   `0` (NOERROR): The dns query was successful
*   `1` (FORMERR): The dns query format was invalid.
*   `2` (SERVFAIL): The server failed to process the dns query due to a problem.
*   `3` (NXDOMAIN): The queried domain name does not exist.
*   `4` (NOTIMP): The server does not support the requested query type.
*   `5` (REFUSED): The server refuses to answer the query for policy reasons.

This makes it pretty easy to see if the DNS query was successful or not.

Next, you are probably wondering what those boolean keys are:

*   `TC` (Truncated) - Indicates whether the response has been truncated due to size limitations.
*   `RD` (Recursion Desired) - Indicates whether the client requested recursive resolution.
*   `RA` (Recursion Available) - Whether the server supports recursive resolution
*   `AD` (Authentic Data) - Indicates if the response has been validated using DNSSEC (Domain Name System Security Extensions)
*   `CD` (Checking Disabled) - Whether the client has requested that DNSSEC validation be disabled.

### It's not RFC 8484, DoH!

It turns out what I was using is not actually the RFC 8484 DNS over HTTPS, but rather an implementation inspired by a 2013 draft specification called *JSON format to represent DNS data draft-bortzmeyer-dns-json-01*.

This actual RCF 8484 DNS over HTTPS specification is much much harder to work with, because you have to send binary encoded messages confirming to RFC 1035\. For example if you want to send a A record lookup of `www.example.com` you could do it like this:

```
curl 'https://dns.google/dns-query?dns=AAABAAABAAAAAAAAA3d3dwdleGFtcGxlA2NvbQAAAQAB' | cat -v
```

The dns query is a url safe base64 encoding of the binary DNS record request.

I'm using `cat -v` to show the response since the binary response contains characters your terminal will hide by default.

### In summary

It was interesting to find out that what I thought was *DNS over HTTPS* is actually not what the RFC 8484 standard defines. What CloudFlare, and Google provide is just a really handy JSON api implementation based upon a 2013 IETF Internet-Draft.

There is in fact an RFC 8427 titled *Representing DNS Messages in JSON*, but this is not the format that CloudFlare / Google's servers are using. It is more closely aligned to provide a mapping from the binary format to a JSON representation.

I'll probably stick to using the JSON api, rather than the actual DNS over HTTPS protocol, but it's good to know there is a difference.