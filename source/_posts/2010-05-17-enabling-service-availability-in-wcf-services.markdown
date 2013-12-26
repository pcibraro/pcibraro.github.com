---
layout: post
title: "Enabling Service Availability in WCF Services"
date: 2010-05-17
comments: true
categories: .NET
---

It is very important for the enterprise to know which services are
operational at any given point. There are many factors that can affect
the availability of the services, some of them are external like a
database not responding or any dependant service not working. However,
in some cases, you only want to know whether a service is up or down, so
a simple heart-beat mechanism with “Ping” messages would do the trick.
Unfortunately, WCF does not provide a built-in mechanism to support this
functionality, and you probably don’t to implement a “Ping” operation in
any service that you have out there. For solving this in a generic way,
there is a WCF extensibility point that comes to help us, the “Operation
Invokers”. In a nutshell, an operation invoker is the class responsible
invoking the service method with a set of parameters and generate the
output parameters with the return value.

What I am going to do here is to implement a custom operation invoker
that intercepts any call to the service, and detects whether a “Ping”
header was attached to the message. If the “Ping” header is detected,
the operation invoker returns a new header to tell the client that the
service is alive, and the real operation execution is omitted. In that
way, we have a simple heart beat mechanism based on the messages that
include a "Ping” header, so the client application can determine at any
point whether the service is up or down.

My operation invoker wraps the default implementation attached by
default to any operation by WCF.

~~~~ {.code}
internal class PingOperationInvoker : IOperationInvoker
{
  IOperationInvoker innerInvoker;
  object[] outputs = null;
  object returnValue = null;

  public const string PingHeaderName = "Ping";
  public const string PingHeaderNamespace = "http://tellago.serviceModel";

  public PingOperationInvoker(IOperationInvoker innerInvoker, OperationDescription description)
  {
    this.innerInvoker = innerInvoker;
    outputs = description.SyncMethod.GetParameters()
          .Where(p => p.IsOut)
          .Select(p => DefaultForType(p.ParameterType)).ToArray();

    var returnValue = DefaultForType(description.SyncMethod.ReturnType);
  }

  private static object DefaultForType(Type targetType)
  {
    return targetType.IsValueType ? Activator.CreateInstance(targetType) : null;
  }
        
  public object Invoke(object instance, object[] inputs, out object[] outputs)
  {
    object returnValue;

    if (Invoke(out returnValue, out outputs))
    {
      return returnValue;
    }
    else
    {
      return this.innerInvoker.Invoke(instance, inputs, out outputs);
    }
  }

  private bool Invoke(out object returnValue, out object[] outputs)
  {
    object untypedProperty = null;
    if (OperationContext.Current
~~~~

~~~~ {.code}
.IncomingMessageProperties.TryGetValue(HttpRequestMessageProperty.Name, out untypedProperty))
    {
      var httpRequestProperty = untypedProperty as HttpRequestMessageProperty;
      if (httpRequestProperty != null)
      {
        if (httpRequestProperty.Headers[PingHeaderName] != null)
        {
          outputs = this.outputs;

          if (OperationContext.Current
~~~~

~~~~ {.code}
.IncomingMessageProperties.TryGetValue(HttpRequestMessageProperty.Name, out untypedProperty))
          {
             var httpResponseProperty = untypedProperty as HttpResponseMessageProperty;

             httpResponseProperty.Headers.Add(PingHeaderName, "Ok");
          }

          returnValue = this.returnValue;

          return true;
       }
    }
   }           

   var headers = OperationContext.Current.IncomingMessageHeaders;
   if (headers.FindHeader(PingHeaderName, PingHeaderNamespace) > -1)
   {
     outputs = this.outputs;

     MessageHeader<string> header = new MessageHeader<string>("Ok");
     var untyped = header.GetUntypedHeader(PingHeaderName, PingHeaderNamespace);

     OperationContext.Current.OutgoingMessageHeaders.Add(untyped);

     returnValue = this.returnValue;

     return true;
   }            

   returnValue = null;
   outputs = null;

   return false;
 }
}
~~~~

