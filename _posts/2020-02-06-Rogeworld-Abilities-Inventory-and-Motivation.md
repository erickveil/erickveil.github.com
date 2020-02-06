---
layout: post
title: Rogueworld: Abilities, Inventory, and Motivation
date: 20-02-05 06:00:00 -0800
categories: SSH, SSHFS, SFTP, Linux, Cloud
---

![Rogueworld Camping](https://i.imgur.com/IohIny4.jpg)

Oh good! I've come back to post after only a week. I almost forgot, but as I opened Unity to get productive, I remembered that I wanted to post something, and so all productivity is lost due to writing reports about productivity. This is getting to be like how actual work plays out!

So the intense, obsessive motivation is definitely on the wane, as I predicted it would be. But I don't think it's completely gone. There's just enough left for me to continue work without skipping meals, self care, and the attention my family needs. Which is perfect, even though it is slower.

## Combat

Before the steam ran out, I managed to finish combat. Now bumping on the world map switches you to a tactics map. There, you progress in an "I Go, You Go" turn taking manner until one side is defeated. The combat routine itself began life as three class methods: One for the player activated by input, one for the mobs activated by their bumping into a PC, and one activated by the mobs' AI under certain conditions.

Then the combat code was consolidated into a single static method. Rather than create a whole new class for this method, I just picked the player movement class and attached it there. As development went on over the course of days, I found that the static method was requiring a large number of arguments to feed into combat. We need the attacker's stats, the defender's stats, the inventory of both parties (weapons and armor data), and the attributes of each parties (melee skills and such). With the large argument list, I refactored the method to require a single argument that is a data object containing all of those as attributes.

After a night's rest, I realized that this new data class, which was an agnostic middle ground between player and mob, would be the ideal holder for my static combat resolution method. And once *that* was in place, I realized that the method no longer needed to be static. We now have a class dedicated to handling combat and all of its data. Elegant, yet organic problem resolutions such as this will be forever fascinating to me. It's like solving a math problem without knowing the formula, but stumbling to the correct answer anyway because you followed some basic order of operations and observed the arithmetic properties you learned about in grade school. In this case, I stumbled on the **Extract Class** Refactor [(Fowler, 1999)](https://martinfowler.com/books/refactoring.html).

## Managing the Feature List

At one point, a friend asked for guidance on deploying a project from Unity. Before sending instructions, I thought I should walk through the process myself. In doing so, I discovered that this game lacks any feedback as to what's going on at all. When you bump into the impassible mountains or sea, your passage is blocked, and nothing happens. Did the game not register the keypress? Same when in combat. You bump the other party, but nothing happens. This is because for me as the developer, I have tons of feedback! I have the debug log open while playing which feeds me all sorts of information about what's going on. The compiled game doesn't have that.

So I created the "nudge" movement. When you try to move onto a space and the game does *something else* -- either block passage or attack an opponent, your character nudges a little way in that direction before returning to the space they started in. It's just enough to tell you, "Yes, I got the instruction." There needs to be a lot more of this sort of thing before this game is useable.

I've [told you](https://erickveil.github.io/ssh,/sshfs,/sftp,/linux,/cloud/2020/01/28/HowTo-Finish-A-Project.html) that I want this game to get finished, and my strategy is to have a short, finite feature list. But elements such as this, which come up during the course of development, are unavoidable. You discover things during the course of development that absolutely must be included for this to be a workable game.

The feature list must be slightly malleable, but somehow rules must be made as to how it can change. After adding the "feedback" feature requirements, I gave the list another hard look to see if it's really the list I needed. Yes, the feedback needed to be there. What about the other stuff?

I looked at "Saving the Game". There's to be a whole lot of work in gathering data for serialization and deserialization. While there are packages out there that make it easy, there is no magic package that fits every possible use. They all center around native data types, and in Unity we have a lot of non-natives that are inaccessible. Some manipulation of the data is required in every instance. And then there's the user interface required for it.

But looking at the game, there's no real progress in this minimum, viable, completed game. You just go out and kill monsters. There's no leveling up. No changing the map. No clearing the board since the mobs always respawn. There's no point is saving the game!

Now, I would love to insert mechanics that rely on progress. But first I must finish the game. That's a feature for the list that comes after. Only when we are making progress playing the game that demands we have something to save to we need to include a save function.

## No Download Yet

I did promise that I would try to get you a downloadable version of the game, but it's not going to happen this week. I just ran a build, and now I've discovered that in a build the game doesn't start properly, even though it starts properly in the editor. This will have to be sorted out before I can get you something that at least you can move around on and explore.

For now, you will have to content yourself with reading.
