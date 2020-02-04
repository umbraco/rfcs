# Project UmbGet Package format 📦

Request for Contribution (RFC) 18 : _Package Format_

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is: Package developers and consumers

## Summary

Umbraco currently has two used package formats: 
- The out of the box Umbraco ZIP package that can be generated from the backoffice
- The NuGet package that is the standard in .NET

This RFC's purpose is to suggest a way to bridge the gap and unify the two package formats while also looking closer at what a modern package format *should* offer.

## Motivation

A lot of package developers end up publishing their package as both a Zip and a NuGet package as both formats have unique advantages. However, as a developer, working with NuGet packages are highly preferred, as it has some very essential features - the main one being automatic dependency handling.

We want to make it easier and simpler to create packages by only maintaining one format while getting the best features of both.

## Detailed Design

The high-level overview is that we want to remove the zip package format and move to a NuGet zip (.nupkg), and then make sure that that the benefits of the zip format is maintained:

- Listing in the backoffice
- Installing from the backoffice
- Ability to install Umbraco content and schema
- Ability to run code within the Umbraco context on install and uninstall

We have a more detailed list of features we want the new package format to support below. If you feel something is missing please check the **Out of scope** section before commenting, we may have the same idea but want to leave it out for this first iteration!

### Creating a package

#### Umbraco content & media

We want to ensure that we don't take a step back with functionality compared to what you can do today. One of the things you can do in an Umbraco Zip package but can't do in a NuGet package today is to include content and schema (and media with a lot of extra work). 

We want to ensure that when you create a package in the backoffice you can include content and media (see schema below), to do that we will need to support serializing and deserializing content and media in the CMS.

#### Umbraco schema

Similar to the Umbraco content and media above, we want to keep the option of installing schema (document types, data types, templates, languages, etc.). 

We also want to simplify the package creation approach when including Umbraco schema and content to only allow you to pick all items. Most packages made are made on a specific site to develop packages, we believe it will make it faster and simpler to create and manage packages. For packages based on files the option to pick files will stay as-is.

#### Package migrations 

Package actions are to be replaced with Package Migrations. The intent for package migrations is to **not** run on startup or install as we are used to with Package Actions and CMS migrations. Instead, when you install a package the migrations will be something you opt into running, along with an overview of what it will change.

Another huge benefit of these package migrations is when you deploy between different environments you can ensure that they run on each environment!

The details of these are still not set in stone, would appreciate any POC of either functionality of UI!

#### New default package migrations

Umbraco 8 currently has two default package actions, [allow Document Type](https://our.umbraco.com/documentation/Extending/Packages/Package-Actions/#allow-document-type) & [Publish root Document](https://our.umbraco.com/documentation/Extending/Packages/Package-Actions/#publish-root-document). In line with the above changes to when package migrations are run, a new default package migration would be added that takes a serialized file of content and schema and adds it to the database when run.


### Installing / uninstalling a package

#### Supporting NuGet dependency handling

One of the biggest benefits and most demanded feature of NuGet packages that Umbraco ZIP packages don't have is dependency handling. When you install a NuGet package in Visual Studio it will automatically install the highest version of the package that is compatible with your current installed packages. If you then later want to update your package to a newer version it will know what the dependencies for that package is and make sure to update them at the same time.

If you try to install a package and it is dependent on a package with a version that conflicts with your current packages it fails.

We want to bring a similar experience into the Umbraco backoffice when you install a package from the package dashboard. When you attempt to install a package Umbraco should restore the dependencies for you, or fail if there are version conflicts.

The specifics for how it would work are not set in stone, Arkadiusz Biel has made a [nice POC](https://github.com/umbraco/Umbraco.Packages/issues/19#issuecomment-554763674) we can start from.


#### Uniform install experience

This ties into the above, but we want to ensure that the install experience is the same no matter where you install your package, whether you drag and drop a .nupkg file into the backoffice, do it through Visual Studio or select a package in the backoffice it should feel and do the same.

We want to make sure that things like the dependency management works in each case as it could otherwise open up for confusion amongst package consumers.

#### Opt-in to changes

This was mentioned briefly in the package migrations section above, but one thing we want to accomplish with this new package format is to make it more user-friendly. Currently, when you install a package there is no transparency and you have no control over what happens. Installing a package can add files, schema, content, potentially unwanted or malicious things, deletions, not handle personal data properly, etc.

In an ideal world, you would install a package and get full control and insight into what it does, but that would also cause a lot of extra bloat. One thing we want to accomplish right now is to at least allow admin users to opt into when package migrations run.

If the package you install or update has one or more package migrations that haven't been run you will need to authorize them with an admin account. This accomplishes two things - first is that you would have more control over things being run on the database (file changes are a bit easier to manage using a versioning tool).

We want to show some sort of summary of what the migrations will do and present them to the user before they sign off on it.

Opting for migrations will also allow the user to plan when these things run. Can help with slow startup times due to migrations automatically running on startup.

#### Warn about what will be removed on uninstall

One thing that we want to see improved is the process of uninstalling a package. Currently, it can mess up your site if you uninstall a package you have used for content. Right now when you install a package it saves a list of all files and schema and content that it installed initially, and when you then uninstall it removes it all.

So if you start with a starter kit and then build upon document types to make a site, and then remove the package all content built on the starter kit document types would disappear.

We want to make it safer and more transparent what happens when you uninstall a package, we have several ideas but it became a huge project on its own. So, for now, the main idea is to show the user a list of things that will be removed when they try to uninstall. See more ideas under the Out of Scope section below.

### Other

#### NuGet feed on Our

We would change the current package repository on Our to a NuGet feed to host packages. Ideas on the best way of handling this are greatly appreciated.

### Mockups

TODO: File structure overview of the Umbraco NuGet package

TODO: Explain what technically happens on Package creation

TODO: Explain what technically happens when installing from the backoffice

TODO: Explain what happens when uninstalling

## Drawbacks

Discuss any disadvantages or sideeffects of this RFC.

## Alternatives

- Use proprietary format for backoffice creation, installation and uninstallation, similar to what exists.

## Out of Scope

We have a few ideas that would be awesome to have but will be out of scope for this RFC:

- Automatically check for upgrades and notify users with the right permissions
- Check for references in other content / schema when trying to uninstall something and warn the user
- Add option on uninstall to choose whether to uninstall the Umbraco content & schema

## Unresolved Issues

This RFC is mostly a feature plan for what we want to achieve with the first iteration of a new package format. Some of the features are on an early idea level and will need to be defined clearer before implementation can start. 

## Related RFCs 

* There are no related RFCs

## Contributors

This RFC was compiled by:

* Jesper Mayntzhusen (Umbraco HQ)
* Niels Hartvig (Umbraco HQ)
* Shannon Deminick (Umbraco HQ)
