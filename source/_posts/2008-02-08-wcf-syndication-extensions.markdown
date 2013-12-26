---
layout: post
title: "WCF - Syndication Extensions"
date: 2008-02-08
comments: true
categories: WCF
---

Syndication support was one of the nice features introduced in the
latest WCF drop (3.5).  At first glance, I can say the APIs are great,
allowing you to write a syndication feed in matter of minutes without
worrying much of some underline implementation details pertinent to RSS
or ATOM.

Another aspect that makes these APIs very powerful is the extensibility
support through what the WCF team called "Syndication Extensions". As
its name clearly states, a syndication extension is a mechanism to
extend a normal RSS or ATOM feed with custom elements or attributes,
which is a good thing to implement many of the specifications available
nowadays, for instance, FeedSync or GeoRss to name a few.

WCF particularly supports two kinds of extensions, loose-typed and typed
extensions.

In the first one, the extensions are accessed in a loosely-typed way
through the AttributeExtensions (for xml attribute extensions) and
ElementExtensions (for xml element extensions) properties in the
syndication item. The only requirement for this type of extension is
that the referenced type should be a valid data contract or xml
serializable type.

On the other hand, a typed extension  requires a bit more of code since
the SyndicationItem class must be extended to parse the corresponding
attribute or element extensions. A good thing about this type of
extension is that the developer gets access to a Syndication Item
instance with typed support for each one of the extensions, for
instance, the developer can have a typed class FeedSyncItem, which
essentially represents a SyndicationItem with FeedSync metadata to
support synchronization through RSS or ATOM.

The base class SyndicationItem for representing  simple items (which
internally can be ATOM or RSS) contains four methods that can be
overridden by the developer to support the corresponding extensions,

TryParseElement and WriteElementExtension to read and write element
extensions.

TryParseAttribute and WriteAttributeExtension to read and write
attribute extensions.

Let's discuss both approaches with a simple implementation of FeedSync,
(By the way, FeedSync is the final name for what was formerly called
SSE, I already discussed SSE in previous posts).  The latest FeedSync
specification is available at this URL,
[http://dev.live.com/feedsync/spec/](http://dev.live.com/feedsync/spec/)

A simple XML serializable class representing the FeedSync metadata
should looks like this:

[XmlRoot(ElementName = "sync", Namespace =
"http://feedsync.org/2007/feedsync", IsNullable = false)]

public class Sync

{

  [XmlElement("history")]

  public List\<History\> history { get; set; }

  [XmlAttribute(AttributeName = "id")]

  public string Id { get; set; }

  [XmlAttribute(AttributeName = "updates")]

  public int Updates { get; set; }

  [XmlAttribute(AttributeName = "deleted")]

  public bool Deleted { get; set; }

  [XmlAttribute(AttributeName = "noconflicts")]

  public bool NoConflicts { get; set; }

}

In the loose-typed approach, our FeedSync element can be accessed
through the following code:

// .. Read feed

XmlSerializer serializer = new XmlSerializer(typeof(Sync));

foreach (SyndicationItem item in feed.Items)

{

  foreach (SyndicationElementExtension extension in
item.ElementExtensions)

  {

    if (extension.OuterName == "sync" && extension.OuterNamespace == 
"http://feedsync.org/2007/feedsync")

    {

      Sync sync = extension.GetObject\<Sync\>(serializer);

    }

  }

}

Quite simple, we have to look up the right extension with some help of
the OuterName and OuterNamespace properties, and then, deserialize the
extension using the right formatter (For the example above, a
XmlSerializer).

The second approach requires more code on our side, two new classes have
to be created, one for the feedsync items and another one for the feed.

public class FeedSyncSyndicationItem : SyndicationItem

{

  public FeedSyncSyndicationItem()

  : base()

  {

  }

  private Sync sync;

  public Sync Sync

  {

    get { return sync; }

  }

  protected override bool TryParseElement(System.Xml.XmlReader reader,
string version)

  {

    if (reader.LocalName == "sync" && reader.NamespaceURI ==
"http://feedsync.org/2007/feedsync")

    {

      XmlSerializer serializer = new XmlSerializer(typeof(Sync));

      this.sync = (Sync)serializer.Deserialize(reader);

      return true;

    }

    return base.TryParseElement(reader, version);

  }

}

In this case, I only implemented the TryParseElement since the FeedSync
metadata is just an element (Other extension may be attributes)

The feed implementation is only needed to specify the concrete class for
the items,

public class FeedSyncSyndicationFeed : SyndicationFeed

{

  public FeedSyncSyndicationFeed()

  : base()

  {

  }

  protected override SyndicationItem CreateItem()

  {

  return new FeedSyncSyndicationItem();

}

}

Finally, we can create an instance of our typed feed (or item) using the
static method .Load\<\> available in the SyndicationItem and
SyndicationFeed classes.

FeedSyncSyndicationFeed feed =
SyndicationFeed.Load\<FeedSyncSyndicationFeed\>(reader);

The reader in the example above points to an xml stream containing a
valid feed with FeedSync metadata.

