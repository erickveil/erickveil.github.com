---
layout: post
title: How to Reconnect With Your Github Repositories
date: 22-07-26 06:00:00 -0800
categories: Github
---

# Why?

One annoying thing that happened in the past year is that Github moved from allowing me to just enter my password whenever I wanted to push to a repository. 

Now they want a special "key" to be used, which, honestly, is just a password. 

A bad password. 

It's random, and can't be memorized, so now Github users keep a plain text password in a file to use anytime they need to enter it via cut an paste. Whereas a password that can be held in the brain is more secure. 

Are user passwords often bad? Yes. But Github is not Facebook. Your tech-illiterate grandma isn't using it. Software developers are. Microsoft doesn't really need to hold our hand when we cross the street.

And, why? Is someone going to steal my Github password from my terminal somehow and make a commit for me? So what? By the way, that same password still gets them access to the website, where they can do just as much "damage". It's security theater.

The change makes sense if you work in information security: Your job is to make sure systems are secure, which is actually not a heavy workload once you've nailed everything down. It's one of those jobs where if you do it well, nobody in management hears from you, and so management begins to think you don't do any work, and so they get rid of you, and so the security risks return in your absense. 

Managers, you see, are the biggest obstacle to actual productivity. (Not all of them, of course. I've had some good ones.)

So to justify your continued employment, you have to invent problems to solve. This is why we periodically get "new" zero-day vulnerabilities with their own websites, logos, and catchy names while ignoring the thousands of other vulnerabilities that are still out there and have been for some time. The new marketing team is trying to drum up business so the security company doesn't go out of business. Managers eat that stuff up with a side of Blockchain and Enterprise Cloud as a Service just before they click the link on the phishing email.

So here we are. Github made an arbitrary change (that is actually *worse* than the original solution) and Bitbucket followed just because a manager didn't want to look like they didn't know what they were doing.

# How?

- If needed sign in at https://github.com On the right-hand side of the screen click on the Menu button and Select Settings
- From settings, on the left-hand side Select “Developer settings”
- Then Select “Personal access tokens”
- Generate a new token.
- Select all the Permissions available
- Once all the permissions are selected Click on Generate Token Button 
- Your personal access token will be displayed.
- This token can be used in place of a password. 
- Click on the blue copy icon (clipboard) to copy your token. 

Put it in a password manager that you'll have to open any time you want to use Github from now on.

Voila! You now have an extra step to perform every time you want to just push something. Isn't that great?

