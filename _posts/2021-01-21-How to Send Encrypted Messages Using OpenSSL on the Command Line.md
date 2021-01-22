---
layout: post
title: How To Compose a Singleton in C++ Qt
date: 21-01-21 06:00:00 -0800
categories: OpenSSL, SSL, Encryption, Socket, Network, Bash, Linux
---
![Post Banner](https://i.imgur.com/MFVpkYA.jpg)

# Intro Blah Blah Blah

I've done a lot of work with network socket communications, but what if I want that communication encrypted? SSL is the best way to go about this. A problem with SSL is getting information: If you Google SSL, the default assumption is https communication. Everyone wants to assume you're running a web site and want to buy a certificate or set up your secure web browsing for your site. But if you're me, you aren't a web dev, and you don't care about all that.

While it's true, SSL is the power behind secure web development, and it's likely the most common use for it, it's only one part of what SSL is capable of. SSL will encrypt *any* network communication. And since my focus is on writing network enabled applications that have nothing to do with a web server or a web browser, this is still exactly what we're looking for to do the job.

# What You're Here For

First up, install OpenSSL: `sudo dnf install openssl`
As usual, if your package manager isn't dnf, use the one appropriate for your system. OpenSSL is pretty much ubiquitous across platforms.

For this project, we're going to use **openssl** on the command line the same way we would use **ncat**.

## Create a Certificate

The internet will caution you against anything other than buying into the certification racket. If you're managing web sites, your hands are pretty much tied, and you are stuck slapping cash down at one of the extortionist Certificate Authorities. This problem has recently been relieved, thanks to [Let's Encrypt](https://letsencrypt.org/).

However, for application programming, I'm going to say that this isn't necessary. OpenSSL provides the means for creating a certificate – intended for development testing. If you're the developer in charge of creating both the server and the client, then you can authenticate your own certificate.

In this case, we're just running commands on the command line. You can use this to create a private chat room style environment between you and someone else. You can meet in a secluded parking garage under cover of night and exchange the certificate on a USB drive, and forever after be secure in knowing that your encrypted messages are being exchanged with the right person.

The [man page](https://www.openssl.org/docs/man1.1.0/man1/req.html) for `openssl-req` is great. There are lots of examples and trouble shooting tips.

Here's a quick and dirty command for getting a certificate:

```
    openssl req -x509 -nodes -days 365 -newkey rsa -keyout keyfile.key -out certfile.crt
```

Options breakdown:

    -x509               Causes "output of a self signed certificate instead of a certificate request... a large random number will be used for the serial number"

    -nodes             "... if a private key is created it will not be encrypted" I am uncertain what the implication is.

    -days               Specifies the number of days the certificate is "certified" for. Default is 30 days. Uncertain what "certified" means, or what happens after the time is up. Is it possible to have a certificate that doesn't expire?

    -newkey rsa         We're familiar with rsa keys from ssh work. This generates an rsa key of the size indicated in "the config" (likely at /usr/local/ssl/openssl.cnf). Use -rsa:1024 to generate a 1024 bit rsa key.

    -keyout keyfile.key Output file for the key. Config will define any default file if not provided. Man examples all show "key.pem" so maybe this is a better name (or at leas extension).

    -out certfile.crt   "specifies the output filename" according to the man, but doesn't tell what's outputted. Also states that it defaults to stdout if not provided. This example suggests that it's the certificate that's output. Man examples show this filename to be req.pem.

The man example is more like this:

    openssl req -x509 -newkey rsa:2048 -keyout key.pem -out req.pem

- It skips the -nodes and encrypts the private key
- It skips the days and defaults to 30
- It sets the rsa key to a nice size
- The rest is basically unchanged

## The Server

```
    openssl s_server -accept <port> -key <keyfile> -cert <certfile>
```

Here is the man page:
https://www.openssl.org/docs/man1.1.1/man1/openssl-s_server.html

Here is a breakdown of the options:

    -accept <port>      According to the man page, this will accept an IP and port. It's not clear from the text, but it looks like something like 192.168.0.1:4433 or *:4433 is the valid format. An example would be nice. There is an example for a -www connection where just the port is used (openssl s_server -accept 443 -www) to imitate a secure web server.

    -key <keyfile>      Is the private key to use. I'm again assuming this is a filepath to the file, similar to how we specify the rsa file in an ssh command. "If not specified then the certificate file will be used." Assume that the certificate is like the default rsa file in ssh after creation?

    -cert <certfile>    "If not specified, then the filename "server.pem" will be used."

## The Client

```
    openssl s_client -connect host:port
```

This appears pretty straight forward. Other options allow you to get specific about what cert/key algorithms you want to blacklist or whitelist. But unless you have a reason to get that complicated, I don't see why you need to get into it.

## The Transaction

Now in one terminal, we're going to run the server, and in another terminal, we're going to run the client.

The location of your certificate doesn't matter, as long as the options you use for the certificate points to the file location. For this project, our certificate will be in the working directory.

First make your certificate. Set the `-days` option for 1 for now. Tomorrow you can come back and see what happens when you use an expired certificate. For all the questions, you can make it easy on yourself for now and just press enter for the default on each. For real world use, you may want to fill in some of the fields.

### Run Server

Input:

```
openssl s_server -accept 60100 -key keyfile.key -cert certfile.crt
```

Output:

```
Using default temp DH parameters
ACCEPT

```

Like **ncat** you will have an open prompt for stdin and stdout.
You can also instead pipe in a server response for an immediate respond and hangup the same way you do with **ncat** (`echo Acknowledged | openssl s_server ...`).

## Run Client

Here you'll get a lot of information as the connection is established:

Input:

```
openssl s_client -connect 127.0.0.1:60100
```

Output:

```
CONNECTED(00000003)
depth=0 C = XX, L = Default City, O = Default Company Ltd
verify error:num=18:self signed certificate
verify return:1
```

Note we have some errors. This one just tells us that we've committed the cardinal sin of creating our own certficate, and that if this were some environment that has no control over its clients and servers (like the web) there is no way to actually verify that we are talking to who we think we are without that third party certificate validation.

```
depth=0 C = XX, L = Default City, O = Default Company Ltd
verify error:num=10:certificate has expired
notAfter=Jan 12 20:03:30 2021 GMT
verify return:1
depth=0 C = XX, L = Default City, O = Default Company Ltd
notAfter=Jan 12 20:03:30 2021 GMT
verify return:1
```

This error is because I waited some time after my expiration date to use the certificate. If we were running this inside some other application, we would detect this error and act on it. Maybe email a reminder to the server owner to keep up on their certifications.

```
---
Certificate chain
 0 s:/C=XX/L=Default City/O=Default Company Ltd
   i:/C=XX/L=Default City/O=Default Company Ltd
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIDWjCCAkKgAwIBAgIJAINJEd92e9BHMA0GCSqGSIb3DQEBCwUAMEIxCzAJBgNV
BAYTAlhYMRUwEwYDVQQHDAxEZWZhdWx0IENpdHkxHDAaBgNVBAoME0RlZmF1bHQg
Q29tcGFueSBMdGQwHhcNMjEwMTExMjAwMzMwWhcNMjEwMTEyMjAwMzMwWjBCMQsw
CQYDVQQGEwJYWDEVMBMGA1UEBwwMRGVmYXVsdCBDaXR5MRwwGgYDVQQKDBNEZWZh
dWx0IENvbXBhbnkgTHRkMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA
mniZr4jNQ/qqWZE/D78lo9VM5uIvQp+wOZD38nBbRpFzxrLcU2LMDni2fH2OvTZP
wUY062Fw7Fauhk5m7Gmq3ATQybiAUlL02jEYPpAmjw/tkwla24BU4Njb9zrhJIB/
COQYl8iJ9gVqz9XGBNvQzj5HHDH56WlUmKYTn2V2IDNg/lL9IYFk7XK2h/vbRhJi
bnQclZ/XSmkCJ8LIY+KYIJwyBEUarc5wV2J2/Nznrab2/R+Knt/1/+q2d8812Uj7
FlfWjWxN427zCNzrnW7SrOSlCSOHc6UU70i75QBBQmdCW+SvA26VrqtBNiGGbnDl
tl5PiB2nxgP8VEGSFJf+CQIDAQABo1MwUTAdBgNVHQ4EFgQUtYghPz9sMaPIrmjo
ktpSKloF9qMwHwYDVR0jBBgwFoAUtYghPz9sMaPIrmjoktpSKloF9qMwDwYDVR0T
AQH/BAUwAwEB/zANBgkqhkiG9w0BAQsFAAOCAQEAO7ymnHTNdc8n5hhjwMlGfeWu
J46N+GAMXGhx4hTZ9zzd9TZ6MgGUdsALlMiC9xl5WOuijnteJvW4GDGXy5wqu2hB
Ja9BNwbHiOVHo0PJ7E90oD2IaqCYWumuF2MmDCkDqIzNp4mD8RKFWbpydsJ8svxK
ThgAL6yS3iy8auBig0GumrmO4yzmXQ132zaloRTnFDiYpRi4qbFQSypxb5c5peWw
HIurAxm5iOANZaHJkPJW5zzqWVRJPGzRdHsptjcCjNAH+QDjaTrDok1botHAe5B5
sT7NBYaeWkGwkRPanBggqVQeYZ9s4dcudhP74DpNsf9QCBW8DR11qRH9E7ng+w==
-----END CERTIFICATE-----
```

Here's the certificate from the server. Again, if we were running this from inside an application, we would be able to verify the certificate is the one we expect by comparing this one. For now, your meat-brain will have to do this work.

```
subject=/C=XX/L=Default City/O=Default Company Ltd
issuer=/C=XX/L=Default City/O=Default Company Ltd
---
No client certificate CA names sent
Peer signing digest: SHA512
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 1487 bytes and written 347 bytes
Verification error: certificate has expired
---
New, TLSv1.2, Cipher is ECDHE-RSA-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-RSA-AES256-GCM-SHA384
    Session-ID: FFD7F1DB2B6D716D4B58133B0E048B48B358A713E2CAB3A312BC071E965320C6
    Session-ID-ctx:
    Master-Key: D943B9AA4B89300D07EFACBE6A2217C7A96BCF25E57EB43E58A994FA9D3EFABC9F0B9E593AE4BCC02049BF712741E17B
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 7200 (seconds)
    TLS session ticket:
    0000 - d5 6a 3d 17 00 4d 04 40-b1 a1 5b 8a ab 99 04 f5   .j=..M.@..[.....
    0010 - e2 34 97 71 dd 0b a9 4d-b0 70 5d 53 75 0b ed d3   .4.q...M.p]Su...
    0020 - bd e8 23 54 b6 9e e6 ba-c4 a8 8d 75 5f c1 3b 4c   ..#T.......u_.;L
    0030 - cf 6d dc 07 05 d3 c0 1a-e4 07 38 37 e3 50 07 9b   .m........87.P..
    0040 - 1f 42 3e d3 d5 0e 4c 92-23 02 aa 42 15 90 a7 42   .B>...L.#..B...B
    0050 - 71 68 58 c3 14 03 a6 be-13 bb 31 5f e1 1c 31 a7   qhX.......1_..1.
    0060 - 42 a2 d9 73 4d dd 0a e8-77 c7 90 1c 3f 0e 3a 90   B..sM...w...?.:.
    0070 - 23 b6 2f 84 16 10 cd e4-6c cd 14 01 68 39 c2 52   #./.....l...h9.R
    0080 - 65 9c a3 1e e6 0b 82 d5-3a 77 8d fe 5a 57 39 23   e.......:w..ZW9#
    0090 - 7c 65 c7 af 22 31 1f b6-e1 30 06 84 eb f1 11 b0   |e.."1...0......

    Start Time: 1611340988
    Timeout   : 7200 (sec)
    Verify return code: 10 (certificate has expired)
    Extended master secret: yes
---

```

Now our client has its open stdin/stdout prompt after all that. You will also notice on the server a big header has appeared talking about all the fancy encryption it recognizes.

At this point, you can type on the client terminal, and it will be sent securely to the server, and vice versa. Go ahead and have a short conversation with yourself.

That's it! If you disconnect the client, the server will note it, but still be waiting for new connections. If you disconnect the server while the client is attached, the client will spit out a `read:errno=0` before exiting.

## Renewing Your Certificate

Two commands to renew:

```
openssl req -in new.csr -noout -text
openssl x509 -req -days 1 -in new.csr -signkey keyfile.key -out updated.crt
```

And a command for you to see your expiration date:

```
openssl x509 -in updated.crt -noout -enddate
```
