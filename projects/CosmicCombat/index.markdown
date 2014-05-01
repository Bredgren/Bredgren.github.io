---
layout: default
title: Projects | Cosmic Combat
project_links: project_links.html
---

Cosmic Combat - {{site.data.projects.all-projects.CosmicCombat.year}}
=============
{% include {{page.project_links}} proj=site.data.projects.all-projects.CosmicCombat %}
This is a game that I've been slowly designing as time permitted since the summer of 2012. I finally had enough figured out and enough time to start implementing it in March of 2014. My intention is for this game to be an adventure game of sorts, with a fighting system inspired by Dragon Ball Z and a few other elements inspired by NetHack.

This project is extremely ambitious, but what makes it unique for me is that it is the only project I've ever started that I haven't given up on even after months of doing no work on it. I feel like it is within my abilities but only barely. I don't expect to be anywhere near completion until at least 2016, if I'm lucky.

The idea for this game came from trying to understand how the character's energy in Dragon Ball Z worked.

What I decided to end up using is a system that involves one bar for all purposes, as opposed to a game like Skyrim, for example, which has 3 bars (health, stamina, and mana). My energy bar serves all those functions as well as a couple others. It works like this: there is a maximum energy level, an "active" energy level, and an energy level I call "strength". They share the relationship 0 <= strength <= active <= maximum. The maximum can be thought of as your characters level. Performing attacks or getting hit decreases the active level, if it reaches 0 you die. Active slowly recovers, and while it increases, the maximum level also increases at a slower rate. This means you "level up" by spending and recovering energy. Strength contributes to how strong your attacks are and is also what others sense your energy to be (it is like the "power level" in Dragon Ball Z).

After coming up with this I decided I wanted to see what a game that used this would be like. I had recently played NetHack and liked the ways it incorporated randomness to each play-through, and so I tried to combine some of those ideas as well. The main way I've applied it is to the attacks.

Attacks are performed via combos, however the number of keys that the game can use to make a combo is about half the keyboard. The idea is for each attack to have a number of keys associated with it (more keys might mean the attack is better), but the specific keys used will be generated semi-randomly each play-through.

Right now it's really hard for me to say how these kind of mechanics will actually feel. At the very least I think they are interesting so I'm going to try to get them to work the best I can. Of course I would not be surprised to find out that it just isn't as fun as I hope it will be, in which case they will have to be changed ('tis the nature of these things).
