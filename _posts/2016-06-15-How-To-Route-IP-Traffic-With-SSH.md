---
layout: post
title: How To Route IP Traffic With SSH
date: 16-06-15 08:20:38 -0800
categories: 
---

Real world scenario that I was in: A new guy had taken over the IT department at work, and the previous guy resigned suddenly without providing any training. The guy who left had a [truck factor](https://en.wikipedia.org/wiki/Bus_factor) of one when it came to the management of our internal network. The new guy will get it eventually, but in the meantime I have this customer.

One of our products is a network enabled, embedded system. We have an API that a client must use to communicate with the device, and this new customer wants to test the API on a device I have set up in our lab. The customer is far away, and needs to reach the device over the internet. Our new IT guy can not get our network to cooperate on the simple task of forwarding a port to this device, and our customer's deadline is looming.

No problem. This looks like a job for SSH tunneling. However, these embedded devices do not run an ssh server. I'm going to have to use an intermediate computer to route traffic to the device and reverse tunnel out of our network.

![Connection Diagram](http://i.imgur.com/XrB6piq.png)

The client will connect to an external Linode computer's port. That computer will use a reverse tunnel connection to reach a computer I have set up inside our network. That internal computer, the Tunnel Router, will be running the SSH server that will maintain the reverse tunnel to the external Linode computer, and forward ports to the embedded Target Device.

I need to set up a [Linode](http://linode.com "Linode") computer first. This will allow me to get the customer a nice external IP to point their client to. Now I need to set up some ports for them to connect.

On the Tunnel Router, I forward ports to the Target Device. Sending API commands to this **Forwarded Port** will be just like sending the commands directly to the Target Device's **Listening Port**.

On the Tunnel Router, I run:

    ssh -f -g -L <Forwarded Port>:<Target Device IP>:<Listening Port> <user name>@<Tunnel Router IP> -N

	# example:
    ssh -f -g -L 5053:192.168.60.101:502 user@192.168.60.100 -N

- `-f` Puts ssh in the background when running. We don't need a shell to work with.
- `-g` Allows a remote host to connect, which we need since we will be connecting from outside the network.
- `-L` Performs the actual port forwarding. Connecting to the Tunnel Router's Forwarded Port will route to the Target Device's Listening Port.
- `-N` Tells SSH not to execute a command. Typically used when just forwarding ports.

At this point, we have a setup where our customer can connect to the Target Device indirectly, but they would have to be on our internal network to do so. We need to get past our own firewall!

To do this, we set up a reverse tunnel between the Tunnel Router's **Forwarded Port** and the Linode's **Client Port**. I like to keep the Client Port and the Forwarded Port the same value, just to make my life easier. The port number can be the same for both, because even though it's the same port, it's on two different computers.

Set up a reverse tunnel on the Tunnel Router:

    ssh -f -g -N -T -R "[::]:<Client Port>:localhost:<Forwarded Port>" <Linode User>@<Linode IP Address>

	# example:
    ssh -f -g -N -T -R "[::]:5053:localhost:5053" user@54.33.11.28

Some of the options are the same as for the port forwarding.

- `-T` Prevents termination of the tunnel if a terminal is requested. This could probably be left out of this command, since we will not be making a terminal connection.
- `-R` Defines this as a reverse tunnel.

Notice the `[::]:` before we define the port forwarding. This allows a computer outside the network to connect.

## SSHd Config Settings

We will need to set some of the settings in sshd_config to keep the connection from dieing:

    GatewayPorts yes
    TCPKeepAlive yes
    ClientAliveInterval 30
    ClientAliveCountMax 99999

## Firewalld settings
We will also need to open the ports we use in our firewall daemon. I use Fedora, and I find firewalld to be the easiest command line firewall manager I've ever used. These steps need to be performed for both the Linode and the Tunnel Router. My embedded devices don't run a firewall.

First get active zones and apply command to that zone:

    firewall-cmd --get-active-zones

On Remote open port 5053:

    firewall-cmd --zone=FedoraServer --add-port=5053/tcp --permanent

Reload firewall:

    firewall-cmd --reload

