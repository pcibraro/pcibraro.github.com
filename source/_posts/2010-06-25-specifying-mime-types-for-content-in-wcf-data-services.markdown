---
layout: post
title: "Specifying mime types for content in WCF Data Services"
date: 2010-06-25
comments: true
categories: .NET
---

WCF Data Services provides an specific operator “\$value” for retrieving
the underline value of any of the properties in exposed resources. Let’s
say you have a resource “Configuration” that exposes a field “Xml” whose
content should be treated as a mime type “text/xml”. When you execute a
query like this to retrieve the value of that field,
“myService.svc/Configurations(1)/Xml/\$value”, what you get is the
content of that field but expressed as plain text (text/plain). This
happens because the default behavior for WCF Data Services is to return
most of the primitive types as “text/plain” and some others like byte
arrays as “application/octet-stream”. So, how do you change that default
behavior to retrieve property content with other mime types ?. Here is
where the MimeTypeAttribute can be used to change that behavior.

The MimeTypeAttribute can actually only be used with the
ReflectionProvider, which is the default provider for exposing objects
that are not EF entities. If you want to change the mime type for the EF
Provider, you need to modify the underline CSDL model (This is the part
that is not well documented, and I only found some incomplete references
in some forums).

For the reflection provider, the attribute must be specified at class
level,

[MimeType("Xml", "text/xml")] \
public class Configuration \
{ \
  public string Xml { get; set; } \
}

For the EF provider, a few additional changes are required in the CSDL
model. Edit the CSDL by opening the EDM model in Xml view, and adding a
reference to this namespace
"xmlns:m1="[http://schemas.microsoft.com/ado/2007/08/dataservices/metadata](http://schemas.microsoft.com/ado/2007/08/dataservices/metadata)""
in the \<edmx:ConceptualModels\> node  of the CSDL section in the EDMX
file. Right after that, locate the entity that you want to modify and
add the MimeType definition in the property.

\<EntityType Name="Configuration"\> \
          \<Property Name="Xml" m1:MimeType="text/xml" Type="String"
MaxLength="Max" Unicode="true" FixedLength="false" /\>

