---
layout: post
title: "OData to the rescue. Exposing the eventlog as a data feed"
date: 2010-03-15
comments: true
categories: .NET
---

In one of the project where I was working one, we used the [Microsoft
Enterprise Library Exception Application
Block](http://msdn.microsoft.com/en-us/library/cc309212.aspx)
integration with WCF for logging all the technical issues on the
services/backend in Windows Event Log. This application block worked
like a charm, all the errors were correctly logged on the Event Log
without even needing to modify the service code. However, we also needed
to provide a quick way to expose all those events to the different
system users so they could get access to all the them remotely. In just
a couple of minutes I came up with a simple solution based on ADO.NET
Data Services. ADO.NET data services is very powerful in this sense, you
only need to provide a IQueryable implementation, and that’s all. You
get a RESTful service with rich query support for free.

In this sample, I used Linq to Objects to get the latest entries from
the Event Log, and I also filter the entries by the category used by the
Application Block to avoid loading unnecessary entries in memory.

public class LogDataSource \
    { \
        string source;

        public LogDataSource(string source) \
        { \
            this.source = source; \
        }

        public LogDataSource() \
        { \
        }

        public IQueryable\<LogEntry\> LogEntries \
        { \
            get { return GetEntries().AsQueryable().OrderBy(e =\>
e.TimeGenerated); } \
        }

        private IEnumerable\<LogEntry\> GetEntries() \
        { \
            var applicationLog =
System.Diagnostics.EventLog.GetEventLogs().Where(e =\> e.Log ==
"Application") \
                .FirstOrDefault();

            var entries = new List\<LogEntry\>();

            if (applicationLog != null) \
            { \
                foreach (EventLogEntry entry in applicationLog.Entries)
\
                { \
                    if (source == null || entry.Source.Equals(source,
StringComparison.InvariantCultureIgnoreCase)) \
                    { \
                        entries.Add(new LogEntry \
                        { \
                            Category = entry.Category, \
                            EventID = entry.InstanceId, \
                            Message = entry.Message, \
                            TimeGenerated = entry.TimeGenerated, \
                            Source = entry.Source, \
                        }); \
                    } \
                } \
            }

            return entries.OrderByDescending(e =\> e.TimeGenerated) \
                        .Take(200); \
        } \
    }

LogEntry is class I created for this service to expose an Event Log
Entry.

    [EntityPropertyMappingAttribute("Source", \
        SyndicationItemProperty.Title, \
        SyndicationTextContentKind.Plaintext, true)] \
    [EntityPropertyMapping("Message", \
        SyndicationItemProperty.Summary, \
        SyndicationTextContentKind.Plaintext, true)] \
    [EntityPropertyMapping("TimeGenerated", \
        SyndicationItemProperty.Updated, \
        SyndicationTextContentKind.Plaintext, true)] \
    [DataServiceKey("EventID")] \
    public class LogEntry \
    { \
        public long EventID \
        { \
            get; \
            set; \
        }

        public string Category \
        { \
            get; \
            set; \
        }

        public string Message \
        { \
            get; \
            set; \
        }

        public DateTime TimeGenerated \
        { \
            get; \
            set; \
        }

        public string Source \
        { \
            get; \
            set; \
        }

    }

As you can see, I used the new feature [“Friendly
feeds”](http://blogs.msdn.com/phaniraj/archive/2009/03/28/ado-net-data-services-friendly-feeds-mapping-edm-types-i.aspx)
to map several properties in the entries with standard ATOM elements.
The “DataServiceKey” is only necessary because I am using the Reflection
Provider (the exposed IQueryable implementation is just Linq to Objects)
rather than the default Entity Framework Provider.

The data service implementation is also quite simple, just a couple of
lines were needed to expose the data source created previously.

public class LogDataService : DataService\<LogDataSource\> \
    { \
        public static void InitializeService(IDataServiceConfiguration
config) \
        { \
            config.SetEntitySetAccessRule("\*",
EntitySetRights.AllRead); \
        }

        protected override LogDataSource CreateDataSource() \
        { \
            string source =
ConfigurationManager.AppSettings["EventLogSource"]; \
            if (source == null) \
            { \
                throw new ApplicationException("The EventLogSource
appsetting is missing in the configuration file"); \
            }

            return new LogDataSource(source); \
        } \
    }

With this implementation in place, the final users not only get a feed
with all the latest errors in the event log, but also support for
performing queries against that data. This is one of the great things
about ADO.NET Data services.

