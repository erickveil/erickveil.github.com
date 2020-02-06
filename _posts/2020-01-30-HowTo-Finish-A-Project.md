---
layout: post
title: How To Finish Projects
date: 20-01-28 06:00:00 -0800
categories: Rogueworld, Rogelikes, Game Design, Unity
---

![Rogueworld](https://i.imgur.com/zNazoy2.jpg)

It's been a while. The thing about being creative is that you don't always have
time to get into the meta about writing about being creative. I'm hoping I can get back into it.

# What I'm Working On

**Rogueworld** (working title) is a roguelike game where you wander around a procedurally generated world and kill monsters.

Technically, it's been in development since I was 12. I've written designs for games like this, imagined features and how to implement them, and tinkered with the ideas since SSI released a game called [Questron](https://en.wikipedia.org/wiki/Questron_(video_game)) on the Commodore 64. At one point my C64 died, and I would not own another computer until Windows 98. But during that intermission, I still designed this game in BASIC - on paper and in my head.

# About Roguelikes

Back in those early days, we didn't call them *roguelikes*, as the only roguelike that existed was [Rogue](https://en.wikipedia.org/wiki/Rogue_%28video_game%29) itself. We just called them adventure games, or CRPGs. So I have a much stricter version of the term than even most modern roguelike enthusiasts, let alone The Marketing Department. If Steam is any indication, *roguelike* has become a meaningless buzzword that you attach to a video game to sell units. It's one of those words like *blockchain, The Cloud, VR, AR, AI, etc*. A word that has a real meaning, but The Marketing Department ignores it and assigns the definition: "Mysterious quality that you definitely want in our product."

For me, and my strict definition, a roguelike has ASCII graphics, or a low, actually 8 bit, 256 color, tile based graphic style. My Rogueworld obviously doesn't fit my own definition, but it does fit into some of the more common definitions that are used today by those who are not as narrow minded as me, yet still maintain a reasonable checklist of what is required for the label.

The roguelike elements of Rogueworld are:

- Classic roguelike keybindings, such as numpad movement.
- Bump into the enemy to attack them.
- Procedurally generated game board.
- An adventure RPG style gameplay.

# Rogueworld

One of the interesting new things I've brought into this game is the hexagon world map, instead of the traditional square grid. It's procedurally generated out of several overlapping Perlin noise maps to determine the terrain types. This isn't the first roguelike to use hexagons. There is very little that is original about my designs. I hopefully borrow the most interesting (to me) concepts from a wide enough range of other sources, and mash them together thoroughly enough together that the end result at least appears original. That's kind of how creativity works.

I did have plans to give each terrain type its own sprite, but once I got the world generated, altering the solid hex colors on a Perlin gradient height map, I really liked the minimalistic look. There's just enough information to convey the idea, and no more, and that is another key concept of roguelikes which is often overlooked. You can perceive the edges of each space, and that they are hex shaped, and the colors suggest terrain type fairly well. Perhaps in the future I'll add an interface to inspect them to confirm what the user might believe each color stands for.

The characters have been called "8 bit" in style, but they only look that way. They're actually designed as 16x16 pixel art with a much larger color palette, then enlarged to 72x72 pixels to add subtle details. The ones in the game now have been used as Roll20 tokens in my weekly D&D game, and they have made splendid stand-ins for game pieces until I settle on the art style.

# What Does It Mean To "Finish"

This incarnation of the game began as an abandoned attempt at a 4X game. The scope of that game was far too grand right out the gate. This is a way I tend to begin most of my personal projects. I have never finished a personal video game project, because my initial urge is to design a complex simulation that you can play inside of. After the 4X game gathered dust on the shelf, I sat down and wrote a list of types of games, and the minimum components that those games require. They wouldn't be groundbreaking, but the feature list was limited enough that there was a realistic chance that if I picked one up to start, it could be finished.

But what does "finished" mean? I struggle with this question whenever my boss at work asks me, "When will it be finished?" Is finished the Minimum Viable Product? If so, then my work project has been finished for over a year. It does its base job just fine. But there's other requirements. Our product has to compete with other like products out there, which do what my MVP does, but with more features. There's a lot of supporting development that has to happen around security, leaving a trail for post-mortems when the customer accuses the software of misbehaving (both rightly and mistakenly). There's the Requests for Proposals brought to me from Sales, with lists of features that a potential customer requires before they'd consider purchasing. There's bugs. No matter how many bugs you fix, there's always more bugs. It's never really "finished."

Thankfully, for my own projects, I can define the terms. The short list of requirements is all I have to fulfill to declare this project finished. Of course, there are gigatons of cool ideas that pop up along the way, and the goal is to mitigate them. Put them on a list to add *after* it's finished. Stay focused on the small goal. Add features later, if I'm still inclined.

That's how you finish. Ask any programmer on the internet: Keep the bar low. Lower than you think. Your eyes are bigger than your stomach.

# Most Recent Progress

This past week, I've nearly finished up the combat system. There's a transition when you bump your enemy on the world map, where you're sent to the combat map and it turns into a sort of simplified tactics game between you and the enemy. Or enemies - as the overworld avatars of your enemies actually represent a party of one or more bad guys you'll have to kill to win the combat.

I set up the input scheme to be easily added to a configuration interface if I want to - just in case a user other than me wants to change the command keys. The configuration is in a singleton that gets used by multiple interface scripts. There's one script for the overworld control, and one for the combat control. I have a manager script that controls which of the interface scripts is enabled at any time. That way when you move around on the combat screen, you aren't also moving you overworld persona around while its hidden. The config singleton allows all the input configuration to remain in one place, and still be used by the multiple scripts.

I would provide you a link to download the game in progress, but as of last night's commit, it doesn't compile. I don't usually commit in a state that won't compile, but I ran out of time. I have to set hard limits to the programming I do in my free time, otherwise it will drive away my loved ones, prevent my from maintaining my health, and completely consume my life. That's not hyperbole: For me it's a real possibility if I'm not careful.


# What's Next

I'm hoping to blog more about the incremental progress on this project. We'll see how my compulsion to work on it holds up, and if the closer finish line doesn't motivate me to reach it. I also have a small backlog of other miscellaneous "How To.." posts that exist as notes in a directory collected over the past couple of years that I hope to post for you, my imaginary audience (nobody actually reads this blog, I'm not *that* delusional). If by the next time I post about this game I have made it to a stopping point that compiles, I'll post a link, warts, crashes, *et al*.

Next on my list for this game is to finish up the combat system. I actually had a simpler version of it working earlier this week, but I've added an aspect system - skills and equipment - that affect all the combat stats. That should be next to be complete. After that, I'll add a wider variety of opponents, which should be simple work as the foundation will be fully laid out. Then the next step is the save/load game process. I'm going with JSON.net for that, as I enjoy working with it, I have bad luck with binary serialization, and I have no requirements to conceal the save file (though compression might be in its future, we'll see).

Will I finish this project? If not, I will at least append this post with an epilogue. But if work continues, you should see more posting.

Good luck.
