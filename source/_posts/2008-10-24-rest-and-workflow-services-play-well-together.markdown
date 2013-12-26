---
layout: post
title: "REST and Workflow services play well together"
date: 2008-10-24
comments: true
categories: REST
---

REST is not just about CRUD interfaces for your services, a complete
long-running workflow can be modeled through the basic verbs as well. As
my friend [Jesus
mentioned](http://weblogs.asp.net/gsusx/archive/2008/10/03/new-rest-quot-must-read-quot-papers.aspx),
the article "[How to get a cup of
coffee](http://www.infoq.com/articles/webber-rest-workflow)" represents
an excellent source about this topic.

The example I will show here is based on that article, during the course
of this post I will try to illustrate all the steps required to
implement a workflow as the one mentioned there using WF services.
Unfortunately, no examples exist (or at least I could not find any)
about how to configure the WCF web model to use the new WF services, so
I decided to create a new one from scratch.

This example only illustrates the workflow from the customer point of
view, he can place an order, pay it and wait for his drink. He can also
modify the order in the middle before it is paid.

 ![](/images/legacy/OrderWorkflow.jpg)

The interface for our REST service will looks like this:

[ServiceContract]

public interface IOrderService

{

  [OperationContract]

  [WebInvoke(Method="POST", UriTemplate="order")]

  Order PlaceOrder(Order order);

 

  [OperationContract]

  [WebInvoke(Method = "PUT", UriTemplate = "order/{id}")]

  Order UpdateOrder(string id, Order order);

 

  [OperationContract]

  [WebInvoke(Method="PUT", UriTemplate="payment/order/{id}")]

  void PayOrder(string id, Payment payment);

}

Only three operations available, each one maps exactly with one
ReceiveActivity in the workflow. For instance, the receive activity
OnOrderPlaced waits for a client call to the PlaceOrder operation and
then moves the workflow to the next state, which is "OrderPlaced" in
this case. While the workflow is in the state "OrderPlaced", only the
operations UpdateOrder and PayOrder can be executed by the client (If he
try to execute PlaceOrder again, it will receive a 404 http error, "Not
found").

![](/images/legacy/OnOrderPlaced.jpg)

The OnOrderPlacedCode is an simple code activity that contains the
actual operation implementation. I used a code activity for a sake of
simplicity, a custom activity for creating the order could also be used
there.

The code for this activity is quite simple, most of it is hard-coded,

private void OnOrderPlacedCode\_ExecuteCode(object sender, EventArgs e)

{

  currentOrder = new Order();

  currentOrder.OrderId = Guid.NewGuid().ToString();

  currentOrder.Cost = (receivedOrder.Drink == "latte") ? 6 : 10;

  currentOrder.Drink = receivedOrder.Drink;

  currentOrder.Next = new Next[] {

      new Next

      {

         Rel = "http://starbucks.example.org/payment",

         Uri = "http://localhost:8000/payment/order/" +
currentOrder.OrderId.ToString(),

      },

      new Next

      {

         Rel = "http://starbucks.example/order/update",

         Uri = "http://localhost:8000/order/" +
currentOrder.OrderId.ToString()

      }};

 

  //Persist the order in some place.....

 

  WebOperationContext.Current.OutgoingResponse.StatusCode =
System.Net.HttpStatusCode.Created;

}

currentOrder contains the instance of the order that will be returned by
the service operation whereas receivedOrder is the order sent by the
client application. This was map to the operation signature in the
receive activity,

 ![](/images/legacy/ReceiveOrderSettings.jpg)

Once the workflow is ready to be used, all we need is to configure the
service host so the client application can start consuming the
operations. At this point, we will face three issues:

​1. If you want to host a WF service, a WorkflowServiceHost class has to
be used in the host program. Therefore, we will not have all the
automatic infrastructure configuration provided by WebServiceHost, all
the configuration must be done manually. It would be great to have here
a combination of both service hosts.

​2. There is not a context binding for WebHttpBinding, only
BasicHttpContextBinding, NetTcpContextBinding and WsHttpContextBinding
are supported. We will have to use a custom binding.

​3. A cookie must be used to transfer the context information between
the client and the service, it would much better to have support for
http headers here. According to this [post written by
Jesus](http://weblogs.asp.net/gsusx/archive/2008/06/27/combining-durable-messaging-and-workflow-services-building-a-custom-context-channel.aspx),
this should not be complex to do.

\<system.serviceModel\>

  \<serviceHostingEnvironment aspNetCompatibilityEnabled="true" /\>

  \<services\>

    \<service name="RestWorkflows.OrderWorkflow"\>

     \<endpoint address="" behaviorConfiguration="MyServiceBehavior"
binding="customBinding" bindingConfiguration="myServiceBinding"
contract="RestWorkflows.IOrderService" /\>

  \</service\>

\</services\>

\<bindings\>

\<customBinding\>

  \<binding name="myServiceBinding"\>

  \<webMessageEncoding\>\</webMessageEncoding\>

  \<context contextExchangeMechanism="HttpCookie"
protectionLevel="None"/\>

  \<httpTransport manualAddressing="true"/\>

  \</binding\>

\</customBinding\>

\</bindings\>

\<behaviors\>

\<endpointBehaviors\>

  \<behavior name="MyServiceBehavior"\>

    \<webHttp /\>

  \</behavior\>

\</endpointBehaviors\>

\</behaviors\>

\</system.serviceModel\>

The code for the client application looks also quite simple, you only
have to remember to include the cookie with the context between calls,
otherwise WF will try to create a new instance of the workflow resulting
in a bad request error.

//Lets create a new order

Order order = new Order { Drink = "latte" };

 

HttpWebRequest createOrderRequest = CreateRequest(new
Uri("http://localhost:8000/order"), "POST", null, order);

string context =
GetContext((HttpWebResponse)createOrderRequest.GetResponse());

order = Execute\<Order\>(createOrderRequest.GetResponse());

 

Console.WriteLine("Order Cost {0}", order.Cost);

foreach (Next next in order.Next)

{

  Console.WriteLine("Possible next step {0}", next.Uri);

}

 

//Lets update the existing order....

order.Drink = "cappuchino";

 

Uri updateOrderUri = new Uri(order.Next.Where(n =\> n.Rel ==
"http://starbucks.example/order/update").Single().Uri);

HttpWebRequest updateOrderRequest = CreateRequest(updateOrderUri, "PUT",
context, order);

Order updatedOrder = Execute\<Order\>(updateOrderRequest.GetResponse());

 

Console.WriteLine("Order Cost {0}", updatedOrder.Cost);

foreach (Next next in updatedOrder.Next)

{

  Console.WriteLine("Possible next step {0}", next.Uri);

}

 

//Lets have our drink...

Payment payment = new Payment { Name = "John Doe", CardNumber =
"1234567", Expires = "06/08", Amount =
updatedOrder.Cost.GetValueOrDefault() };

 

Uri payOrderUri = new Uri(order.Next.Where(n =\> n.Rel ==
"http://starbucks.example.org/payment").Single().Uri);

HttpWebRequest payOrderRequest = CreateRequest(payOrderUri, "PUT",
context, payment);

 

int statusCode = Execute(payOrderRequest.GetResponse());

 

if (statusCode == 201)

  Console.WriteLine("Here is your drink!!!");

else

  Console.WriteLine("Sorry, there was some error while trying to process
the payment");

As you can see in the code above, I used some helper methods to execute
the operations and retrieve the response/context information. These
methods only represent a few lines of code,

static HttpWebRequest CreateRequest(Uri address, string method, string
context, object contract)

{

  HttpWebRequest webRequest =
(HttpWebRequest)WebRequest.Create(address);

  webRequest.ContentType = "application/xml";

  webRequest.Timeout = 30000;

  webRequest.Method = method;

  webRequest.CookieContainer = new CookieContainer();

 

  if (context != null)

  {

    Cookie cookie = new Cookie("WscContext", context,
address.PathAndQuery, address.Authority);

    webRequest.CookieContainer.Add(cookie);

  }

 

  DataContractSerializer serializer = new
DataContractSerializer(contract.GetType());

  using (Stream stream = webRequest.GetRequestStream())

  {

    serializer.WriteObject(stream, contract);

    stream.Flush();

  }

 

  return webRequest;

}

 

static string GetContext(HttpWebResponse response)

{

  if (response.Cookies["WscContext"] != null)

  {

    return response.Cookies["WscContext"].Value;

  }

 

  return null;

}

 

static T Execute\<T\>(WebResponse response)

{

  DataContractSerializer serializer = new
DataContractSerializer(typeof(T));

  using (Stream stream = response.GetResponseStream())

  {

    return (T)serializer.ReadObject(stream);   

  }

}

The complete solution is available to download from
[here](/images/legacy/RestWorkflows.zip).

