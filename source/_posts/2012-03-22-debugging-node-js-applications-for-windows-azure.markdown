---
layout: post
title: "Debugging Node.js applications for Windows Azure"
date: 2012-03-22
comments: true
categories: .NET
---

In case you are developing a new web application with Node.js for
Windows Azure, you might notice there is no easy way to debug the
application unless you are developing in an integrated IDE like Cloud9.
For those that develop applications locally using a text editor (or
WebMatrix) and Windows Azure Powershell for Node.js, it requires some
steps not documented anywhere for the moment.

I spent a few hours on this the other day I practically got nowhere
until I received some help from [Tomek](http://tomasz.janczuk.org) and
the rest of them. The IISNode version that currently ships with the
Windows Azure for Node.js SDK does not support debugging by default, so
you need to install the IISNode full version available in the [github
repository](https://github.com/windowsazure/iisnode/downloads). 

Once you have installed the full version, you need to enable debugging
for the web application by modifying the web.config file

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: white; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
<iisnode 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
     debuggingEnabled="true"
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: white; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
     loggingEnabled="true"
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: #f4f4f4; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
     devErrorsEnabled="true"
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; background-color: white; margin: 0em; border-left-style: none; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; border-right-style: none; font-size: 8pt; overflow: visible; padding-top: 0px"}
   />
~~~~

The xml above needs to be inserted within the existing
“\<system.webServer/\>” section.

The last step is to open a WebKit browser (e.g. Chrome) and navigate to
the URL where your application is hosted but adding the segment “/debug”
to  the end. The full URL to the node.js application must be used, for
example,
[http://localhost:81/myserver.js/debug](http://localhost:81/myserver.js/debug)

That should open a new instance of Node inspector on the browser, so you
can debug the application from there.

Enjoy!!

