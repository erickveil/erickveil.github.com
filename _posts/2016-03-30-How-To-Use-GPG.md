---
layout: post
title: How To Use GPG
date: 16-03-30 13:07:11 -0800
categories: 
---

Create
====

To create a key:

```
gpg --gen-key
```

The fingerprint is printed, you should save this in a safe place. It allows you to confirm with others that they have received your actual key. Like a hash check.

If you are on ssh and you run out of random bytes, run this command:

```
rngd -f -r /dev/urandom
```

You can check the amount of available entropy here:

```
cat /proc/sys/kernel/random/entropy_avail
```


Fingerprint
====

Generate the fingerprint at any time:

```
gpg --fingerprint user@emailaddress.com
```


Revocation Certificate
====

Then generate a revocation certificate. Just in case your key is compromised, you can use this to invalidate it.
KEYNAME can be either the ID or the fingerprint.

```
gpg --output filname-revoke.asc --gen-revoke KEYNAME
```


Backup Private Key
====

Backup the key and also keep it in a safe place.

```
gpg --export-secret-keys --armor user@emailaddress.com > filename-private.asc
```


Send Public Key to a Keyserver
====

Make the public key available to others.  This will send the key to the default [public key server](http://pgp.key-server.io/)

```
gpg --send-key KEYNAME
```

To chose a different server, enter its address in this line:

```
gpg --keyserver hkp://php.mit.edu --send-key KEYNAME
```


Create a Public Key File
====

Alternatively, you might want the key to deliver via a different means:

```
gpg --export --armor user@emailaddress.com > filename-public.asc
```


Import a Key You Have Recieved
====

When you want to use someone else's public key, you will need to import it.

```
gpg --import filename-public.asc
```

After import, and that you trust that the key is from the correct person, sign the key. 
Once signed, output the signed version of that key.

```
gpg --sign-key friend@emailaddress.com
gpg --export --armor friend@emailaddress.com
```

This gives this public key your personal verification that it is genuine. You act as the Certificate Authority in this case. Others who receive this key can now see that you claim that this is a valid key.

If the signed key was from your public key, then when you get it back, you should import it again.

```
gpg --import signed_key.asc
```


Revoke Your Key
====

If the key is compromised, you can make it unusable.
First, import the revocation certificate you created.

```
gpg --import revoke.asc
```

Then send the revocation notice to a server, even if this is not how the key was distributed.

```
gpg --keyserver subkeys.php.net --send KEYNAME
```

How and Where to Use Your Keys
====

Public keys can be freely distributed. They can be saved on keyservers and linked to. They can be copied to removable media and handed to a person. They can be downloaded and emailed, but check the fingerprint to make sure an MITM has not intercepted and switched keys on you.


Encrypt Messages
----

Use the `--encrypt` flag to encrypt a message.

```
gpg --encrypt --sign --armor -r friend@email.com filename-to-encrypt
```

You will not be able to read the resulting message without "friend's" private key. To make a message you can decrypt also, add yourself as a recipient.

```
gpg --encrypt --sign --armor -r friend@email.com -r you@email.com filename-to-encrypt
```

Decrypt Message
----

The simplest command there is. You will, of course require the appropriate key.

```
gpg filename
```

As you see, you generally encrypt with the recipient's public key so that only that recipient can decipher the message with their own private key. So before you send someone a message, they will be required to get you their public key to use.

So, for encryption:
Recipient Public key encrypt. Recipient Private key decrypt.

However, for *signing*:
Your Private key encrypt. Your Pubic key decrypt.

With signing, _anybody_ who has access to your public key can decrypt the message, so it's not much in the way of security. So what is the point? The point of signing is integrity. Because only your public key can decipher the message, it is now definite the that message came from you and you only (unless your private key has been compromised!)

To sign a message:

```
gpg --sign myfile.ext
```


Other Tasks
====

List the keys you have:

```
gpg --list-keys
```

Update key information by fetching new info from key servers (Updates you would need to know if any of your friends have revoked their keys).

```
gpg --refresh-keys
```


What to Use GPG for
====

You use it for encrypting messages, of course! The problem is: To who?? The complexity of GPG excludes the majority of my friends and family, all of whom are non-technical. Even at work, if I started requiring my boss and co-workers, let alone our customers to jump through GPG hoops to read my messages, I would not have a job for long.

As much as we want to protect our privacy, I'm afraid we've lost the advantage of acceptance and convenience. If there are appropriate venues for posting or exchanging encrypted messages, they're not in easy and obvious places, beyond https and default phone encryption.

Luckily, I, and hopefully you, develop software. Several of the programs we develop communicate over a network or the internet. This might be a great place to use a little encryption. If it's not going to be openly pursued, we can at least build it into the things we make, and make encryption ubiquitous. The best encryption appears to be the type that the user doesn't have to think about.

So instead of using GPG for humans to communicate, wrap your API in GPG and let your programs communicate with a little privacy. No doubt there are more appropriate encryption schemes for this use case as well.

On the server, set up a decrypting listener:

```
ncat -lk 50000 | gpg
```

The `ncat` program is used on Red Hat (Fedora, etc.) systems, while on a Debian
(Ubuntu, etc.) system, you will use `nc`. Same thing, different names for some reason. I think even the options are the same.

This will listen for messages on port 50000, then decrypt anything that comes through, printing it to stdout.

On the client:

```
echo "Be sure to drink your Ovaltine!" | gpg --encrypt server@server.com | ncat server.com 50000
```

This will send a string to our server after encrypting it with the server's public key. Of course, the keys need to be set up on either machine beforehand.


My Public Key:
===

[My current key can be found here.](http://pgp.zdv.uni-mainz.de:11371/pks/lookup?op=index&search=Erick+Veil)

