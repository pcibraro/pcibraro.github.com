---
layout: post
title: "Using SqlCache dependencies with the new WCF WebCache attribute (REST Starter KIT)"
date: 2008-11-03
comments: true
categories: REST
---

Let's begin from a hypothetical example that we want to publish a simple
product catalog as an ATOM feed. The product table schema in our
database was initially designed as follows,

![](/images/legacy/OrdersTable.jpg)

The first step is to enable the SQL notification for the products
database, that can be done with the following commands:

​1. Enable cache notifications for the database: aspnet\_regsql
[Credentials] -ed -d [DatabaseName] . For example, aspnet\_regsql -S
.\\SqlExpress -E -ed -d Northwind

​2. Enable cache notifications for an specific table in that database:
aspnet\_regsql [Credentials] -et -d [DatabaseName] -t [TableName] . For
example, aspnet\_regsql -S .\\SqlExpress -E -et -d Northwind  -t
Products

Once we have enabled the ASP.NET SQL notification for the table we want
to use (Products in this example), the next step is to implement the
service and configure it correctly to use the new WebCache behavior.

The contract for the service is quite simple, it receives an optional
categoryId attribute just in case we want to filter the products.

[ServiceContract]

public interface IProductCatalog

{

  [WebCache(Location = OutputCacheLocation.ServerAndClient,
SqlDependency = "myDatabase:Products", VaryByParam = "categoryId")]

  [WebGet(UriTemplate = "?category={categoryId}")]

  [OperationContract]

  Atom10FeedFormatter GetProducts(int categoryId);

}

As you can see in the code above, I defined the new WebCache behavior as
part of my operation contract. I also specified that I want to
invalidate the ASP.NET cache entries when two of the following
conditions are true,

​1. Some change is introduced in the products table (The ASP.NET Sql
Dependency)

​2. A different categoryId argument is specified in query string.

The caching preferences for this service are configured in the
application web configuration file,

\<connectionStrings\>

  \<add name="myDatabase" connectionString="Data
source=.\\SQLExpress;Initial
Catalog=Northwind;Trusted\_Connection=yes"/\>

\</connectionStrings\>

\<system.web\>

  \<compilation debug="true"\>\</compilation\>

\<caching\>

  \<sqlCacheDependency enabled="true" pollTime="1000" \>

    \<databases\>

      \<add name="myDatabase" connectionStringName="myDatabase" /\>

    \</databases\>

  \</sqlCacheDependency\>

\</caching\>

\</system.web\>

\<system.serviceModel\>

    \<serviceHostingEnvironment aspNetCompatibilityEnabled="true"/\>

\</system.serviceModel\>

The ASP.NET compatibility mode is required in order to use this new
caching mechanism in our REST service. The service implementation is
quite normal, there was no need to add some extra code to support
caching, which is something great thanks to the new WebCache behavior.

[AspNetCompatibilityRequirements(RequirementsMode =
AspNetCompatibilityRequirementsMode.Allowed)]

public class ProductCatalog : IProductCatalog

{

    public Atom10FeedFormatter GetProducts(int categoryId)

    {

        var connectionString =
ConfigurationManager.ConnectionStrings["myDatabase"].ConnectionString;

 

        var items = new List\<SyndicationItem\>();

        using (var connection = new SqlConnection(connectionString))

        {

            var command = new SqlCommand();

            if (categoryId == 0)

                command.CommandText = "SELECT \* FROM Products";

            else

            {

                command.CommandText = "SELECT \* FROM Products WHERE
CategoryId = @categoryId";

                command.Parameters.Add(new SqlParameter("@categoryId",
categoryId));

            }

 

            command.Connection = connection;

            connection.Open();

 

            using(var reader =
command.ExecuteReader(CommandBehavior.CloseConnection))

            {

                while(reader.Read())

                {

                    items.Add(new SyndicationItem()

                    {

                        Id = String.Format(CultureInfo.InvariantCulture,
"http://products/{0}", (int)reader["ProductID"]),

                        Title = new
TextSyndicationContent((string)reader["ProductName"]),

                        LastUpdatedTime = new DateTime(2008, 7, 1, 0, 0,
0, DateTimeKind.Utc),

                        Authors =

                        {

                            new SyndicationPerson()

                            {

                                Name = "cibrax"

                            }

                        },

                        Content = new
TextSyndicationContent(string.Format("Category Id {0} - Price {1}",

                            (int)reader["CategoryId"],
(decimal)reader["UnitPrice"]))

                    });

                }

            }

        }

 

        var feed = new SyndicationFeed()

        {

            Id = "http://Products",

            Title = new TextSyndicationContent("Product catalog"),

            Items = items

        };

 

        WebOperationContext.Current.OutgoingResponse.ContentType =
"application/atom+xml";

        return feed.GetAtom10Formatter();

    }

}

If you are interested in knowing more details about the WebCache
behavior implementation, my friend Jesus Rodriguez has written an
[excellent
post](http://weblogs.asp.net/gsusx/archive/2008/10/29/adding-caching-to-wcf-restful-services-using-the-rest-starter-kit.aspx)
some days ago.

