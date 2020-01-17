# Project UmbGet Package format 📦

Request for Contribution (RFC) 18 : _Package Format_

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is: Package developers and consumers

## Summary

Umbraco currently has two very used package formats: 
- The out of the box Umbraco ZIP package that can be generated from the backoffice
- The NuGet package that is the standard in .NET

This RFC's purpose is to suggest a way to bridge the gap and unify the two package formats while also looking closer at what a modern package format *should* solve.

## Motivation

A lot of package developers end up publishing their package as both a Zip and a NuGet package as there are benefits to either that the other doesn't have. However, as a developer working with NuGet packages are highly preferred, as it has some very essential features - the main one being automatic dependency handling.

We want to make it easier and simpler to create packages by only maintaining one format, while getting the best features of both.

## Detailed Design

The high level overview is that we want to remove the zip package format, and move to a NuGet zip (.nupkg), and then make sure that we can ensure that the benefits of the zip format are implemented:

- Listing in the backoffice
- Installing from the backoffice
- Ability to install Umbraco content and schema
- Ability to run code within the Umbraco context on install and uninstall

We have a more detailed list of features we want the new package format to support below. If you feel something is missing please check the **Out of scope** section before commenting, we may have the same idea but want to leave it out for this first phase!

### Creating a package

#### Umbraco content & media

We want to ensure that we don't take a step back with functionality compared to what you can do today. One of the things you can do in an Umbraco Zip package but can't do in a NuGet package today is to include content and schema (and media with a lot of extra work). 

We want to ensure that when you create a package in the backoffice you can include content and media (see schema below), to do that we will need to support serializing and deserializing content and media in the CMS.

We also want to simplify the package creation approach when including Umbraco schema and content to only allow you to pick all items. Most packages made this was are made on a specific site to develop packages, we believe it will make it faster and simpler to create and manage packages. For packages based on files that option will stay as is.

#### Umbraco schema

Similar to the Umbraco content and media above, we want to keep the option of installing schema (document types, data types, templates, languages, etc.). 

We also want to simplify the package creation approach when including Umbraco schema and content to only allow you to pick all items. Most packages made this was are made on a specific site to develop packages, we believe it will make it faster and simpler to create and manage packages. For packages based on files that option will stay as is.

### Installing / uninstalling a package

#### Supporting NuGet dependency handling


Nuget restoring dependencies and disallowing installing if dependencies conflict


#### Uniform install experience

Install experience is identical no matter where you install. 


#### Package migrations

Run when you want, not start up


#### Opt-in to changes

Package migrations will be opt-in from the backoffice if you have the right permissions
If you already ran package migrations it won’t run again in other environments if your database is in sync


#### Warn about what will be removed on uninstall

Warn what will be uninstalled
(More ideas, check Out of Scope)

### Other

#### New default package migrations

For adding content and schema based on serialized files


#### NuGet feed on Our

### Mockups

## Drawbacks

Discuss any disadvantages or sideeffects of this RFC.

## Alternatives

Are there any alternatives in approach that could be taken? 

## Out of Scope

We have a few ideas that would be awesome to have but will be out of scope for this RFC:

- Automatically check for upgrades and notify users with the right permissions
- Check for references in other content / schema when trying to uninstall something and warn the user
- Add option on uninstall to choose whether to uninstall the Umbraco content & schema
- Ideas to further improve and add to the suggested package actions

## Unresolved Issues

The answers that we are hoping to get from the community & Umbraco HQ is:

## Related RFCs 

List any other related RFCs. Provide links where you can.

## Contributors

This RFC was compiled by:

* Jesper Mayntzhusen (Umbraco HQ)
* Niels Hartvig (Umbraco HQ)
* Shannon Deminick (Umbraco HQ)
