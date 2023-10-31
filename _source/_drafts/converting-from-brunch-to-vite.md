---
layout: post
title: "Converting a JavaScript project from Brunch to Vite"
---

# Brunch deprecated and in archive mode

But I loved it. Because it worked and I used it in several places. Now I have to take that apart and fix it. Writing down what I did, so that I remember what to do when I next encounter an old project I try to tinker on and find that it is a brunch project.

# Choosing Bun, JK

Its the new hotness, might as well. I've done webpack things before and they seemed convoluted. I closed my eyes and used the stuff that appeared before me. Brunch made sense, the way it worked made sense, the way it was configured made sense, I was able to create a plugin for it even. Lets see if Bun makes such sense.

# Choosing Vite, actually

Could not find a gd Bun vanilla example. Don't want to buy into the ecosystem hard atm. Vite clean and I like it.

# Sidenote for tests

Used istanbul and mocha. Looks like Istanbul is renamed to NYC?? and Mocha might be replaced by Bun test if it works nice. I liked the code coverage stuff for Nordrassil. Vite required me to move to ESM, which is a good move anyway, and istanbul/nyc doesn't support that. Picked up c8 instead for that reason. Supports ESM and works like istanbul/nyc with 1/3rd the dependencies.

# Remove Brunch

`git rm brunch-config.coffee brunch-config.js`

Remove brunch and assorted libraries from dependencies then `npm prune` to clean up node_modules.
