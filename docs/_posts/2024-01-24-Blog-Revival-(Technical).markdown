---
layout: post
title:  "Revival Of This Site (Part 2)"
date:   2024-01-24
categories: [news, meta, technical]
---


# Introduction
As referenced in part 1, I'm trying to bring this site back from the dead. That, of course, did not come without its share of technical issues, which I will now reserruct my site by talking about.

Notably, this section will lack many screenshots. Being so focused on fixing the actual blog, I neglected that it would be a great post! I hope the text is sufficient, I'll keep it short ;). 

## The problems
When I made this site, I had used an (apparently) pretty deep mix of hacks to get it to work as I wanted it to. Of course, at the time, I didn't quite understand the value of documenting said hacks... Predictably, I regret that this weekend. Typically with jekyll on Github Pages, you would run `bundle exec jekyll serve` to spin up your own instance of the site for development purposes. However, thanks to my incredibly dated Jekyll website, bundle was not very happy to work with the resources I had on my local machine. So unhappy, in fact, that the Ruby interpreter itself threw its hands up in a segmentation fault (!!).

One long update later, I managed to get Jekyll to run, however my page ceased to work properly. The CSS was not applying, the layouts were not working, and everything was proportioned wrongly. Here is how I got Jekyll to run, and then proceeded to fix my issues.

# The solutions
## Problem #1 - Webrick
This is a [somewhat well documented issue](https://github.com/jekyll/jekyll/issues/8523), and is thankfully an easy fix. If Jekyll complains about lacking the webrick library, running `bundle add webrick` in the working directory worked great for me.

Otherwise, you can also add `gem "webrick", "~>VERSION"` to your Gemfile (where VERSION is the library version).

## Problem #2 - json
This one was odd, because I'm pretty sure the `json` library is part of the standard libraries for ruby. Either way, it manifested for me, and the fix was simple. If `bundle exec` complains, throw `gem "json", ">=VERSION"` into your genfile works.

Note, if the json library updates in an unpredictable way and ceases to work, consider downgrading by replacing `>=VERSION` with a more specific `~>VERSION`.

## Problem #3 - Theming
This was something I was not going to give up lightly. The default Jekyll theme, minima, has seemingly ceased development for quite some time now. In the [repository on GitHub](https://github.com/jekyll/minima), development has seemingly halted. This includes development of a sub-theme feature of the theme, which means no dark mode for me.

I've seen [others](https://blog.slowb.ro/dark-theme-for-minima-jekyll/) point to updating the Gemfile to point at the minima repository, to pull the latest changes and use the feature. This did not work for me (bundle would cite compatibility issues with GitHub Pages). Instead, I opted to create an `assets` folder for my project. This folder is unique, in that Jekyll will overwite other assets with the contents of this folder where applicable. This was handy, as I could just pull a generated assets file from `_site/`, and place it in the overwrite folder.

It requires more knowledge of how CSS works, but it's not so bad. Modifying your own main.css is both a good exercise, and offers you a lot of fine-grain control over the looks of your site. This is how I achieved the coloring results of my page.

# Conclusion
With those problems out of the way, the site looks and behaves as I would expect! Until next time, when GitHub Pages updates and breaks my whole stack again. And never forget your documentation! There's a reason I write all of this down, nowadays.