---
layout: post
title: "We have IQueryable, so why bother with a repository"
date: 2011-05-05
comments: true
categories: .NET
---

The repository pattern became popular a couple of years ago by the hand
of Eric Evans with the DDD (Domain Driven Design) movement and Martin
Fowler with his catalog of Enterprise Application Patterns.

The idea of having persistence ignorant entities inferred from the
business domain and a repository as a simple intermediary for
abstracting the underline persistence storage details had a great
acceptance in the development community. No matter if you decided to
stick to DDD or not, the repository pattern brought an incredible value
to the table by decoupling your business domain code from details of the
database access code (or any other underline storage).  This was
particularly important for unit testing your business domain classes. I
personally don’t think in a case of completely replacing  the underline
storage, from a database to a http service or a file for example,
because those scenarios are not very common and usually involves more
changes than just switch the repository layer. 

The idea of using an abstraction layer at that level was not new at all,
but it was mutating with different names and shapes over the time. I am
pretty sure many of you still remember the data access layer in the old
times of COM when the N Tier architecture was a popular idea pushed by
Microsoft. Same idea but different name, design and implementation
details.

However, a time after the repository became popular, a great game
changer was introduced in the .NET ecosystem with something with all
know as Linq.

Linq already provides an abstraction layer with common query
capabilities on top of any data source so why bother with yet another
abstraction layer like a repository. It results that the entry point to
the query system in Linq, IQueryable, is not enough most of the times
and only provides a read-only view with query support over the data
source. We also need operations to persist changes in the repository,
and that’s not something we get out of the box with IQueryable.

Microsoft on his side has provided some popular Linq implementations as
part of the .NET framework to address particular scenarios or common
problems like object relation mapping for databases with Entity
Framework, xml management with Linq to XML or data management over http
with WCF data services to name a few.

An initial problem with some of these implementations is that they
didn’t make certain abstractions implicit making really hard to
replicate their behavior with mocks or stubs as part of an unit test. A
typical example was the the “eager loading” capability of EF in the
initial versions. It was not possible to use lazy loading for
associations, and the Include method for loading those was not something
you could easily abstract as part of the repository. While this issue
was partially addressed with POCOs, only the latest EF code first bits
makes the approach of making an unit testable  repository [something
possible](http://blogs.clariusconsulting.net/kzu/how-to-design-a-unit-testable-domain-model-with-entity-framework-code-first/).

I agree with Daniel that a repository should only provide IQueryable for
the aggregated roots  and methods for persisting changes in the
underline store. This is how the ideal repository abstraction should
look to me,

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
public interface IRepository
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
{
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    void SaveChanges();
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    void Add<T>(T entity) where T : IEntity;
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    void Update<T>(T entity) where T : IEntity;
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    void Delete<T>(long id) where T : IEntity;
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    IQueryable<OneEntity> Entities { get; }
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
 
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
    // Other aggregate roots.
~~~~

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: #f4f4f4; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
}
~~~~

Before you say something, this is not the typical repository pattern,
yes, because it also uses the unit of work pattern to submit the
changes. To be honest, I don’t really care having the unit of work as
part of this abstraction because it makes the repository easier to test
and use too. However, there are some disadvantages of exposing
IQueryable directly in your repository.

-   You can not easily reuse queries. At that point, you might want to
    have some pre-defined extension methods for creating IQueryable
    instances with all the filters already set. If you are still not a
    big fan of using this approach, you might want to switch to an
    approach based on query specs as this one,
    [LinqSpecs](http://linqspecs.codeplex.com/). 
-   Other developers can alter the original purpose of the queries, or
    make things wrong with it. I think this is not a problem of this
    approach at all, but it mostly related to [“developer
    protection”](http://davybrion.com/blog/2009/04/educate-developers-instead-of-protecting-them/).

What about methods for filtering data that receive an expression like
this one,

~~~~ {style="border-bottom-style: none; text-align: left; padding-bottom: 0px; line-height: 12pt; border-right-style: none; background-color: white; margin: 0em; padding-left: 0px; width: 100%; padding-right: 0px; font-family: 'Courier New', courier, monospace; direction: ltr; border-top-style: none; color: black; font-size: 8pt; border-left-style: none; overflow: visible; padding-top: 0px"}
IQueryable<T> Find(Expression<Func<T,bool>> predicate); 
~~~~

Unless Microsoft provides a good support for comparing expression trees
in the .NET framework, I would stay away from that approach. I made a
mistake myself of using it in the past for a repository over WCF Data
Services. What I learned from that experience is that most of the
mocking frameworks don’t support that approach very well, so it works
[some
times](http://www.codethinked.com/comparing-simple-lambda-expressions-with-moq).

In conclusion, try to provide a single entry point to your unit of work
and expose all the aggregate roots with IQueryable as part of it
whenever it is possible.

