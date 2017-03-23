---
layout: post
title: How To Host Your Own Cloud Storage With SSH
date: 17-03-20 08:20:00 -0800
categories: SSH, SSHFS, SFTP, Linux, Cloud
---

The Cloud. When somebody first explained The Cloud to me, I said, "Isn't that just another word for The Internet?" The joke is that The Cloud is just someone else's computer. You are keeping your important files on someone else's computer for the convenience of universal access. With Dropbox, I've got my files on all of my computers and my phone, fully synced. But that means that other people also have access to my files. A company can promise privacy all day long. That doesn't stop the janitor at the server farm from snooping through your stuff when nobody's looking. It doesn't stop the company from being sold and your stuff being shared before you've had a chance to notice. When you're not in control of your files, you have no idea how they're managing their security. 

One solution is to do it yourself. The last time I set up an OwnCloud server, it took me about a week of looking up stuff, making sure the right dependencies were installed, making sure permissions were set, files were in the right place, learning how the settings worked. I controlled the server, but I didn't feel like I had total control over the software. Then my old, spare laptop that was running the server died. I didn't lose any files, due to the distributed nature of cloud storage, but when I replaced the machine and attempted to set up OwnCloud a second time, I gave up. It's too much work.

There is a simpler solution for lazy people like me: Just mount a remote Linux file system over SSH.

## The Basic Command: SSHFS

Let's get right to the point with some commands. Once installed, you can look up the `man` file and get more information at your convenience.

The basic remote mount command is sshfs. You will likely need to install it via your package manager if you haven't already. It basically uses SFTP over SSH to control file access over the network.

    sudo sshfs <Remote User Name>@<Remote IP>:/<Path to Directory> <Path to Local Mount Point>

If you are mounting on an Ubuntu system, you should add the option `-o allow_other` to the command:

    sudo sshfs -o allow_other <Remote User Name>@<Remote IP>:/<Path to Directory> <Path to Local Mount Point>

If your remote SSH connection requires a key file like it should, then add the `IdentityFile` option:

    sudo sshfs -o IdentityFile=<Path to public key file> <Remote User Name>@<Remote IP>:/<Path to Directory> <Path to local Mount point>

For example:

    sudo mkdir /mnt/mountpoint
    sudo sshfs user@example.com:/home/user/ /mnt/mountpoint
    cd /mnt/mountpoint
    ls

## Making the Mount Permanent

To permanently mount the remote file system, you will first need to set up [key access for SSH.](https://erickveil.github.io/2016/03/06/How-To-Secure-Your-SSH-Login-With-Keys.html) Your key should not require a password for this setup. It takes away that second factor of security, but it allows you to have the file system mounted automatically. There is always a balance between security and convenience. In this case, the risk is that if your public key falls into the wrong hands (someone steals your laptop where you auto mount a remote home file system, for example) then they now have remote access to your remote machine via ssh.

Next, we will need to edit the fstab:

    sudo vim /etc/fstab

Add to the bottom of the file:

    sshfs#<Remote User Name>@<Remote IP>:<Path to Directory> <Path to local mount point> fuse.sshfs IdentityFile=<Path to public key> defaults 0 0

Then apply the changes to the fstab:

    mount -a

To remove the file system, it's just like any other file system. Use `umount`.

## Android Access

Now, not every machine I use is Linux. In fact, my gaming rig and Surface Tablet are Windows, and my phone is Android, and I want access to my files when I'm away from home. Otherwise, what's the point?

At this time, you will need to have a rooted Android phone in order to mount. I've found two options on Google Play. One is [free](https://play.google.com/store/search?q=sshfs&c=apps), and the other is [paid](https://play.google.com/store/apps/details?id=com.chaos9k.sshfsandroid). I cannot provide input on which is better, as they both require root. (I've been putting off rooting my phone unless I have a really good reason to put the effort into it).

In the past, I've used [Astro](https://play.google.com/store/apps/details?id=com.metago.astro) as my go-to file browser that allows me to access my SMB network hard drive when I'm at home on my network. It does have an option for a SFTP connection, but it doesn't appear to have a place to enter a key file, or even a password which may be why it crashes if I try to use it to connect to a standard SSH server. For now, the separate SFTP app will have to be my way of connecting.

### The Best Way So Far

