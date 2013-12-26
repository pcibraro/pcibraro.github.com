---
layout: post
title: "Leveraging NuGet as a central repository for PowerShell modules"
date: 2011-11-28
comments: true
categories: .NET
---

We have been working a lot lately with PowerShell as part of our star
product at Tellago Studios, “Moesion”. One of the main features we
provide in Moesion is the ability to execute PowerShell commands
remotely in a given server using a web mobile interface (You can read
more in my previous
[post](http://weblogs.asp.net/cibrax/archive/2011/09/19/pub-sub-in-the-cloud-for-it-management.aspx)
about Moesion).

One of the things we realized in all this time is that PowerShell lacks
of a central repository where IT guys or we, the developers, can easily
grab and reuse commands.  All the commands or modules are basically
spread across multiple places or websites, like personal blogs, TechNet
or CodePlex projects to name a few making the search of them very hard.
You are usually limited to use your favorite search engine and copy what
you find. In addition, there is not an easy way to reuse, extend or
version these commands, which also limits any contribution that you
could make to the community. 

My friend Jose wrote a great
[post](http://joseoncode.com/2011/11/24/sharing-powershell-modules-easily/)
the other day about the importance of reusing PowerShell modules, and
what is the mechanism to reuse them. Jose, however, based his post in a
custom implementation using a GIT repository for storing the modules.

We have NuGet in the .NET platform for sharing and reusing existing
libraries or code, so why can’t just leverage it for reusing PowerShell
modules as well ?. Some teams in Microsoft are using NuGet for
distributing libraries and binaries so it would be a great thing for all
of us if they also distribute the scripting interfaces in PowerShell
using NuGet. This applies to the .NET OS community as well.

In fact, it looks like [Andrew Nurse](http://vibrantcode.com) had the
same idea and implemented a project for this in BitBucket,
[PsGet](https://bitbucket.org/anurse/psget).

