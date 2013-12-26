---
layout: post
title: "Streaming large content with WCF and deferred execution"
date: 2008-06-10
comments: true
categories: WCF
---

I will use this post to discuss an scenario that you may run into while
working with WCF, a service that returns a lot of objects (Or large
data) to the client applications. This scenario is not about
transferring files, that is a completely different story, and I already
discussed it some time ago in another post.

The scalability of the service's host may be affected if the service is
not well designed or configured. WCF will use large memory buffers to
keep the complete message before it start sending that message through
the wire.

Using data contracts or XML serializable classes in this scenario does
not scale well because the complete object graph has to be loaded in
memory before the corresponding serializer (Data Contract serializer or
Xml Serializer) transform it into an stream of bytes.

In addition, if the service throttling settings are not configured
carefully for that service, a large number of requests will practically
consume all the available resources (If that happens, the server will
stop processing additional requests and you may run into a OutOfMemory
exception as well). For more information about Service Throttling, I
recommend this [Kenny Wolf's post](http://kennyw.com/indigo/150).

Before getting into the streaming solution, which looks very complex at
first glance, you should consider refactoring your service's API to
support data paging. If you use paging, the design of the final service
will be simpler and you should not have any of the mentioned problems.
What's more, streaming does not work well with message security, the
default behavior for WCF is to buffer all messages when message security
is set for the channel. In order to combine message security with
streaming you should use the Chunking Channel.

Data Paging design sample,

[XmlRoot("Customer")]

public class Customer

{

  public string FirstName { get; set; }

 

  public string LastName { get; set; }

 

  public string Address { get; set; }

}

 

[XmlRoot("Customers")]

public class Customers

{

    public Customer[] Customers { get; set; }

}

 

public interface ISampleService

{

  [OperationContract]

  Customers GetAllCustomers(int page, int count, out int
totalCustomers); 

}

The client application will have to call that method as many times as it
needs to retrieve all the customers. The service should have a limit in
the number of customers it can return per call, otherwise, we will have
the same problems if the client application asks for a large number of
customers.

Deferred execution is a cool feature introduced in .NET 2.0 for
enumerations. Basically the enumeration does not happen until some
calling code wants to examine the enumeration.

Let's discuss now how deferred execution can be achieved in WCF with the
help of the streaming model.

​1. The first extensibility point that we will need for this solution is
a custom BodyWriter class. A BodyWriter has the knowledge to serialize
an entire object graph as xml into the body of a soap message. The class
Message has an static method to create a new message from a  BodyWriter
instance.

public abstract class Message : ....

{

    public static Message CreateMessage(MessageVersion version, string
action, BodyWriter body)

}

The "BodyWriter" class is an abstract and contains a single method to
serialize the complete object graph into the soap body.

public class CustomBodyWriter : BodyWriter

  {

    private IEnumerable\<Customer\> customers;

 

    public CustomBodyWriter(IEnumerable\<Customer\> customers)

      : base(false) // False should be passed here to avoid buffering
the message

    {

      this.customers = customers;

    }

 

    protected override void
OnWriteBodyContents(System.Xml.XmlDictionaryWriter writer)

    {

      XmlSerializer serializer = new XmlSerializer(typeof(Customer));

      writer.WriteStartElement("customers");

      foreach (Customer customer in customers)

      {

        serializer.Serialize(writer, customer);

      }

      writer.WriteEndElement();

    }

  }

As you can see in the code above, our custom implementation of the
"OnWriteBodyContents" receives an enumeration of objects that we want to
serialize into the message. A "false" value is passed as argument in the
constructor to the base class to specify that our custom implementation
can not be buffered.

​2. We will use deferred execution to retrieve all the customers from
the database (The following code actually emulates that),

public IEnumerable\<Customer\> GetAllCustomersImpl()

{

   for(long i = 0; i \< 1000; i++) //All the customers should be read
from the database

   {

      yield return new Customer { FirstName = "Foo", LastName = "Bar",
Address = "FooBar 123" };

   }

 

   yield break;

}

What's it is important here is that we are not returning all the
customer objects at the same time (we are returning one at time, so they
are not loaded in memory) using the "yield" operator.

​3. The service implementation returns a new message created from the
custom "CustomBodyWriter" class.

public class SampleService : ISampleService

  {

    \#region ISampleService Members

 

    public Message GetAllCustomers()

    {

      Message message = Message.CreateMessage(MessageVersion.Soap11,
"GetAllCustomers", new CustomBodyWriter(GetAllCustomersImpl()));

      return message;

    }

 

    \#endregion

    

  }

 

The first two arguments of the CreateMessage method specifies the soap
version and soap action for the response message.

​4. The client application also reads one customer object at time from
the response message,

static IEnumerable\<Customer\> GetAllCustomers(Message message)

{

  XmlReader reader = message.GetReaderAtBodyContents();

  if (reader.LocalName != "customers")

  {

     throw new Exception("The service returned an invalid message");

  }

 

  XmlSerializer serializer = new XmlSerializer(typeof(Customer));

  reader.ReadStartElement("customers");

 

  while(!reader.EOF && reader.LocalName == "Customer")

  {

     Customer customer = (Customer)serializer.Deserialize(reader);

     yield return customer;

  }

 

  reader.ReadEndElement();

}

​5. Finally, the client and the service, both have to be configured to
use an "streamed" as transfer mode. Otherwise, all the messages will be
buffered in memory.

\<basicHttpBinding\>

   \<binding name="basic" transferMode="StreamedResponse"
maxReceivedMessageSize="10000000"\>

      \<security mode="None"\>\</security\>

   \</binding\>

\</basicHttpBinding\>

The "maxReceivedMessageSize" setting is also important because it limits
the amount of data that the service can return. You should adjust it to
a value according to the amount of data you might return in the service
implementation.

You can download the complete sample from this
[location](/images/legacy/StreamingObjects.zip).

