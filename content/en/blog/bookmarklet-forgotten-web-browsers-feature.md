+++
title = "Bookmarklet - Forgotten Web Browsers' Feature"
date = 2025-12-06T00:10:26+01:00
draft = false
layout = 'article'
description = 'This article is about bookmarklets, short JavaScript snippets you can run quickly as browser bookmarks.'
+++

Modern web is a horrible place. "Surfing the web" is more like swimming in the pool full of plastic garbage nowadays. Greater part of new websites is created using many unnecessary JavaScript frameworks, libraries, and other dependencies. I don't say that is totally bad. There are many cases when they're useful, but even simple article only websites are fulfilled with unnecessary scripts without any particular reason.

There are also web extensions. They provide many additional features, but they are slowing down the browser and have access to many private data you probably don't want to share. Sometimes the extension is made only for one task, e.g. *do some action on the website I'm currently on*. 

What if I tell you that you can remove some of them and keep their functionality? In all major browsers, like Firefox, Chrom(e)ium, Safari, and their forks (and even mobile versions) the feature called [bookmarklet](https://en.wikipedia.org/wiki/Bookmarklet) is supported. This mind-blowing technology was introduced 25 years ago and is supported to `Date.now()`!

## Explanation
Bookmarklet is a bookmark that launches JavaScript instead of a website, that's it and nothing more. They're also called *favelets* sometimes.

## Examples 
*You can select JavaScript lines below and grab them to your browser's bookmark toolbar[^1].*

[^1]: Using third-party bookmarklets without verification is a really poor idea. This sentence is also valid if you replace word *bookmarklets* with *extensions*, bash *scripts* or just **software** at all.

### Wayback Machine
Access dead website or save the state of it if it's still (a)live[^2], without using any extension.

[^2]: [Portal mentioned](https://en.wikipedia.org/wiki/Still_Alive)

**Viewing**:
```javascript
javascript:location.href='https://web.archive.org/web/*/'+document.location.href.replace(/\/$/, '');
```

**Saving**:
```javascript
javascript:void(window.open('https://web.archive.org/save/'+location.href));
```

Source: [Wikipedia](https://en.wikipedia.org/wiki/Help:Using_the_Wayback_Machine#JavaScript_bookmarklet)

### Dictionaries
Send your current selection to online dictionary or translator.

```javascript
javascript:void(window.open('https://www.diki.pl/?q='+encodeURIComponent(document.getSelection().toString())));
```

### Notes
Some services even offer their own bookmarklets you can use, e.g. [Miniflux](https://miniflux.app/) have a bookmarklet to add new feeds, and [Linkding](https://linkding.link/) have one for adding bookmarks.

As you can see in my examples I use *favelets* mainly to call external services, but you can run any other scripts with some exceptions I mention in [problems](#problems) section.

## Problems
Maybe you're wondering why this technology isn't widely used if it's so good. It's caused by [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP). It was created to mitigate [XSS](https://en.wikipedia.org/wiki/Cross-site_scripting) vulnerabilities on some websites, but it also causes that some bookmarklets will not work properly on all websites. The good news is that it doesn't affect all bookmarklets, and many of them will work everywhere, especially ones I mentioned in [examples](#examples).