However, there is a way to do this without root. Get an [SFTP client](https://play.google.com/store/apps/details?id=lysesoft.andftp). Since we're using SSH, you can plug in your credentials and key file and access the remote system just as easily.

## Windows Access

As anyone who's done SSH over Windows can tell you, there's going to be some GUI based programs you're going to need to manage your connections. I hear tales that Windows 10 has some sort of SSH support, but I haven't tested it. I'm reluctant to install actual spyware as an operating system.

### Win SSHFS

[Win-sshfs](https://code.google.com/archive/p/win-sshfs/) is going to allow you to have a mounted directory that can be set to auto-mount when your machine starts.

- Download and run [dokany](https://github.com/dokan-dev/dokany/releases)
- Download and run [win-sshfs](https://github.com/Foreveryone-cz/win-sshfs/releases)

Dokany provides a file system driver for non FAT or NTFS file systems. Linux (your cloud server) systems these days can be on something like Ext2fs. If Windows sees that file system, it will just want to format it, since it wants to format any disk it does not recognize.

When you install win-sshfs, the installer just blinks by without any apparent interaction. It's alarming, but when it's done after 500 ms, there should be an icon to launch the program on your desktop. Running and connecting is pretty self-explanatory.

### WinSCP

[WinSCP](https://winscp.net/eng/index.php) is pretty much standard issue along with Putty and other crutches we have to give Windows in order to make it functional.

The up side is that you may already be familiar with it.

The down sides is that you have to work through the application. No auto-mount on startup. No simple, mounted directory to drag and drop.

## Sharing Files

One of the nice features of cloud storage services like Dropbox is the ability to generate a link and give random people limited access to your files. You could give someone your public key, but then they have unlimited access to your entire cloud server! There may be some workarounds you could use.

You can create a temporary SSH key. Perhaps create a sharing user account with limited access. Place any files to be shared in that account's accessible directories and then provide the end user with a temporary key and an `scp` command they can paste into their terminal.

The drawbacks being that it doesn't scale well to multiple users if you do this a lot. Those users must grasp SSH keys and how they work. They must understand SCP, and they would require access to a machine that could run it. You wouldn't be able to use this method with the technologically incompetent. I've met people who could barely manage Dropbox, but could still manage to download a Dropbox file if given the link. Such people would not be able to get a file from you using this method.

Another option is to run a web server of your choice on the server machine. Then:

1. Create a random directory somewhere in the web directory using `mktemp -d`.
2. Copy your file to be shared in there.
3. Share the link.

This way is easy for users, since they just get the file by clicking the link. It also prevents them from snooping out other files you didn't give them. The random directory name helps to keep them from guessing links.

It's still a little less usable for any of my non-technical family members to use if I give them a "home cloud server account". One way to overcome this is to give them a "shared" folder on their user account. They can just copy any file they want to share in there. A cron job will periodically (once a minute) run a script that finds any new files and copies them to their random web directory, then emails the user a link. It would be worth writing just for the sake of my own laziness, even if there is no one else using the server.

There's also the situation with HTTP vs HTTPS and certificate warnings that scare people. I have yet to figure out how to get a free certificate for a home server. A warning page from someone clicking the link who doesn't grasp SSL can be off-putting, while an unencrypted link would be dangerous. You might decide to provide links to either, depending on the need.

There's no simpler way that I see to pull this feature off. I suppose we will file this under "security vs. convenience".

## Backups

The file system is just mounted. If the server's hard drive dies, those files are gone. It's not distributed like an actual cloud server would be. I highly recommend periodic, automatic, off-site backup of your files. [There are a lot of solutions for this](http://www.techrepublic.com/blog/10-things/10-outstanding-linux-backup-utilities/). 

This puts us back in the position of hosting our files on someone else's computer, unless you happen to have two homes. However, you can at least enjoy the greater choice of who you trust with your files. Also, you can write a cron job that compresses the files in an encrypted tarball before backing them up. Then you can ship the backups to any remote cloud storage service you want and not worry about it. Eat your cake and have it too.

## Sources

[https://github.com/libfuse/sshfs](https://github.com/libfuse/sshfs)

[http://www.tecmint.com/sshfs-mount-remote-linux-filesystem-directory-using-ssh/?_utm_source=1-2-2](http://www.tecmint.com/sshfs-mount-remote-linux-filesystem-directory-using-ssh/?_utm_source=1-2-2)

[https://linhost.info/2012/09/sshfs-in-windows/](https://linhost.info/2012/09/sshfs-in-windows/)

[https://code.google.com/archive/p/win-sshfs/](https://code.google.com/archive/p/win-sshfs/)

[https://igikorn.com/sshfs-windows-10/](https://igikorn.com/sshfs-windows-10/)

[https://github.com/dokan-dev/dokany](https://github.com/dokan-dev/dokany)

[https://github.com/Foreveryone-cz/win-sshfs](https://github.com/Foreveryone-cz/win-sshfs)

[https://winscp.net/eng/index.php](https://winscp.net/eng/index.php)

[https://linux.die.net/man/1/mktemp](https://linux.die.net/man/1/mktemp)

## Addendum: Not a Great Cloud Replacement

After trying this for a few days, it seems like an impractical replacement for a cloud storage service. Every action in the mounted directory lags, as you are now transferring files over the Internet.

The android SSHFS requires root. SFTP requires manually moving files and keeping track of which is newer. The Astro file browser does not appear to support key files. It will mount the directory, but you will need to allow password access (weak). Also, the lag issue.

Something based on rsync might be better.

A better use for SSHFS would be for mounting on a fixed computer over a local network, in a situation where you find yourself very frequently moving files between two computers, and the remote one is Linux.



