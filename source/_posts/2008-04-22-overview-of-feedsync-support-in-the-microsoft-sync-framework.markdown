---
layout: post
title: "Overview of FeedSync support in the Microsoft Sync Framework"
date: 2008-04-22
comments: true
categories: FeedSync
---

### 

### What is FeedSync ?

If you have not heard about "FeedSync" yet, it is the official name of
an open and interoperable standard for representing synchronization
metadata in a XML format. This specification was formerly called by
Microsoft as "SSE" (Simple Sharing Extensions), and its name was changed
to "FeedSync" prior to the first release version.

The fact that the "FeedSync" model is completely based on XML and it is
quite simple to represent, makes this specification a good candidate to
guarantee interoperability with other platforms.

The complete specification is based on two fundamental aspects,

​1. A way to represent the "FeedSync" model as XML extensions that can
be easily attached to any existing document. The most common use
nowadays is to include them as part of a syndication feed like RSS or
ATOM to keep synchronization metadata about each available item. Many
people have started proving parsers for this specification as part of
their Syndication libraries, some examples are [Simple Sharing
Extensions For .NET](http://www.codeplex.com/sse), [Rome for
Java](https://rome.dev.java.net/), or [Argotic
(.NET)](http://www.codeplex.com/Argotic)

Let's take a quick look at the metadata attached to some items in a RSS
feed.

\<feed xmlns:sx='http://feedsync.org/2007/feedsync'\>

  \<title\>Hello World\</title\>

  \<link\>http://weblogs.asp.net/cibrax\</link\>

  \<description\>this is my feed\</description\>

  \<sx:sharing since='11-05-2007T19:33:27Z'
until='14-05-2007T19:33:27Z'\>

    \<sx:related link='http://kzu/full' type='complete' /\>

  \</sx:sharing\>

  \<item\>

   \<title\>Foo Title\</title\>

   \<description\>Foo Description\</description\>

   \<Foo Title='Foo' /\>

   \<sx:sync id='d92d64f5-c006-4086-a35a-c5195448d3ad' updates='2'
deleted='false' noconflicts='false'\>

     \<sx:history sequence='2' when='18-05-2007T19:33:27Z' by='Cibrax'
/\>

     \<sx:history sequence='1' when='14-05-2007T19:33:27Z' by='JohnFoo'
/\>

    \</sx:sync\>  \</item\>

 

\</feed\>

First of all, it adds optional metadata at feed level through the
"sharing" element to include aggregated information about related feeds
or sources. (The since and until attributes in that element represents
the lower and upper bounds of the items contained within the feed).

Secondly, it includes synchronization metadata at item level with the
"sync" element. This element contains the following information:

​a. An identifier to represent the item (Id attribute), this identifier
must be unique across all the synchronization peers.

​b. An history of updates that includes the person that made a change to
the item, and the date/time of that change. As you can see, this
information is quite simple, it does not say anything about what were
the changes made to the item (We will see later that this information is
actually not important for the merging algorithm).

​c. Flags to indicate if the item was deleted or contains conflicts.
(The same item version or sequence number was modified in one or more
replicas at the same time).

The rest of the information not mentioned here is part of the RSS
specification itself and it does not have anything to do with
"FeedSync".

​2. An algorithm that use the synchronization metadata to merge the
items across several peers. For instance, to perform a bidirectional
synchronization of local information between two peers. The algorithm
also uses the metadata to detect conflicts at the moment of doing the
merge operation.

For example, if we have the following information in two different
replicas

+--------------------------------------+--------------------------------------+
| REPLICA A                            | REPLICA B                            |
+--------------------------------------+--------------------------------------+
|   ItemA-Sequence1                    | ItemC-Sequence1                      |
+--------------------------------------+--------------------------------------+
|   ItemB-Sequence1                    |                                      |
+--------------------------------------+--------------------------------------+

After the first synchronization, the resulting information will be:

+--------------------------------------+--------------------------------------+
| REPLICA A                            | REPLICA B                            |
+--------------------------------------+--------------------------------------+
|   ItemA-Sequence1                    | ItemC-Sequence1                      |
+--------------------------------------+--------------------------------------+
|   ItemB-Sequence1                    |  ItemA-Sequence1                     |
+--------------------------------------+--------------------------------------+
|   ItemC-Sequence1                    |  ItemB-Sequence1                     |
+--------------------------------------+--------------------------------------+

Now, Let's say the replica B changes the item A and B( The sequence or
version is incremented)

