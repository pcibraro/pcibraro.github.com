---
layout: post
title: "OAuth in action – Linq2Twitter"
date: 2009-08-19
comments: true
categories: .NET
---

The other day I came across a pretty cool project,
[Linq2Twitter](http://linqtotwitter.codeplex.com/), that basically
implements a linq provider for consuming the Twitter REST Api.  This
project is not only interesting because it provides an intuitive model
for incorporating Twitter calls into any existing application, but also
because it shows how to use the OAuth authentication mode that the
Twitter Api supports.

As it is common with other linq providers, this library provides a root
class “TwitterContext” for executing queries or calling other services
in the API.

So you can do things like this,

var twitterCxt = new TwitterContext();

var tweets =

    from tweet in twitterCtx.Status

    where tweet.Type == StatusType.Friends

    select tweet;

tweets.ToList().ForEach(

    tweet =\> Console.WriteLine(

        "Friend: {0}\\nTweet: {1}\\n",

        tweet.User.Name,

        tweet.Text));

The example above shows some code to gets the status of all your
friends.

Before using any of the services available in the API, you must first
authenticate in Twitter using either basic authentication (The
traditional way with username and password) or OAuth. The authentication
mode is specified in the twitter context.

In case you decide to go with OAuth, it is very simple to use. You only
need to provide the OAuth secret and key associated to an application
that you previously registered in Twitter.

Console.Write("Consumer Key: ");

twitterCtx.ConsumerKey = Console.ReadLine();

Console.Write("Consumer Secret: ");

twitterCtx.ConsumerSecret = Console.ReadLine();

string link = twitterCtx.GetAuthorizationPageLink(false, false);

Console.WriteLine("Authorization Page Link: {0}\\n", link);

Console.WriteLine("Next, you'll need to tell Twitter to authorize
access. This program will not have access to your credentials, which is
the benefit of OAuth.  Once you log into Twitter and give this program
permission, come back to this console and press Enter to complete the
authorization sequence.\\n\\nPress Enter now to continue.");

Console.ReadKey();

// launches browser so you can log in and give permissions

Process.Start(link);

Console.WriteLine("\\nYou should see your browser navigate to Twitter,
saying that your application wants to access your Twitter account. Once
you've authorized this program, return to this console and press any key
to execute the LINQ to Twitter code.");

Console.ReadKey();

var uri = new Uri(link);

NameValueCollection urlParams = HttpUtility.ParseQueryString(uri.Query);

string oAuthToken = urlParams["oauth\_token"];

twitterCtx.RetrieveAccessToken(oAuthToken);

if (twitterCtx.AuthorizedViaOAuth)

OAuth requires a browser session so you can log into twitter and
authorize the client application to get your data. This is not a problem
if the client application is a web application, the authentication and
authorization processes flow very natural. However, in the case of
Windows or Console Application, a browser instance needs to be created,
this is what the example above is doing in the line “Process.Start”. If
you are already authenticated in twitter, you will see this page where
you need to authorize the application that wants to get access to your
data.

[![Twitter](http://weblogs.asp.net/blogs/cibrax/Twitter_thumb_322D3775.gif "Twitter")](http://weblogs.asp.net/blogs/cibrax/Twitter_03972E93.gif)

It looks like [Andrew Arnott](http://blog.nerdbank.net/) have been
involved in the OAuth implementation for this library, which is a really
good thing since he is the main contributor in two of the most stables
projects for OpenID
([DotNetOpenId](http://code.google.com/p/dotnetopenid/)) and OAuth
([DotNetOpenAuth](http://dotnetopenauth.net:8000/)) in the .NET world. 

