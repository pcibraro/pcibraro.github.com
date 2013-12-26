---
layout: post
title: "Implementing resource oriented controllers in ASP.NET MVC"
date: 2012-01-25
comments: true
categories: .NET
---

One common problem with the naming convention and default routing
mechanism in ASP.NET MVC is that we tend to group actions in a
controller for sharing an URL space.  This basically leads to complex
controllers with a lot of unrelated methods that break the SOLID
principles.  Too many responsibilities in a simple class affects
maintainability in the long run and causes secondary effects that
complicates unit testing. For example, we usually end up with class
constructors that receives too many dependencies as it is discussed
[here in
SO](http://stackoverflow.com/questions/6025482/strategy-to-refactor-when-too-many-dependencies-injected-into-service-or-control).

A service façade or a service locator are not the solution for this
problem either. I personally think the solution here is to use
controllers with fewer responsibilities and define the right routing in
case you want to share the URL space. There are some [discussions
around](http://lostechies.com/chadmyers/2009/06/18/going-controller-less-in-mvc-the-way-fowler-meant-it/)
the idea of using what was called controller-less actions, in which a
controller only performs a single thing. Without going to that extreme,
we can use a more resource oriented approach for assigning
responsibilities to a controller.  A resource in http is uniquely
identified by an URI  and can be manipulated through an unified
interface with http verbs. Although some verbs do not make much sense
when working with a browser like “delete” or “put”, we can replace those
with “post”.

Implementing a delete action with an http get is definitely wrong, as an
http get should be
[idempotent](http://en.wikipedia.org/wiki/Idempotence).

Let’s start an example by defining a simple scenario for managing a set
of customers. We can initially define two resources, Customers for
performing actions in the collection, and Customers/{id} for performing
actions in a single customer. Although they both share the same URL
space “Customers”, it does not mean we need to implement all the
functionality in a single controller called “Customers”. We can still
have a “CustomersController” for the collection, and a
“CustomerController” for the individual customers. We can use routing
for share the same URL and still forward the requests to the right
controller.

We can define the following operations in the “CustomersController”,

-   Add (GET Customers/Add): Retrieves the Html form representation for
    creating a new customer
-   Add (POST Customers/Add): Receives a representation (encoded in the
    form) for creating a new customer in the collection
-   Delete (POST Customers/{id}/Delete: Removes a customer from the
    collection
-   Index (GET Customers): Retrieves a Html view representing a list of
    customers

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
public class CustomersController : Controller
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
{
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    ICustomerRepository repository;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    public CustomersController()
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        : this(new CustomerRepository())
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    public CustomersController(ICustomerRepository repository)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        this.repository = repository;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    public ActionResult Index(string filter = null)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        IEnumerable<Customer> customers = null;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        if (!string.IsNullOrEmpty(filter))
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
            customers = this.repository.GetAll()
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
                .Where(c => c.FirstName.Contains(filter) || c.LastName.Contains(filter));
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        else
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
            customers = this.repository.GetAll();
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        return View(customers);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    [HttpGet]
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    public ActionResult Add()
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        return View();
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    [HttpPost]
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    public ActionResult Add(Customer customer)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        if(ModelState.IsValid)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
            this.repository.Add(customer);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        return RedirectToAction("Index");
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    [HttpPost]
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    public ActionResult Delete(int id)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        this.repository.Delete(id);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        if (Request.IsAjaxRequest())
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
            return new HttpStatusCodeResult((int)HttpStatusCode.OK);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        return RedirectToAction("Index");
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
}
~~~~

The “CustomerController” can have the following operations:

-   Get (GET Customers/{id}): Retrieves an Html form representing an
    specific customer
-   Update (POST Customers/{id}): Receives a representation (encoded in
    the form) for updating the customer

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
public class CustomerController : Controller
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
{
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    ICustomerRepository repository;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    public CustomerController()
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        : this(new CustomerRepository())
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    public CustomerController(ICustomerRepository repository)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        this.repository = repository;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    [HttpGet]
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    public ActionResult Get(int id)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        var customer = this.repository.GetAll()
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
            .Where(c => c.Id == id)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
            .FirstOrDefault();
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        if (customer == null)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
            return new HttpStatusCodeResult((int)HttpStatusCode.NotFound);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        return View(customer);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    [HttpPost]
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    public ActionResult Update(Customer customer)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        if (ModelState.IsValid)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
            this.repository.Update(customer);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        return RedirectToAction("Index", "Customers");
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
}
~~~~

This is how the routing table looks like,

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
public static void RegisterRoutes(RouteCollection routes)
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
{
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    routes.IgnoreRoute("{resource}.axd/{*pathInfo}");
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    //This map is required so the Add segment is not used as {id}
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    routes.MapRoute(
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
      name: "CustomerAdd",
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
      url: "Customers/Add",
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
      defaults: new { controller = "Customers", action = "Add" }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
      );
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    routes.MapRoute(
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
       name: "CustomerGet",
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
       url: "Customers/{id}",
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
       defaults: new { controller = "Customer", action = "Get" },
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
       constraints: new { httpMethod = new HttpMethodConstraint("GET") }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
       );
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
    routes.MapRoute(
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        name: "CustomerUpdate",
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        url: "Customers/{id}",
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        defaults: new { controller = "Customer", action = "Update" },
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        constraints: new { httpMethod = new HttpMethodConstraint("POST") }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        );
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    routes.MapRoute(
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        name: "MVC Default",
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
        url: "{controller}/{action}/{id}",
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
        defaults: new { controller = "Home", action = "Index", id = UrlParameter.Optional }
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
    );
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
 
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
   
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
}
~~~~

You can override the default route for “Customers/{id}” for getting or
updating an specific customer by using an http method constraint. I also
added the first  route for adding a new customer so the “Add” segment is
not used as the “{id}” wildcard.

As you could see, we could split all the responsibilities in two
controllers, which look simpler at first glance. In conclusion, you
don’t need to assume the same URL means the same controller.

The “delete” action is implemented as an http post. This can be sent
from the browser by using an Ajax call or a Http form. For example, the
following code shows how to do that using a JQuery Ajax call.

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
@Html.ActionLink("Delete", "Delete", new { id = item.Id }, new { @class = "delete" })
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
<script type="text/javascript" language="javascript">   1:  
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
   2:     $(function () {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
   3:         $('.delete').click(function () {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
   4:             var that = $(this);
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
   5:             var url = that.attr('href');
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
   6:  
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
   7:             $.post(url, function () {
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
   8:                 alert('delete called');
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
   9:             });
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
  10:  
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
  11:             return false;
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: rgb(244, 244, 244);"}
  12:         });
~~~~

~~~~ {style="margin: 0em; padding: 0px; width: 100%; text-align: left; color: black; line-height: 12pt; overflow: visible; font-family: "Courier New", courier, monospace; font-size: 8pt; direction: ltr; background-color: white;"}
  13:     });
~~~~

\</script\>

The code is available for download from
[here](/images/legacy/MVCResources.zip)

