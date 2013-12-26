---
layout: post
title: "WCF Dependency Injection Behavior"
date: 2007-12-13
comments: true
categories: WCF
---

Many times the services are just part of a service layer interface or
facade that only delegate calls to other components in lower layers,
such as entity translators, business logic components, data access
components or service agents that work as proxy with other systems. To
clarify more this point, this is how the Patterns & Practices team
envisioned  a general multi-layer architecture for .NET applications,
[http://blogs.msdn.com/donsmith/archive/2006/07/21/673481.aspx](http://blogs.msdn.com/donsmith/archive/2006/07/21/673481.aspx)

When this happens, the ideal scenario to test the service code is to
replace the dependencies on lower layers by mocks or stubs objects and
focus our unit tests in the service code only. Fortunately, the
Dependency Injection pattern comes to help us here. Oran Dennison
already discussed a similar approach in this excellent post,
[http://orand.blogspot.com/2006/10/wcf-service-dependency-injection.html](http://orand.blogspot.com/2006/10/wcf-service-dependency-injection.html).

During the course of this post I will take a different path using the
"Dependency Injection Container" sample that comes with [Object
Builder](http://www.codeplex.com/ObjectBuilder) . This sample also
includes good documentation about the supported features.

So, suppose we have a service implementation with the following
dependencies:

[ServiceContract(Namespace =
[http://Microsoft.ServiceModel.Samples](http://microsoft.servicemodel.samples/))]

public interface ICustomer

{

   [OperationContract]

   string GetFullName(string customerId);

}

 

public class CustomerService : ICustomer

{

  ICustomerBusinessComponent businessComponent;

 

  public CustomerService(ICustomerBusinessComponent businessComponent)

  {

    this.businessComponent = businessComponent;

  }

 

  public string GetFullName(string customerId)

  {

    return businessComponent.GetFullName(customerId);

  }

}

 

public interface ICustomerDataAccess

{

  string GetFullName(string customerId);

}

 

public class CustomerDataAccess : ICustomerDataAccess

{

  public CustomerDataAccess()

  {

  }

 

  public string GetFullName(string customerId)

  {

    return "FOO";

  }

}

As you can see in the code above, our service has a direct dependency
with a business component, and at the same time, our business component
has a dependency with another component in the data access layer. What
we want to do here is to inject both dependencies at runtime using the
Dependency injection pattern.

The equivalent code with the Dependency Container looks could be,

DependencyContainer container = new DependencyContainer();

 

container.RegisterTypeMapping\<ICustomerBusinessComponent,
CustomerBusinessComponent\>();

container.RegisterTypeMapping\<ICustomerDataAccess,
CustomerDataAccess\>();

 

CustomerService service = container.Get\<CustomerService\>();

Quite easy, the only requirement is the mapping, otherwise the container
will not know how to create instances of the interfaces. Now, let's move
forward to try configuring this in WCF using a behavior as extensibility
point. WCF supports an extension called IInstanceProvider that controls
the lifecycle of a WCF service instance. We will use one this provider
to hook up our custom code and inject the dependencies at runtime.

public class DIInstanceProvider : IInstanceProvider

{

  private Type serviceType;

  List\<TypeMapping\> typeMappings;

 

  public DIInstanceProvider(Type serviceType, List\<TypeMapping\>
typeMappings)

  {

    this.serviceType = serviceType;

    this.typeMappings = typeMappings;

  }

 

  public object GetInstance(InstanceContext instanceContext)

  {

    return GetInstance(instanceContext, null);

  }

 

  public object GetInstance(InstanceContext instanceContext, Message
message)

  {

    DependencyContainer container = new DependencyContainer();

 

    foreach (TypeMapping typeMapping in this.typeMappings)

    {

      container.RegisterTypeMapping(typeMapping.TypeRequested,
typeMapping.TypeToBuild);

    }

 

    return container.Get(this.serviceType);

  }

 

  public void ReleaseInstance(InstanceContext instanceContext, object
instance)

  {

 

  }

}

 

The typeMapping that you can see there in the code are extensions that
we are going to set up through configuration. This class basically
represents a mapping between a requested type (Which usually is an
interface) and the type to build by the object builder (Which usually is
the concrete implementation).

public class TypeMapping

{

  private Type typeRequested;

  private Type typeToBuild;

 

  public TypeMapping(Type typeRequested, Type typeToBuild)

  {

    this.typeRequested = typeRequested;

    this.typeToBuild = typeToBuild;

  }

 

  public Type TypeRequested

  {

    get { return typeRequested; }

  }

 

  public Type TypeToBuild

  {

    get { return typeToBuild; }

  }

}

Once we have the behavior implementation, the final step is to configure
it in our application. This can be done as follows,

\<behaviors\>

      \<serviceBehaviors\>

        \<behavior name="Behaviors1"\>

          \<dependencyInjection\>

            \<typeMappings\>

              \<add name="DataAccess"
typeRequested="SampleService.ICustomerDataAccess, SampleService"
typeToBuild="SampleService.CustomerDataAccess, SampleService"/\>

              \<add name="BusinessComponent"
typeRequested="SampleService.ICustomerBusinessComponent, SampleService"
typeToBuild="SampleService.CustomerBusinessComponent, SampleService"/\>

            \</typeMappings\>

          \</dependencyInjection\>

        \</behavior\>

      \</serviceBehaviors\>   

\</behaviors\>

The configuration looks simple, I just mapped two interfaces to the real
implementations (Which could be replaced during testing through mocks or
stub objects). No additional code needed for the WCF service :-)

As a final comment, the dependency container also supports method
injection (This is to inject the dependencies that are arguments of a
method) and setter injections (This is to inject dependencies through
property setters).

Download the complete sample from this
[location](/images/legacy/WCFDependencyInjection.zip).

