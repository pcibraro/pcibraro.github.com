---
layout: post
title: "Hosting your own Pub/Sub in the cloud with AppHarbor and Hermes"
date: 2011-07-25
comments: true
categories: .NET
---

As you might read in my latest post, Hermes is one our new pet projects
in Tellago for doing Pub/Sub over http. The idea is simple, but still
very useful for integration scenarios in the enterprise. The fact that
Hermes is all based on Http and uses one of the most famous open source
initiatives for NoSQL databases like MongoDB, makes this project very
appealing for the cloud as well. Many of the cloud platforms already
provide MongoDB as a service that you can use in your applications
hosted in the cloud.

AppHarbor is probably one of the best “PaaS” solutions for running .NET
applications in the cloud. The marketing slogan for this platform is
“Azure done right”, so you can take your own conclusions
![Smile](http://weblogs.asp.net/blogs/cibrax/wlEmoticon-smile_34496852.png).
AppHabor not only let us running any regular .NET application in the
cloud, but also offers MongoDB as an addon that you can configure in
your deployment, so it makes a good use case for running Hermes in the
cloud.

The Hermes’s source code that you get from the [GitHub
repository](https://github.com/TellagoDevLabs/Hermes) uses by default
NuGet for resolving all the external dependencies, which [is not
something currently supported in
AppHarbor](http://blog.dantup.com/2011/05/vote-for-native-nuget-support-on-appharbor-to-save-your-repositories-from-the-bloatmonster),
so the first thing you need to do is to prepare the visual studio
solution and all the projects to reference the binaries from a local
folder (let’s say “libs”). The only project that you should deploy to
the cloud is RestService, which represents the host for the REST
services that you might want to use for publishing or subscribing
messages to/from Hermes.

[![Hermes\_Cloud](http://weblogs.asp.net/blogs/cibrax/Hermes_Cloud_thumb_4B2839CE.png "Hermes_Cloud")](http://weblogs.asp.net/blogs/cibrax/Hermes_Cloud_3A903EE0.png)

The line you need to comment out in the RestService project file for not
using NuGet to resolve the external dependencies,

\<Target Name="BeforeBuild"\> \
    \<Exec Condition="Exists('\$(ProjectDir)packages.config')"
Command="&quot;\$(SolutionDir)..\\Tools\\nuget.exe&quot; install
&quot;\$(ProjectDir)packages.config&quot; -o
&quot;\$(SolutionDir)Packages&quot;" /\> \
 \</Target\>

Once you have that project running and using local references, you are
ready to deploy it to the cloud. You might want to create a new project
in AppHabor or use an existing one for doing this.

Every project in AppHarbor is basically mapped to a Git repository that
you can use. Every time you make a push to that repository, they take
care of building the solution, running the unit tests on it and publish
it in a public address in their platform. The instructions for deploying
the solution to their Git repository is all available in the project’s
home page, and I can say it is pretty straightforward.

[![Hermes\_Cloud1](http://weblogs.asp.net/blogs/cibrax/Hermes_Cloud1_thumb_34A5757A.png "Hermes_Cloud1")](http://weblogs.asp.net/blogs/cibrax/Hermes_Cloud1_05A339A3.png)

The public URL will be available in the home page as well once you make
the first deployment as you can see in the image above. You can also
associate a MongoDB instance to the project through the Add-ons option.

You will have to configure two additional things in the configuration
file (web.config) of the RestService, the public URL in AppHarbor and
the MondoDB connection string (available through a configuration
variable).

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
<connectionStrings>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    <add name="db.connectionString" connectionString="mongodb://appharbor:XXXXXXXX" />
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
  </connectionStrings>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
  <appSettings>
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    <add key="baseAddress" value="http://XXXXXXXXXX.apphb.com" />
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    ............
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
  </appSettings>
~~~~

And that’s all, once you have configured that in the web.config file and
pushed those changes to AppHarbor, you should be able to start using
Hermes in the cloud. You can try the Publisher and Subscriber sample
applications in the source code for testing your deployment in the
cloud.

Enjoy
![Smile](http://weblogs.asp.net/blogs/cibrax/wlEmoticon-smile_34496852.png)

