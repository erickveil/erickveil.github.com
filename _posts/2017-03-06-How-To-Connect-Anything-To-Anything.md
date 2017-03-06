---
layout: post
title: How To Connect Anything To Anything Without Learning a Million APIs
date: 17-03-06 08:20:00 -0800
categories: API, IFTT
---

> Laziness: The quality that makes you go to great effort to reduce overall energy expenditure. It makes you write labor-saving programs that other people will find useful and document what you wrote so you don't have to answer so many questions about it.
> 
> - Larry Wall, *The Three Great Virtues of a Programmer*

[IFTT](https://ifttt.com) has been a semi-useful service for trivial life-automation. The services I want are often half there or the things I want to do with them half aren't. Still, it's a neat toy. If I ever decide it's worth the money to invest in home automation, I'll definitely be looking to see if it's connected to IFTT. 

On the other hand, I could have more freedom and control over my personal automation if I just write my own scripts and tie them into the many [APIs](https://www.programmableweb.com/apis/directory) available on the internet. The problem is, when it comes to personal automation, I tend to not want to put a lot of effort into it. I don't want to learn your API. I don't want to set up [OAuth](https://oauth.net/) or whatever crap you're requiring me to deal with just to get information out of your service. I have a queue of interesting hacking projects so long that I'd need to hire a development team or two just to see my hobbies completely explored.

I mean, I'm lazy.

But there's a simple solution. IFTT has done the heavy lifting for me. Now I can write a script that, for example, scrapes an email from Amazon for the things I've impulse bought, pulling out just the data I want and enters it into an SQL database on my server so I can see exactly what a bad idea it is for me to have access to one-click ability to order books. And to get that Amazon email out of my Gmail, I can use IFTT to connect Gmail to my script.

I can do it on both ends: I can write a server script that awaits data for me to do something with, and a client script that sends the data. It can be completely mine and I don't have to hook into third party services at all. Sure, I can write a listener and socket client, but let's remember how lazy I am.

To open ourselves to the world of effortless connectivity, we first need an IFTT account. I recommend playing around with it a while to get familiar with how it works. It's fairly simple. Set up an "applet" that sends you an email every time the President tweets the word "sad" so you know how you're supposed to feel about his performance.

The next thing we do is connect the **Maker** service. You will see that with Maker, there is one trigger, and one action. This is your gateway to connecting everything. To trigger things with your scripts, you send an HTTP request to a provided address. To get data to your script, you use the action to send an HTTP request to your script for parsing.

## Receive a Request

On IFTT, go to *My Applets* at the top of your page and then select the *New Applet* button.

Select the *+this* part of the command, and search for *Maker*. There you will see the only trigger you have access to, the *Receive a web request* trigger. Reading the block of text in that trigger, we see that we will be provided with a URL once we get set up.

Click on the trigger and give the event a unique name. This is how IFTT will tell apart all of your triggers. You see, even though there is only the one trigger, this label ensures that you actually have infinite unique event triggers to hang your events on.

Now, back at the applet page, you click the *+that* button. Let's send ourselves an email for this test. Any time we trigger this event, an email will be sent to our Gmail with the details.

You will see in your test email the data you've configured for the request.

What can you do with this? You can install IFTT on your phone and create a panic button that emails a friend in the event that you get kidnapped. You can write a script that checks APIs not connected to IFTT and triggers services that are. You can write a script that follows more complicated tree of logic than a simple `if ... then ...` and trigger IFTT events. You can connect IFTT to your AI network and create the beginning of Skynet.

## Send a Request

For this example, get yourself a computer that you can open a port to the internet. Run a simple listener: `ncat -lk ${PORT}` on the command line, then test it from another computer to make sure you opened the firewall correctly. Send it a test message using something like `echo hello | ncat ${IP} ${PORT}`.

Once you've got the listener running, you can create a Maker web request. Get IFTT on your phone, because that will allow you to put a push-button widget on your home page. This makes a simple trigger where whenever you push the button, it does the thing you tell it to in IFTT.

Set the push-button as the trigger for a new applet, then set the Maker *Make a web request* as the event.

For the URL, enter `http://${IP}:${PORT}` with the IP and port data for the simple listener we just set up with ncat. You can leave the method as get (and even insert some get data in the URL if you want). Select the content type to "text/plain" (Take note of the types you can send for future, actual applications) and just stick "hello world" in the body.

Then press the "Create action" button, and you're good to go. Press the push-button widget on your phone and you should see the HTTP header and "hello world" message sent to the listener you set up.

Now you're prepared to send any of your programs commands. Your AI Skynet now has input from thousands of sources, and can extract data from your email, Twitter, Reddit, GitHub and many other places that it may be a bad idea to feed an AI its data from.


## Reality Check on Uses

You *could* even skip the IFTT services all together and chain together Maker triggers and actions, giving your own scripts a cheap way to talk to each other. But at that point, you might as well just use ncat. Event if you have to call out an exec, it's not like these public HTTP requests are very secure either. 

So this is mainly if you want your personal automation to tie into the specific services that IFTT connects to. While there are a lot of them, I still find it limiting. Then again, when I was a kid, I had hundreds of satellite TV channels to chose from, and still couldn't find anything to watch. So maybe it's just me.

### Sources:

[Every developer should write a personal automation API](https://dev.to/anotherdevblog/every-developer-should-write-a-personal-automation-api)
[IFTTT Adds a Channel for Makers](http://makezine.com/2015/06/26/ifttt-adds-new-channel-makers/)