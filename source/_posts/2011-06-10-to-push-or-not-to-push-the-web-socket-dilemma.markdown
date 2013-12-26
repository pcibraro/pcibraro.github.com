---
layout: post
title: "To push, or not to push, the Web Socket dilemma"
date: 2011-06-10
comments: true
categories: .NET
---

Web Sockets is a relatively new specification introduced as part of HTML
5 to support a full duplex-communication channel over http in web
browsers.  This  represents a great advance toward real-time and event
driven web applications. Before Web Sockets jumped in scene, the only
available  solutions for emulating real time notifications in web
applications were different variants of Http Long polling. Real time
notifications in this context became particularly important for specific
scenarios, such as reporting stock pricing updates, online gaming or
news reports to name a few.

All the Http polling variants were pretty much similar in nature. They
all try to emulate a bidirectional connection over http by keeping
client connections open for a period of time until some notifications
becomes available and can be sent as part of the response or the
connection times out.  However, these techniques usually require the use
of two connections for streaming data to and from the client. Another
common issue with this approach is that developers need to implement the
server side carefully to make an efficient use of the server resources.

There is, however, an area where Http polling has proved to very
effective over the years, and that is pub/sub. We can find in this area
simple usages of pub/sub over http like syndication feeds or more
complex solutions for business to business integration, but as you can
see, they are not related to real time notifications in a web browser at
all.

Web Sockets on the other hand is going to be supported natively on any
web browser compliant with HTML 5, so  removing the need of relying on
different workarounds with HTTP Ajax and timers for emulating real time
notifications in the browser.  As I said before, the specification is
relatively new so the support you find today in all the major browsers
is partial or incompatible in some cases. However, I am pretty sure Web
Sockets will become the standard technology for pushing real time data
to web browsers in the upcoming years. 

Web Sockets in Detail
---------------------

The Web Socket interface definition according to the spec looks as
follow,

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
[Constructor(in DOMString url)]
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
interface WebSocket {
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
readonly attribute DOMString URL;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
// ready state
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
const unsigned short CONNECTING = 0;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
const unsigned short OPEN = 1;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
const unsigned short CLOSED = 2;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
readonly attribute int readyState;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
// networking
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
attribute EventListener onopen;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
attribute EventListener onmessage;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
attribute EventListener onclosed;
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
void postMessage(in DOMString data);
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
void disconnect();
~~~~

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
};
~~~~

As you can see, this API is very straightforward and simple to use.
There is a constructor you can use to create a new Web Socket instance
or connection, a callback for receiving messages (onmessage) and two
additional methods for sending new messages (postMessage) or close the
socket instance (disconnect) respectively. .

A Web Socket connection is established by upgrading from the HTTP
protocol to the Web Socket protocol during an initial handshake between
the client and the server, over the same underlying TCP/IP connection.
Once the connection is established, the data can be sent back and forth
between the client and the server in full-duplex mode. Here is how you
create a new Web Socket connection in the browser,

var mySocket = new WebSocket("ws://weblogs.asp.net/cibrax");

The “ws” prefix indicates a Web Socket connection. There is also a “wss”
prefix for secure connections. Once the connection has been opened, you
can associate a handler for the “onmessage” event for start receiving
messages,

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
mySocket.onmessage = function(evt) { alert( "Message Received:  "  +  evt.data); };
~~~~

Or send messages with the “postMessage” method,

~~~~ {style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: white; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
mySocket.postMessage("Hello World!!!!");
~~~~

There are a couple of implementations already in .NET for implementing
the server side part required for pushing notifications, and some of
there are [Nugget](http://nugget.codeplex.com/) (Not really a good name
for an open source project given the existing NuGet project from
Microsoft) or [Fleck](https://github.com/statianzo/Fleck).   The WCF
team is also working on an implementation as part of the WCF Web Apis
framework and you can see some announcements
[here](http://blogs.msdn.com/b/endpoint/archive/2011/04/24/websockets-ria-js-and-wcf-web-api-at-mix-a-whole-lotta-love-for-the-web.aspx).

Web Sockets in the Cloud
------------------------

There is a particular implementation in the cloud that caught my
attention in the last few months, and that is
[“Pusher”](http://pusher.com/). “Pusher” is a service hosted in the
cloud that provides all the service side infrastructure for pushing
notifications to the different clients (browsers running in all kind of
devices or personal computers) using web sockets if it is available or a
flash-based socket communication otherwise.

 [![Pusher](http://weblogs.asp.net/blogs/cibrax/Pusher_thumb_3B1583BD.jpg "Pusher")](http://weblogs.asp.net/blogs/cibrax/Pusher_03AF728F.jpg)

Your server code publishes notifications in an specific “Pusher” channel
using a simple REST Api, and they take care of broadcasting the
notifications to all the subscribers for that channel. They provide a
javascript API  that you can use in a web page for subscribing to a
channel, and the REST API for publishing messages on the server side.
There is also different open source implementations for wrapping up the
REST API in a simple object model, and you can find here for example to
[“PusherDotNet”](https://github.com/grahamscott/pusherdotnet), a .NET
implementation in C\#. The pricing model is also very easy to
understand, you pay for a flat rate that gives you access to a specific
number of messages and connections per day.

