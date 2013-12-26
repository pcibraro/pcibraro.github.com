---
layout: post
title: "Running the “Express” web development framework on Node for Windows"
date: 2011-08-05
comments: true
categories: .NET
---

As some of you might know, with the release of Node version 0.5.2, there
is now support for Windows. It’s a single executable “node.exe” that you
need to start using node on your machine, and you can get it from this
[location](http://blog.nodejs.org/2011/07/22/node-v0-5-2/).

Express is a Sinatra inspired web development framework for Node. You
can find more information about this framework
[here](http://jade-lang.com/).

Once you download node in your system, you need to setup an optional
environment variable “NODE\_PATH”, which points to the root path or
location that Node will use to resolve the external module references. A
module in Node represents an unit of code reuse, pretty similar to a
library or an assembly in .NET, but it’s not something binary, it is
just JavaScript code. You can also optionally set the node executable
path in the “PATH” environmental variable to run the exe from any
location.

In my case, I created the following structure on my disk for running
Node.

d:\>

----\> NodeJs

------\> Node.exe

------\> Applications

“NODE\_PATH” and “PATH” both points to the folder “NodeJS” on my “d:”
drive. All the applications or samples will be deployed in the
“Applications” folder.

In the Linux world, Node also comes with a very useful package manager
“NPM” that automatically resolves external module dependencies for you. 
As analogy with other platforms like Ruby or .NET, “NPM” would be
equivalent to Ruby GEMS or NuGet.

Unfortunately, “NPM” is still not ready for being used on Windows, there
are some technical details that need to be addressed to migrate the
current implementation to windows. Good news is that Jeroen Janssen has
created a similar implementation in Python so you can run it in windows.
The name of this implementation is “ryppi” and you can get it from this
[github repository](https://github.com/japj/ryppi). It’s a single Phyton
script “ryppi.py” that currently works with Python 2.X only. If you have
Python 2.x already installed on your system, you can run the script as
follow to resolve an external dependency “python ryppi install
\<PACKAGE\_NAME\>”.

In the case of “Express”, you can run  “python ryppi install Express”
from the location where you set the “NODE\_PATH” variable. After running
the command, you should be able to see a new folder “node\_modules” in
that location, and an “express” folder inside of it. That’s the only
package you need to run the “Express” core, but you also might want to
download “jade” and “saas” for using the [Jade View
Engine](http://jade-lang.com/) and the [SAAS](http://sass-lang.com/)
engine respectively.

In addition to the “Express” core library, there is an node application
you can use to setup the initial folder structure for your web
applications. The “Express” application is available under
“node\_modules/express/bin/express”, and you can run it with node
itself. Bad news is that this application does not run on windows
because it uses the “child\_process” module, which is not currently
supported on windows. I modified that application to get rid of the
“child\_process” module and use all the functionality available in
windows. This updated version is available to download from
[here](/images/legacy/express).

If you replace the version in “node\_modules/express/bin/express” by the
one I updated, you should able to run it with node as follow “node.exe
./node\_modules/express/bin/express –help”. That will show the help
instructions to setup a new web application. A new web application with
all the defaults can be created by just passing a name to that “express”
application, “node.exe ./node\_modules/express/bin/express mywebapp”.
One thing to have in mind is that the application will be created as a
new folder in the current location, so you might want to run it from
“NodeJs/Applications” if you are planning to have the web application in
there.

The command will generate a new folder “mywebapp” with all the
boilerplate structure you need to run a web application from scratch. 
The folder structure looks like this,

- public (contains all the JavaScript, images and stylesheets that make
up your site)

- views (contains all the views)

- app.js (the root application)

You can start the web application by running “node app”, and that’s it,
you have your first “express” application running on windows
![Smile](http://weblogs.asp.net/blogs/cibrax/wlEmoticon-smile_32D32B80.png).

