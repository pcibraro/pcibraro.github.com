---
layout: post
title: "Managing the SO-Aware Repository with PowerShell"
date: 2010-09-01
comments: true
categories: .NET
---

As Jesus mentioned in this
[post](http://www.tellagostudios.com/tellago-studios-launching),
SO-Aware provides three interfaces for managing the service repository.
An OData API in case you want to integrate third applications with the
repository. OData is a pure http API that can be easily consumed in any
platform using a simple http client library. The management portal,
which is an ASP.NET MVC user interface layered on top of the OData API
and probably the one most people will use. And finally, a PowerShell
provider that also mounts on top of the OData API to allow
administrators to automate management tasks over the repository with
scripting. 

The SO-Aware PowerShell provider, in that sense offers around 40
commands that enables simple management scenarios like registering
bindings or services or more complex scenarios that involves testing
services or sending alerts when a service is not properly working.  

This provider can be registered as an snapin in an existing script using
the following command,

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
$snapin = get-pssnapin  | select-string "SOAwareSnapIn"if ($snapin -eq $null){    Add-PSSnapin "SOAwareSnapIn"}
~~~~

\

Once you have registered the snapin, you can start using most of the
commands for managing the repository.

The first and more important command is “Set-SWEndpoint”, which allows
you to connect to an existing SO-Aware instance. This command receives
the OData service location as first argument, and it looks as follow,

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
Set-SWEndpoint -uri http://localhost/SOAware/ServiceRepository.svc
~~~~

\

 

As next step, you can start managing or querying data from the
repository using the rest of the commands. For instance, the following
example registers a new binding in the repository only if it was not
created already

~~~~ {#codeSnippet style="BORDER-BOTTOM-STYLE: none; TEXT-ALIGN: left; PADDING-BOTTOM: 0px; LINE-HEIGHT: 12pt; BORDER-RIGHT-STYLE: none; BACKGROUND-COLOR: #f4f4f4; MARGIN: 0em; PADDING-LEFT: 0px; WIDTH: 100%; PADDING-RIGHT: 0px; FONT-FAMILY: 'Courier New', courier, monospace; DIRECTION: ltr; BORDER-TOP-STYLE: none; COLOR: black; FONT-SIZE: 8pt; BORDER-LEFT-STYLE: none; OVERFLOW: visible; PADDING-TOP: 0px"}
function RegisterBinding([string]$name,[string]$type,[string]$xml){    $binding = GetBinding($name);    if(!$binding)    {       Add-SWBinding -Name $name -BindingType $type -Configuration $xml    }}function GetBinding([string]$name){    $bindings = Get-SWBindings    foreach($binding in $bindings)    {        if($binding.Name -eq $name)        {            return $binding        }    }}RegisterBinding "stsBinding" "ws2007HttpBinding" "<binding>        <security mode='Message'>            <message clientCredentialType='UserName' establishSecurityContext='false' negotiateServiceCredential='false'/>        </security> </binding>";
~~~~

\

As you can see, this provider brings a powerful toy that administrators
in any organization can use to manage services or governance aspects by
leveraging their scripting knowledge.