+--------------------------------------+--------------------------------------+
| REPLICA A                            | REPLICA B                            |
+--------------------------------------+--------------------------------------+
|   ItemA-Sequence1                    | ItemC-Sequence1                      |
+--------------------------------------+--------------------------------------+
|   ItemB-Sequence1                    |  ItemA-Sequence2                     |
+--------------------------------------+--------------------------------------+
|   ItemC-Sequence1                    |  ItemB-Sequence2                     |
+--------------------------------------+--------------------------------------+

If they synchronize again, the resulting information will be:

+--------------------------------------+--------------------------------------+
| REPLICA A                            | REPLICA B                            |
+--------------------------------------+--------------------------------------+
|   ItemA-Sequence2                    | ItemC-Sequence1                      |
+--------------------------------------+--------------------------------------+
|   ItemB-Sequence2                    | ItemA-Sequence2                      |
+--------------------------------------+--------------------------------------+
|   ItemC-Sequence1                    | ItemB-Sequence2                      |
+--------------------------------------+--------------------------------------+

Now, if both replicas change the same item before synchronizing, for
instance ItemC. At the moment of synchronizing the information, they
will have the same sequence for ItemC, and therefore a conflict will
exist. (The algorithm to detect conflicts is more complex than this, I
am just giving an example)

+--------------------------------------+--------------------------------------+
| REPLICA A                            | REPLICA B                            |
+--------------------------------------+--------------------------------------+
|   ItemA-Sequence2                    |  ItemC-Sequence2                     |
+--------------------------------------+--------------------------------------+
|   ItemB-Sequence2                    | ItemA-Sequence2                      |
+--------------------------------------+--------------------------------------+
|   ItemC-Sequence2                    | ItemB-Sequence2                      |
+--------------------------------------+--------------------------------------+

Having these two important aspects, a developer wanting to synchronize
information with other applications only has to represent that
information as a FeedSync feed (A normal feed that includes "FeedSync"
metadata) and expose it somewhere. Another application can later on
combine that feed with its own feed using the merging algorithm, which
will result in a full biredirectional synchronization between those two
application.

As you can see, both aspects complement each other, an bidirectional
synchronization can not be done with just one of them.

### 

### How to expose your information in a FeedSync feed with the Sync Framework

