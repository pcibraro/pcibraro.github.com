---
layout: post
title: "Code generation with T4, Entities to DTOs example"
date: 2009-03-11
comments: true
categories: .NET
---

T4 is a powerful template engine for code generation shipped out of the
box within Visual Studio.  It is an evolution of T3, which was initially
introduced a couple of years ago as part of  the DSL toolkits and the
software factories.

Today, it is getting more attention from other product teams as well,
for instance, the ASP.NET MVC and Entity Framework teams have recently
announced that they will ship T4 templates as part of their products.
That will provide a way to customize the code they are generating from
custom tools or visual studio item templates.

A cool thing is that you do not need any custom tool box to automate the
code generation. You can simply add a T4 template to any Visual Studio
project, rename it to use the extension "tt", and visual studio will do
the rest.

One of the major pains, however, is the authoring experience for
creating or modifying existing T4 templates in Visual Studio, there is
not built-in support for doing that.  The company where I am currently
working on, [Clarius Consulting](http://www.clariusconsulting.net/), has
made the best designer ever for authoring T4 templates within Visual
Studio, [the T4 editor](http://www.visualt4.com/). Some of the features
included with this designer are syntax coloring, intellisense or code
preview to name a few.

This last weekend, while I was delayed a complete day in DC for
mechanical problems  in one of the airplanes, I decided it was a good
moment to start playing with this technology and make something
productive with my time. The result was a T4 template for auto
generating a DTO (Data transfer objects) layer based on WCF data
contracts from an existing entity model. Using DTOs is a common practice
for transferring the state of different entities across service
boundaries, they are frequently found in system designs that follow the
DDD principles.

Although the resulting code is practically useless, it can be easily
customized for supporting different scenarios. (Or at least, it will
help to give you an idea about how this thing can be done)

The structure of a T4 template is quite simple (And somehow similar to
an ASP.NET page without any control), "\<\# \#\>" for wrapping multiple
lines of code or "\<\#= \#\>" for inline code, the rest of the template
is treated as text.

For this example I used a model with a few entities (an [anemic domain
model](http://en.wikipedia.org/wiki/Anemic_Domain_Model) I would say),

public class Employee

{

    public string Name

    {

        get;

        set;

    }

    public Employee Boss

    {

        get;

        set;

    }

    public Company Company

    {

        get;

        set;

    }

}

public class Company

{

    public string CompanyName

    {

        get;

        set;

    }

    public List\<Employee\> Employees

    {

        get;

        set;

    }

}

The template filters the types that have to be included in the code
generating process with a Linq expression, all the entities in the
"EntitiesToDTO.Entities" namespace in this case.

\<\#

var entitiesNamespace = "EntitiesToDTO.Entities";

//Use another expression here to filter the entities

var typesToRegister = from t in
LoadProjectAssembly(entitiesAssembly).GetExportedTypes()

                      where t.Namespace == entitiesNamespace &&
t.IsClass && !t.IsAbstract

                      select t;

\#\>

The resulting code is also quite straightforward, it includes a partial
class that can be extended to support additional mappings.

[DataContract(Name="employee", Namespace="urn:EntitiesToDTO/Entities")]

public partial class EmployeeDTO   

{

    [DataMember(Name="name")]

    public System.String Name

    {

        get; set;

    }

    [DataMember(Name="boss")]

    public EmployeeDTO Boss

    {

        get; set;

    }

    [DataMember(Name="company")]

    public CompanyDTO Company

    {

        get; set;

    }

    public Employee MapTo(EmployeeDTO dto)

    {

        return GetMapper().MapTo(dto);

    }

    public static EmployeeDTO MapFrom(Employee entity)

    {

        return GetMapper().MapTo(entity);

    }

    public static EmployeeMapper GetMapper()

    {

        return new EmployeeMapper();  

    }

    public partial class EmployeeMapper

    {

        public EmployeeDTO MapTo(EntitiesToDTO.Entities.Employee entity)

        {

            var dto = new EmployeeDTO

            {  

                Name = entity.Name,

                Boss = EmployeeDTO.GetMapper().MapTo(entity.Boss),

                Company = CompanyDTO.GetMapper().MapTo(entity.Company),

            };

            DoMapping(dto, entity);

            return dto;

        }

        public EntitiesToDTO.Entities.Employee MapTo(EmployeeDTO dto)

        {

            var entity = new EntitiesToDTO.Entities.Employee

            {  

                Name = dto.Name,

                Boss = EmployeeDTO.GetMapper().MapTo(dto.Boss),

                Company = CompanyDTO.GetMapper().MapTo(dto.Company),

            };

            DoMapping(entity, dto);

            return entity;

        }

        partial void DoMapping(EntitiesToDTO.Entities.Employee
fromEntity, EmployeeDTO toDto);

        partial void DoMapping(EmployeeDTO fromDto,
EntitiesToDTO.Entities.Employee toEntity);

    }

} 

The mapper (EmployeeMapper) is an additional class to handle the
mappings between entities and DTOs, a mapping class per DTO is
generated. As you can see in the code below, either the EmployeeDTO or
the EmployeeMapper can be extended with a partial class to perform
additional mappings. For example,

public partial class EmployeeDTO

{

    [DataMember(Name = "fullName")]

    public string FullName { get; set; }

    public partial class EmployeeMapper

    {

        partial void DoMapping(Employee fromEntity, EmployeeDTO toDto)

        {

            toDto.FullName = fromEntity.Name + " Foo";

        }

    }

}

The sample is available to download from
[here](/images/legacy/EntitiesToDTO.zip).

