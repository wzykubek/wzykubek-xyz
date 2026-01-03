---
title: DuckDuckGo bangs without DuckDuckGo
description: This article is about the greatest feature of DuckDuckGo and how to replicate it in any other search engine.
keywords:
  - search-engine
  - bangs
  - DuckDuckGo
tags:
  - self-hosted
  - dev
date: 2026-01-03T20:16:25
modified: 2026-01-03T20:16:25
layout: article
draft: false
---
When you come to the privacy world, then you would probably consider changing Google from being your main search engine. The reasons for that are beyond the scope of this post. Probably the most popular option out there is DuckDuckGo. 

I'm not sure if that's the best option; there are other engines that are definitely worth checking out. However, DuckDuckGo offers some great UX features except for just giving search results. You can, for example, use Vim bindings to jump between entries on desktop; you can customize the theme, use different version of the engine if you don't like some features, e.g., `noai.duckduckgo.com` to disable AI, `lite.duckduckgo.com` for no-JS version, and probably some others I don't even know. 

But my favorite feature is one called [bangs](https://duckduckgo.com/bangs). Predefined shortcuts for other search engines and on-site searches. You can, for example, use `!w linux` to search directly on Wikipedia; `!aw podman` to search on the Arch Wiki and thousands of other sites. There are some of my favorites:
- `!diki` - simple English/Polish dictionary
- `!oald` - more advanced dictionary
- `!ud` - slang dictionary

However, *bangs* have some drawbacks. Firstly, they're a little bit slow, but what's more important, they're vendor-locked to DDG. Not everyone wants to use *duck*, even if the feature is good. There are some solutions out there. 

The first project I've discovered was [oglofus/bangs](https://github.com/oglofus/bangs). This works pretty well, but there were some issues. Because this project is written in Go, I prepared some fixes and improvements to it, and all of them were patched upstream. However, I discovered that the project performs fancy hash-based lookup on a pre-compiled binary file. The author admits that it enables faster queries. I've started thinking if that's really the case. Ok, the raw `bangs.json` file, which contains all the *bangs* that was ripped from DDG itself, has more than 50k lines. But is that binary search strategy actually needed? I decided to test it by myself. I've started a new Go project, copied some stuff from the repo I contributed to (basically just my own code), and replaced the whole logic of querying with lookups through key-value map. 

When I created the working prototype, I ran two applications side by side and benchmarked them using `hyperfine` and results on both implementations took exactly the same time. That being said, I've decided to run my own project called [Banger](https://codeberg.org/wzykubek/banger) instead of developing upstream. I host a [public instance](https://banger.brono.cloud), so you can test it out without installing it on your own server. You can specify your own *fallback* search engine via query parameters and set it as your default search engine in a browser. Any combination is possible, e.g., Banger and Google, Banger and Bing, Banger and Startpage, or just Banger and DuckDuckGo, because why not?

To be honest, there is also another project - [Unduck](https://unduck.link/) - which tries to solve the same issue. It aims to perform all redirects client-side via JavaScript. However, there is some issue with this approach - you can't use it outside a browser. I've observed that *in my case* this isn't any faster than [Banger](https://codeberg.org/wzykubek/banger) running on VPS, but maybe someone will find it useful. It is also a lot more popular.