​1. The first thing you have to do is to determine which information you
will want to expose in the final feed. Let's say that you have several
copies of a database with customer information spread across several
locations (Different company's branches perhaps), and you want to keep
all those copies synchronized. You will have to expose that information
as part of a feed, so people in other locations can synchronize their
data against that feed (This example assumes that one central feed is
used, but you can actually have multiple feeds that synchronize each
other having a kind of tree structure, this is one of the goodness about
FeedSync, no central replica is required).

If you want to represent the customer's full name, address and phone
number as an entry in a RSS feed, the entry will look as follow:

\<item\>

  \<title\>John Doe\</title\>

  \<description\>John Doe's Information\</description\>

  \<Customer\>

    \<FullName\>John Doe\</FullName\>

    \<Address\>Test Adress\</Address\>

    \<PhoneNumber\>xxx-xxxx-xxxx\</PhoneNumber\>   \</Customer\>

\</item\>

This mapping between the real data and a XML payload (The RSS's entry
data) can be done in the Sync Framework by having a concrete
implementation of "FeedItemConverter" class. The implementation of this
class will know how to map custom data to a XML payload and vice versa.

public abstract class FeedItemConverter

{

  protected FeedItemConverter();

  public abstract string ConvertItemDataToXmlText(object itemData);
//Method to convert the actual item data into XML

  public abstract object ConvertXmlToItemData(string itemXml); //Method
to convert the XML data into the item data

}

I personally do not like much the signature of those methods, an string
could be anything (itemXml), not just XML and that can not be enforced
by the current API design. I would prefer to have something like this:

public abstract class FeedItemConverter

{

  protected FeedItemConverter();

  public abstract void WriteItemDataToXml(object itemData, XmlWriter
xmlWriter); //Method to convert the actual item data into XML

  public abstract object ReadItemDataFromXml(XmlReader xmlReader);
//Method to convert the XML data into the item data

}

In the sample above, you should implement a "CustomerFeedItemConverter"
to convert a customer instance into XML the xml representation and vice
versa.

​2. Secondly, you have to provide a mapping between your internal
identifiers and the identifiers used in the synchronization metadata.
From the MSDN documentation "

*"A FeedIdConverter can convert replica IDs and item IDs from the
flexible-length format of the provider to strings, and vice versa. Also,
the ID converter must be able to generate a replica ID for an anonymous
change. The FeedSync history for a change contains three potential
attributes: sequence, when, and by. The by attribute represents the
replica that made the change, but it is not required by the FeedSync
schema and so might be absent. If a change does not include a by value,
a replica ID must be generated for the change by combining the sequence
and when values"*

public abstract class FeedIdConverter

{

  protected FeedIdConverter();

  public abstract SyncIdFormatGroup IdFormats { get; }

  public abstract string ConvertItemIdToString(SyncId itemId);

  public abstract string ConvertReplicaIdToString(SyncId replicaId);

  public abstract SyncId ConvertStringToItemId(string value);

  public abstract SyncId ConvertStringToReplicaId(string value);

  public abstract SyncId GenerateAnonymousReplicaId(string when, uint
sequence);

}

Since the implementation of this provider will be the same most of the
times, I think it would be a good idea to provide a default
implementation of this provider as part of the Sync framework (It could
provide extensibility points through virtual methods).

​3. Once you have the item and id converters, they should be enough to
start consuming or publishing FeedSync feeds with your custom
information. The framework comes with two classes for that purpose,
FeedConsumer and FeedProducer.

public class FeedProducer

{

  public FeedProducer(KnowledgeSyncProvider storeProvider,
FeedIdConverter idConverter, FeedItemConverter itemConverter);

public FeedIdConverter IdConverter { get; set; }

  public EndpointState IncrementalFeedBaseline { get; set; }

  public FeedItemConverter ItemConverter { get; set; }

  public KnowledgeSyncProvider StoreProvider { get; set; }

  public void ProduceFeed(Stream feedStream);

}

public class FeedConsumer

{

  public FeedConsumer(KnowledgeSyncProvider storeProvider,
FeedIdConverter idConverter, FeedItemConverter itemConverter);

  public FeedItemConverter FeedItemConverter { get; set; }

  public FeedIdConverter IdConverter { get; set; }

  public KnowledgeSyncProvider StoreProvider { get; set; }

  public EndpointState ConsumeFeed(Stream feedStream);

}

As you can see, those classes basically receive in the constructor the
converters along with a sync repository (KnowledgeSyncProvider), the one
that knows specific details about the underline data source. (for this
example, it should know about the customers data in the database). All
the magic is done by the methods ProduceFeed and ConsumeFeed, which
reads/saves a feed from/to an existing stream. Again, I have three ideas
to improve these classes,

​a. They are tied to Rss/Atom, it should be extensible enough to support
any document payload.

​b. The underline stream must contain an existing Rss/Atom feed in order
to save/read items on it. This is how these classes determine the format
of the items. It would be great to have a way to specify a formatter for
the items at the moment to save/read them. A approach very similar to
the one taken by the Syndication API in WCF.

​c. I would like to have a better integration with the WCF syndication
API. I mean, a direct way to expose a REST endpoint for a feedsync feed,
the consumer/producer classes could accept a SyndicationFeedFormatter as
input to generate/consume the feed.

Once the data have been read from the underline stream into a sync
repository, it can be used with the
"[SyncOrchestrator](http://msdn2.microsoft.com/en-us/library/microsoft.synchronization.syncorchestrator(SQL.100).aspx)"
(Part of the Sync Framework) to initiate a new synchronization session.

### Complete Example "Browser Favorites"

A complete example that illustrates all the steps to implement a custom
sync provider and expose its data as a FeedSync feed can be found here,
[http://blogs.msdn.com/sync/archive/2008/04/08/synchronization-of-browser-favorites-using-feedsync-and-the-microsoft-sync-framework.aspx](http://blogs.msdn.com/sync/archive/2008/04/08/synchronization-of-browser-favorites-using-feedsync-and-the-microsoft-sync-framework.aspx)

It basically shows how to synchronize the browser favorites information
using FeedSync as the underline synchronization protocol.

### Do you want to synchronize FeedSync data with clients behind a firewall and NAT ?

Now, that is possible thanks to WCF and the Relay service provided in
[Biztalk services](http://labs.biztalk.net/). Imagine a WCF service that
exposes feedsync items through the Relay service, if we use a WCF duplex
contract for that service, it will be able to start a bidirectional
synchronization with any client behind NAT and Firewalls. I will try to
show this functionality any time soon.

In the meantime, If you want to know more about WCF duplex callbacks
through NAT and Firewalls, I recommend [this
post](http://blogs.thinktecture.com/cweyer/archive/2007/04/27/414819.aspx)
written by [Christian Weyer](http://blogs.thinktecture.com/cweyer).