[](http://11011.net/software/vspaste)

The implementation above looks for the “Ping” header either in the Http
Request or the Soap message. \
The next step is to implement a behavior for attaching this operation
invoker to the services we want to monitor.

~~~~ {.code}
[AttributeUsage(AttributeTargets.Method 
  | AttributeTargets.Class, AllowMultiple = false, Inherited = true)]
 public class PingBehavior : Attribute, IServiceBehavior, IOperationBehavior
 {
   public void AddBindingParameters(ServiceDescription serviceDescription, 
       ServiceHostBase serviceHostBase, Collection<ServiceEndpoint> endpoints, 
       BindingParameterCollection bindingParameters)
   {
   }

   public void ApplyDispatchBehavior(ServiceDescription serviceDescription, 
        ServiceHostBase serviceHostBase)
   {
   }

   public void Validate(ServiceDescription serviceDescription, 
        ServiceHostBase serviceHostBase)
   {
     foreach (var endpoint in serviceDescription.Endpoints)
     {
       foreach (var operation in endpoint.Contract.Operations)
       {
         if (operation.Behaviors.Find<PingBehavior>() == null)
              operation.Behaviors.Add(this);
       }
     }
   }

   public void AddBindingParameters(OperationDescription operationDescription, 
        BindingParameterCollection bindingParameters)
   {
   }

   public void ApplyClientBehavior(OperationDescription operationDescription, 
          ClientOperation clientOperation)
   {
   }

   public void ApplyDispatchBehavior(OperationDescription operationDescription, 
        DispatchOperation dispatchOperation)
   {
      dispatchOperation.Invoker = 
          new PingOperationInvoker(dispatchOperation.Invoker, operationDescription);
   }

   public void Validate(OperationDescription operationDescription)
   {

   }
 }
~~~~

As an operation invoker can only be added in an “operation behavior”, a
trick I learned in the past is that you can implement a service behavior
as well and use the “Validate” method to inject it in all the
operations, so the final configuration is much easier and cleaner. You
only need to decorate the service with a simple attribute to enable the
“Ping” functionality.

~~~~ {.code}
[PingBehavior]
public class HelloWorldService : IHelloWorld
{
  public string Hello(string name)
  {
    return "Hello " + name;
  }
}
~~~~

[](http://11011.net/software/vspaste)

On the other hand, the client application needs to send a dummy message
with a “Ping” header to detect whether the service is available or not.
In order to simplify this task, I created a extension method in the WCF
client channel to do this work.

~~~~ {.code}
public static class ClientChannelExtensions
{
  const string PingNamespace = "http://tellago.serviceModel";
  const string PingName = "Ping";

  public static bool IsAvailable<TChannel>(this IClientChannel channel, 
       Action<TChannel> operation)
  {
    try
    {
      using (OperationContextScope scope = new OperationContextScope(channel))
      {
        MessageHeader<string> header = new MessageHeader<string>(PingName);
        var untyped = header.GetUntypedHeader(PingName, PingNamespace);
        OperationContext.Current.OutgoingMessageHeaders.Add(untyped);

        try
        {
          operation((TChannel)channel);
          var headers = OperationContext.Current.IncomingMessageHeaders;
          if (headers.Any(h => h.Name == PingName && h.Namespace == PingNamespace))
          {
            return true;
          }
          else
          {
            return false;
          }
        }
        catch (CommunicationException)
        {
          return false;
        }
       }
     }
     catch (Exception)
     {
       return false;
     }
   }
  }
~~~~

[](http://11011.net/software/vspaste)

This extension method basically adds a “Ping” header to the request
message, executes the operation passed as argument (Action\<TChannel\>
operation), and looks for the corresponding “Ping” header in the
response to see the results.

The client application can use this extension with a single line of
code,

~~~~ {.code}
var client = new ServiceReference.HelloWorldClient();
var isAvailable = client.InnerChannel.IsAvailable<IHelloWorld>((c) => c.Hello(null));
~~~~

[](http://11011.net/software/vspaste)

The “isAvailable” variable will tell the client application whether the
service is available or not.

You can download the complete implementation from this
[location](/images/legacy/Ping.zip).

 

[ ](http://11011.net/software/vspaste)

