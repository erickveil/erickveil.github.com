---
layout: post
title: How To Secure Your SSH Login With Keys
date: 16-03-06 14:47:29 -0800
categories: 
---
# How To Secure Your SSH Login With Keys
Using RSA keys is a more secure way to protect a server that you remotely
access via SSH.
Visual Host Keys
---
You can set up your ssh to draw a "randomart image" every time you ssh into a
computer.
The randomart is unique for each computer you log in to.
This ascii art performs the function of a hash: because it is unique, and
because it is immediately more 
recognizable to a human than a string of characters would be, a user can
immediately tell that the computer 
that is being logged on to is the correct computer.
We humans recognize images-- even random ascii images-- far more easily than we
remember character strings.
 
 You can add the option to your log in:
 `ssh -o VisualHostKey=yes eveil@192.168.58.197`

 Or, even better, you can add it to your `.ssh/config` file so that you see the
 randomart every time you log in.
 If `~/.ssh/config` does not exist on your system, you can add it with vim (or
 your favorite text editor).

 The line that needs to be in the file is:
 `VisualHostKey=yes`

 Some example randomart:
 ```
 Host key fingerprint is SHA256:ebUiTwA1htZ0Rmj4bE5nJLaINuv+zyWe8oj2dLVcCNY
 +---[ECDSA 256]---+
 |      .*=o+      |
 |      +o*=.      |
 |     o *++E .    |
 |    + ..*+oo .   |
 |   . o +So* o    |
 |    .   .B +     |
 |   .  . o =      |
 |    oo.= +       |
 |   ooo++*        |
 +----[SHA256]-----+
 ```
 ```
 ECDSA key fingerprint is SHA256:enY+IENW64ubZW7EzQWIYpNNY1xmiFM/D5Dip/t3Y1w.
 +---[ECDSA 256]---+
 |     B*=+.       |
 |    O.**o .      |
 |   o = .+. .     |
 |    . + .+  .    |
 |     = oSo..     |
 |    . o.= o E    |
 |     ..==+..     |
 |    . .B+o*      |
 |     .++.o.o     |
 +----[SHA256]-----+
 ```

 Generating RSA Keys
 ---
 Keys are more secure than passwords. They are longer, more random, impossible
 to guess, and significantly more 
 difficult to brute force.

 On the client:
 `ssh-keygen -t rsa -b 4096`

 RSA is the
 [cryptosystem](https://en.wikipedia.org/wiki/RSA_%28cryptosystem%29).


 4096 tells ssh-keygen that you want to make a 4096 bit key, which is even
 harder to crack than the default 
 2048.

 Answer the questions. The default answers are in parentheses, you can just
 press `ENTER` to accept the default. 
 I recommend you choose a good password for your keys.

 Next copy the key to the server:
 `ssh-copy-id <username>@<host>`

 Or, if you need to specify a port, you need to add quotes, due to a bug:
 `ssh-copy-id "<username>@<host> -p <port>"`
 Another 
 way is to use `scp` to copy the key over (`id_rsa.pub` is the key that we are
 sending the server) and 
 cat it onto the `~/.ssh/authorized_keys` after backing up:
 `cat id_rsa.pub >> authorized_keys`

 Try using ssh to log on and make sure it worked.
 You will need to use the key's password now, instead of the user login
 password.
 If you know the user password, but forget (gasp!) your key's password, you can
 still just press enter for the 
 user password prompt to get in instead.
 Unless, that is, you've disabled password logins. Which is what you would do.
 Because that's the whole point.

 Once you've created your keys on your client, you can send the public key to
 any machine you log in on for easy 
 key logins.

 Finally, to turn off password authenification, sudo or root edit
 `/etc/ssh/sshd_config`

 And change the `PasswordAuthentication yes` to `PasswordAuthentication no`

 Restart sshd to make your config changes take effect.
 `sudo systemctl restart sshd.service`

 And that's it. You now have a secure server for shelling in to with keys only.
 Managing Multiple Keys
 ----
 Some keys I use just so I can be lazy, and not have to enter a password. Like
 pushing to GitHub, or on the local network at work, where I shell around on
 test machines all day. But I also want a key with a strong password, which
 protects access to my home server. Having a strong password on that key means
 that if my mobile tablet is compromised, the rest of my personal network is
 not.

 The solution is to create a second key. During the question/answer part of
 creation, give the new key a different path and/or name.

 There are two ways you can specify which key you want to use. One is on the
 command line as you shell in:
 ```
 ssh -i <key location> user@server.com
 ```

 The second, and better way is to add your frequently targeted machines to your
 local client's config file in `~/.ssh/config`. This also helps you keep
 straight which key is for which machine, so you don't have to think about it:
 In the config file, enter:
 ```
 Host           friendly-name
 HostName       long.and.cumbersome.server.name
 IdentityFile   ~/.ssh/private_ssh_file
 User           username-on-remote-machine
 ```

 Now you can access that machine with the simple alias that you set up:
 ```
 ssh friendly-name
 ```

 You can have as many machines in your config as you want!

