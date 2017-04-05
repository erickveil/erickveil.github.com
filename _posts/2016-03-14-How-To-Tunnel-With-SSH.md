---
layout: post
title: How To Tunnel With SSH
date: 16-03-14 11:28:42 -0800
categories: 
---

SSH Tunneling
====
The poor man's VPN. SSH can be used for much more than remote access to a computer shell.

Test Your Sockets
====
Before we get into unexplored territory, we want to eliminate any possible unknowns from our experiment. I can't tell you how many times I've tried some new socket-based experiment that kept me up late, when the whole problem was the firewall or something silly like that. So let's make sure our machines can talk to each other in a way that we already know they should.

We will port forward localhost:5000 to server.com:6000, where localhost is our client.

First, test that the client and server can communicate on the ports we will use. I like to ssh into the server to do tests like this. I will have one terminal shelled into the server, and the other on my local shell. That way, I can control both on the same screen.

On server.com:

```
ncat -l 6000
```

Then on localhost:

```
echo hello | ncat server.com 6000
```

The string `hello` should print on the server terminal, and ncat will exit and return control to the user.
Now we test the local port:

On localhost:

```
ncat -l 5000
```

Then on server.com:

```
echo hello | ncat client.com 5000
```

We should get "hello" printed on the client machine. So now we have proven to ourselves that these two machines can communicate with each other on the ports that we will use.


Port Forwarding to a Different Machine: Option -L
====

Now we set up the tunnel. This will work like when we set port-forwarding on our router. Say my internal computer has a server listening on port 6000, but when I connect to my network from the internet, my router takes anything that is sent to port 5000 and routes it to port 6000 on the internal computer. 

That is what we are doing here: The client at localhost is synonymous with the "router" and the server is synonymous with the internal computer. We will set it so that anything sent to client, port 5000 gets routed to server, port 6000.

From the client:

```
ssh -f user@server.com -L 5000:server.com:6000 -N
```

Now, to test that connection.

On the server again:

```
ncat -l 6000
```

Then back on the client:

```
echo hello | ncat 127.0.0.1 5000
```

You would see, even though we sent our string to localhost, it was received on the server end of things.

Three-Way
===

You can also forward a port using an intermediate computer.
Anyone watching the local network will see you connecting to the intermediate computer and not the server.
To see a connection from the intermediate to the end server, someone would have to look at the network that the intermediate is on.

So, if the local router is configured to block connections to server, you could get around it this way. That router wouldn't know about the connection to server to block it, as it would only see the connection to intermediate.

From the client:

```
ssh -f -L 5000:server.com:6000 user@intermediate.com -L
```

Now you can run the same ncat test as before. Listen on the server, and echo to localhost, and the string will make it to the server. No need to touch the intermediate at all, as long as it is running sshd.


Routing Browser Traffic With Socket Secure (SOCKS): Option -D
===
This tunnel trick is good to help you if you intend to browse on a shady internet connection, like a hotel or coffee shop. I don't trust free internet.

First, check your actual internet IP at whatismyip.com and note it for later.

On the client:

```
ssh -D 5000 user@server.com
```

TODO: Try this with -f and -N and observe the difference.

In the browser: 
- Find the connection settings. 
- Set to "Manual proxy configuration." 
- Set the SOCKS Host to 127.0.0.1 and the port (5000 in this case).
- Set the SOCKS connection to SOCKS v5.
- Optionally check "Remote DNS" to tunnel all of the DNS lookups as well.
- Click OK and save the changes.

Finally, visit whatismyip.com again, and note that the IP has changed to that of the proxy we set up.

More info on SOCKS: https://en.wikipedia.org/wiki/SOCKS

Reverse Tunneling: Option -R
====

The scenario: You are appropriately paranoid and you've set your personal network firewall to block all incoming traffic, such as SSH. But you want to make a special case for yourself so you can shell into your own network while you relax on the beach with your portable wireless wifi. The reverse tunnel gives you that special exception.

First, I need to clarify a couple definitions. Since with a reverse tunnel, some of the roles between server and client get switched around, I want to be clear what machine does what and who's address goes where. 

- Destination: The computer that you want to access, which is behind a firewall with no accessible ports.
- Source: The computer you are using to access the destination. It must also be running an SSH server.

Things get confusing, because during setup, the Source is the server, and the Destination is the client, which is reverse of what you are used to.

The process requires some initial access to the Destination. You have to set up access from there in order for this to work. Reverse tunneling will not allow an outsider to break past your firewall, unless he was able to access the Destination computer at some point in the past.

Unlike the other ssh commands, in which you run the ssh command on the Source, this command is executed on the Destination. 

```
ssh -f -N -T -R 60000:localhost:22 sourceuser@source.public.ip
```

NOTE: I've tried this while tunneling a non-ssh port and found that using "localhost" doesn't always work for that. I have had success using the internal network IP of the destination machine instead.

If you are like me, and you have your ssh port forwarded on your router, or your ssh server on your Source machine is different from 22, then you will have to add the `-p` option, with the port number that your Source computer's ssh server listens on.

```
ssh -p 70000 -f -N -T -R 60000:localhost:22 sourceuser@source.public.ip
```

This command used the Destination's ssh client to connect to the Source's server, which is backwards from what you are used to when using ssh. For this reason, you will be required to enter the Source User's ssh password when you run this command. If you have key only access to your Source computer, then the Destination will be required to have the public key that allows it to access the Source's ssh server.

Once this command is run, and your Source password is entered, then anything on Source localhost, port 60000 will forward to Destination's local port 22.

From the Source computer:

```
ssh -p 60000 destination_user@localhost
```

At this point, you will have to enter the Destination user's password to proceed (or have the public key for the Destination).

You can access Destination from a third computer also, as long as it can ssh into the Source computer at port 60000.

### Edit: SSH Config Changes

I came back here today to duplicate these procedures and create a tunnel, and I can't believe I left out some critical configuration changes. Without them, you are likely to experience frequent disconnections of an idle tunnel, especially if, like me, you are working on Linode.

You will want to make these changes on both your internal and external machines. To edit your config for the SSH server, open `/etc/ssh/sshd_config` as root. Find the following lines, uncomment them if needed, and change their values:

    TCPKeepalive yes
    ClientAliveInterval 30
    ClientAliveCountMax 99999

That should help keep the pipe open.

