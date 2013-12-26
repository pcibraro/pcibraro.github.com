---
layout: post
title: "Dependency injection made easy for the ASP.NET MVC"
date: 2008-08-21
comments: true
categories: ASP.NET
---

I decided to write this post to show how cool is Autofac for doing
dependency injection in the ASP.NET MVC framework. Autofac, for me the
Moq stepbrother  in the dependency injection arena because of its
very-easy-to-use fluent interface and nice support of lambda
expressions, comes with a built-in ASP.NET module to automatically
intercept the creation of the controllers and pass the required
dependencies to them, the only thing a programmer has to do is to
provide instances of those dependencies or expressions to build them.

Well, it is time to see Autofac in action with a practical example. If
you have the chance to use the MVC preview 4, you may notice that it
comes with a new controller "Account" to manage the website membership.
This controller receives two dependencies in the constructor,

public AccountController(IFormsAuthentication formsAuth,
MembershipProvider provider)

{

  FormsAuth = formsAuth ?? new FormsAuthenticationWrapper();

  Provider = provider ?? Membership.Provider;

}

 

public IFormsAuthentication FormsAuth

{

  get;

  private set;

}

 

public MembershipProvider Provider

{

  get;

  private set;

}

If those dependencies are not provided, it creates a default
implementation of "FormsAuthenticationWrapper" and use the Membership
singleton instance. Ok, let's make some minor changes to this controller
so we always assume that those instances must be provided by the caller
code (It will be actually responsibility of the DI container).

public AccountController(IFormsAuthentication formsAuth,
MembershipProvider provider)

{

  FormsAuth = formsAuth;

  Provider = provider;

}

We now have to initialize a container to instruct Autofac about how to
initialize or get instances of those classes. This can be done in the
global.asax file,

static IContainerProvider containerProvider;

 

protected void Application\_Start()

{

   RegisterRoutes(RouteTable.Routes);

 

   var builder = new ContainerBuilder();

 

   // Automatically register all controllers in the current assembly.

   builder.RegisterModule(new
AutofacControllerModule(Assembly.GetExecutingAssembly()));

 

   builder.Register\<MembershipProvider\>(container =\>
Membership.Provider).ExternallyOwned();

  
builder.Register\<FormsAuthenticationWrapper\>().As\<IFormsAuthentication\>().FactoryScoped();

 

   containerProvider = new ContainerProvider(builder.Build());

 

   // Hook MVC factory.

   ControllerBuilder.Current.SetControllerFactory(new
AutofacControllerFactory(containerProvider));

}

There are some lines in the code above that deserve special attention,
so let's discuss them in details:

1.

// Automatically register all controllers in the current assembly.

builder.RegisterModule(new
AutofacControllerModule(Assembly.GetExecutingAssembly()));

This line basically discovers and registers all the controllers within
the current assembly (The website itself) into the DI container.

2.

builder.Register\<MembershipProvider\>(container =\>
Membership.Provider).ExternallyOwned();

builder.Register\<FormsAuthenticationWrapper\>().As\<IFormsAuthentication\>().FactoryScoped();

The dependencies are registered in the container builder (The one that
later knows how to create instances of the dependencies). The Register
method optionally receives an lambda expression that will be used later
to create or get the dependency instance, it could be considered a sort
of lazy class construction. The container can also be used in the
expressions to resolve other dependencies, for example,

builder.Register\<MessagingService\>(c =\> new
MessagingService(c.Resolve\<IMessageRepository\>).As\<IMessagingService\>();

Another important aspect in the initialization is the scope, which
basically controls the dependency lifetime. In the code above I used two
scopes, ExternallyOwned (The instance is managed by the application) and
FactoryScoped (A new instance is created for every dependency
resolution, very useful for instances that must be used and disposed
just after, like the DataContext in Linq to SQL). Other possible scopes
could be ContainerScoped (An instance per container) or Singleton (An
instance shared between all containers).

3.

containerProvider = new ContainerProvider(builder.Build());

 

// Hook MVC factory.

ControllerBuilder.Current.SetControllerFactory(new
AutofacControllerFactory(containerProvider));

The container provider is created, and the controller factory
implementation provided by Autofac is set for the current application.

As you can see, most of the plumbing code is already provided by
Autofac, just a couple of lines were needed to start using DI in the MVC
framework.

The code sample is available to [download
here](/images/legacy/MvcDI.zip)

