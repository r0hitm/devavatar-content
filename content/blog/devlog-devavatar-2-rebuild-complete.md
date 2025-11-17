---
title: "Devlog: Devavatar 2.0 - Rebuild complete"
description: "A rundown of the complete rebuild of devavatar, including package upgrades, a new color scheme, UI changes, and more."
tags:
    - devlog
pubDatetime: 2025-06-28T22:39:30.000Z
---

Finally completed the rebuild of devavatar from scratch. The major reason for doing a complete rebuild was 1) to upgrade all packages, 2) to move to using pnpm, and 3) to make some fundamental changes to the internal structure of the template I originally started with (namely, Astro-paper). In the rebuild, I adapted the things from the template that I like, and I did some of my own changes. Here's a crude rundown:

1.  Updated packages and using pnpm now
2.  New color schemes in both light and dark modes, using OKLCH
3.  New header
4.  New footer
5.  Removed pagination from all posts page
6.  New 'project' page - still WIP
7.  New search component that does not require a separate page (tip: press `ctrl + k`)
8.  Replaced most of the icons with lucide, (except the footer social icons - I kept the ones from the previous version)
9.  Minor layout changes on the posts
10. Added mdx support + setup preprocessing to add nofollow noreferrer to all links that direct outside of devavatar
11. Using tailwind v4 now
12. New Open-graph images
13. Scroll to top floating button (See your bottom-right edge).

The old repo for devavatar has been archived now. There are a couple more things that I want to add so keep an eye out for that!
All in all, I'm pretty happy with how this turned out to be!
