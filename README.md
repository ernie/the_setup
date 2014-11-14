# The Setup

## The Prelude

There are a number of ways we developers try to simplify our lives. Many of them
involve adopting [language-specific tools](http://pow.cx/) that work fine for us
locally, but have almost nothing to do with the way that our code is going to
eventually run in production. In my experience, this produces pain when code has
to make that transition from development to production.

In some larger teams, that pain is felt by another group, so it's easy for
developers to take a "not my problem" attitude. The teams I've worked with more
recently are small. We feel the pain. That's a **good** thing, since we're
incentivized to minimize it.

I wrote this in order to share a setup I've been using locally for over a year,
and refining recently, with my coworkers at [nVisium](https://nvisium.com).
I've seen bits and pieces of similar setups around the web, mostly in the form
of a quick mention in a blog post or a tweet, but I've not seen a comprehensive
guide to getting up and running on a current version of OS X before.

After seeing this writeup, nVisium was quick to encourage me to share it more
broadly. nVisium is cool like that. We're also really awesome at helping people
secure their apps, so you should probably [check us out](https://nvisium.com)
after you're done working through this guide. Or before. I can wait.

## The Expectations

The Setup isn't intended to be a replacement for reading the docs or knowing
how to best configure the tools it uses. It's a starting point on the path
toward building an environment that more accurately represents the way our apps
are served up in the real world, and hopefully helps us squash more bugs before
they get the chance to affect a single user.

Once finished, your Mac will be configured to host two custom TLDs, `.devel` and
`.staging`, and you'll have a solid understanding of how to set up new apps, or
an entire SOA-style family of apps, for use with The Setup. You'll even be able
to run a suite of acceptance tests against a full application stack running in
an isolated local staging environment under Docker, if you want. And why
*wouldn't* you want? That sounds awesome!

The base system assumed by this guide is Mac OS X 10.10 Yosemite, with a working
install of [Homebrew](http://brew.sh). While it should work for less current
versions of OS X, changes in launchd may require a few tweaks to plist files.

## The Outline

This guide has been broken into sections for each component involved, each with
a README specific to its setup. They're interlinked in such a way as to allow
navigation for step-by-step setup. Therefore, it is recommended that you clone
this repository, then work from the clone while following along with the
web-based copy of the documentation. Any commands, unless otherwise noted, are
to be run from the directory of the component whose README you are viewing.

The components of The Setup:

1. [nginx](01_nginx/)
2. [dnsmasq](02_dnsmasq/)
3. [pfctl](03_pfctl/)
4. [foreman](04_foreman/)
5. [docker](05_docker/)

Let's get started by setting up [nginx](01_nginx/).
