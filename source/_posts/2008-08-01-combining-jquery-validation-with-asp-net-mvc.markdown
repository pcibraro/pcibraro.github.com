---
layout: post
title: "Combining JQuery Validation with ASP.NET MVC"
date: 2008-08-01
comments: true
categories: ASP.NET
---

One of the most nicest things about JQuery - in addition to the powerful
mechanism it provides to manipulate the HTML DOM - is the great number
of very useful plugins available out there.

[JQuery
Validation](http://bassistance.de/jquery-plugins/jquery-plugin-validation/)
is one of my favorites, and today we will see how this plugin can be
used with the MVC framework to validate all the inputs in a form before
it is submitted to the controller.

This plugin supports the concept of "validation rule", a validation that
has to be performed to an input field. For instance, "The field is
required", "The field should have at least N characters", or "This field
has to be a valid email", many of them are the same you can find in the
ASP.NET validators. (These validators do not work with the MVC framework
because they are tied to the ASP.NET Viewstate). Of course, new rules
can also be created for performing custom validations specific to an
application, some examples of this are also available in the [plugin's
website](http://bassistance.de/jquery-plugins/jquery-plugin-validation/).

A rule can be applied to an input field in two ways:

​1. Declarative, the rule is specified in the input field by means of
the class attribute:

\<input name="email" id="email" maxlength="60" class="required email"
type="text"/\>

As you can see, two rules were specified in the class attribute,
"Required" and "Email", which means that two validations have to be
performed for this field. Many rules can be applied to the same field,
they only have to be separated by an space.

​2. Imperative in code, the rule is specified in an script:

\<script type="text/javascript"\>

\$(document).ready(function(){

  \$("\#form-sign-up").validate( {

    rules: {

      email: {

        required: true,

        email: true

    },

    messages: {

      email: {

        required: "Please provide an email",

        email: "Please provide a valid email"

     } });

});

\</script\>

The validation was attached to the input field "email" in the form
"form-sign-up". The message displayed when a validation fails for an
specific field can also be customized using the "messages" section in
the script. (This is optional, the plugin already comes with a set of
pre-defined error messages)

And finally, one of the most interesting validation rules you can find
there is "remote", which performs a remote validation using an Ajax
endpoint. At this point we can use an MVC controller method to perform
an specific validation, for instance to see if a login name is still
available to be used.

\<script type="text/javascript"\>

\$(document).ready(function(){

\$("\#form-sign-up").validate( {

  rules: {

    login: {

      required: true,

      remote: '\<%=Url.Action("IsLoginAvailable", "Accounts") %\>'

   }

  },

  messages: {

    login: {

     required: "Please provide an alias",

     remote: jQuery.format("{0} is already in use")

   }

  } });

});

\</script\>

The only requirement for the controller is that it must be return Json
with the result of the validation. This can be easily done with MVC,

public JsonResult IsLoginAvailable(string login)

{

    //TODO: Do the validation

    JsonResult result = new JsonResult();

    if (login == "cibrax")

      result.Data = false;

    else

      result.Data = true;

 

    return result;

}

In the example above, if "cibrax" is entered as login name, the
validation will fail and the user will see an error message.

![](/images/legacy/MVC_Validation.jpg)

The styles for the error messages can also be customized with the
following rules,

label.error {

display: block;

color: red;

font-style: italic;

font-weight: normal;

}

input.error {

border: 2px solid red;

}

td.field input.error, td.field select.error, tr.errorRow td.field
input,tr.errorRow td.field select {

border: 2px solid red;

background-color: \#FFFFD5;

margin: 0px;

color: red;

}

As we have discussed here, JQuery validation is a great tool that we all
should consider at the moment to validate data in the ASP.NET MVC
framework.

A complete example with different validations can be [downloaded from
this location](/images/legacy/Mvc.Validation.zip).

