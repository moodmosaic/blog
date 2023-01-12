---
layout: basic
title: Those pesky build servers
---

Using a build server can actually hurt productivity:

* It introduces a single point of failure
* It can yield false-positives
* It can yield false-negatives
* It consumes time and energy to configure it properly

*Think about it* - you may not need a build server, after all.

## It introduces a single point of failure

When things fail on the build server, it looks bad. It doesn't matter if the build succeeds on the developer machines. - *Seriously?*

## It can yield false-positives

The build might succeed on the build server, while fail on the developer machines. - *Now what?*

## It can yield false-negatives

The build might fail on the build server for a whole lot of (unrelated) reasons to the build itself. Does that mean we're screwed? - *We might not.*

## It consumes time and energy to configure it properly

It's one of those days where you end up spending more time getting the build server to work, than working on something else that's probably more useful.

## So...

Bring fun back to work:

* Run the build on a few developer machines.
* Report back whether it succeeds or not, and rely on that.

Get rid of those pesky build servers.
