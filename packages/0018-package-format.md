# Project UmbGet Package format 📦

Request for Contribution (RFC) 22 : _Package Format_

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md).

## Intended Audience

The intended audience for this RFC is: Package developers and consumers.

## Summary

Umbraco currently has two package formats: 
- The default Umbraco ZIP package format that can be generated from the backoffice
- The standard .NET NuGet package format

The purpose of this RFC is to suggest a way to bridge the gap and unify the two package formats while also looking closer at what a modern package format *should* offer.

## Motivation

Many package developers end up publishing their packages as both a Zip and NuGet packages since both formats have unique advantages. However, as a developer, working with NuGet packages are highly preferred as it has some essential features such as having automatic dependency handling and being fully integrated with application development pipelines such as Visual Studio and CI/CD platforms.

We want to make it easier and simpler to create packages by only maintaining one format while getting the best features of both.

## Detailed Design

The high-level overview is that we want to remove the zip package format and move to a NuGet zip (.nupkg), and then make sure that that the benefits of the zip format is maintained:

- Listing in the backoffice
- Installing from the backoffice
- Ability to install Umbraco content and schema
- Ability to run code within the Umbraco context on install and uninstall
- Creating a package from the backoffice

We have a more detailed list of features we want the new package format to support below. If you feel something is missing please check the **Out of scope** section before commenting, we may have the same idea but want to leave it out for this first iteration!

### Creating a package

#### Umbraco content & media

We want to ensure that we don't take a step back with functionality compared to what you can do today. One of the things you can do in an Umbraco Zip package but can't do in a NuGet package today is to include content and schema (and media with a lot of extra work). 

We want to ensure that when you create a package in the backoffice you can include content and media (see schema below), to do that we will need to support serializing and deserializing content and media in the CMS.

#### Umbraco schema

Similar to the Umbraco content and media above, we want to keep the option of installing schema (document types, data types, templates, languages, etc.). 

We also want to simplify the package creation approach when including Umbraco schema and content to **only allow you to pick all items**. Most packages are made on a specific site to develop that one package, we believe it will make it faster and simpler to create and manage packages. For packages based on files the option to pick files will stay as-is.

#### Package Migrations 

What are "Package Migrations"? In Umbraco we have the concept of "Migrations" which are commands or instructions that are executed when Umbraco upgrades. We plan to re-use this same concept and code to allow developers to embed commands/instructions into their package which can be executed during installation and upgrades. 
Using Umbraco Migrations is not new and some packages already choose to use them, but it requires a lot of manual coding/plumbing and is generally inconsistent with how they are detected and executed. Primarily, because there is no common or officially documented way to use them. Package Migrations will be an official, documented and supported way for package developers to use Migrations. We plan on making this as easy as possible to use so package developers don't have to worry about writing any of the plumbing/detection code that they would need to do today.

Like Umbraco's normal Migrations, Package Migrations will ensure that these commands/instructions are only executed once per package version. This ensures they are not accidentally re-executed and are executed in each environment where the package exists. A great benefit of this is that you can be sure that a package's Migrations are executed on each environment when you deploy your changes. 

Note: We plan on updating Umbraco Cloud to ensure that Package Migrations are executed behind the scenes when deploying between environments, like what happens when you perform minor upgrades on Umbraco Cloud.

The old "Package Actions" format will be retired, however, we plan on ensuring that Package Migrations share the same simplicity. Currently, Umbraco Migrations are targeted mostly for database changes and if you wanted to do any content/schema manipulation in Umbraco Migrations, it would take quite a lot of code. We plan on making working with Package Migrations as easy as possible and will include many common operations. For example, working with content and schema in easy to use method calls that won't require writing much code, or even being able to list these operations as part of the package's metadata. The specific implementation details are yet to be determined and will be made more clear once some proof of concepts are created.

The intent for package migrations is to **not** run on startup or install as we are used to with Package Actions and custom Migrations. Instead, when you install a package the migrations will be something you opt into running, along with an overview of what it will change. The UX of this is not designed but the idea is that:

* If you install a .nupgk package via the backoffice, once the package files are installed you will be presented with some UI listing what the Package Migration will execute. An admin can then press a button to accept the changes and the migration will execute. If the migrations are not executed then the admin can re-visit the package's migration page which can be navigated to via the packages section in the backoffice.
* If you install an Umbraco NuGet package via NuGet in Visual Studio (or another IDE), the files will install as per normal NuGet. When the site is started and the admin logs into the backoffice they will need to navigate to the installed package's migration page to execute the migrations.
* In both cases, there may be some new UI that alerts an admin user that there are pending migrations that need executing when they log in.



#### Default package migrations

When a package is created or updated in the backoffice and the package contains content or schema items then a default package migration will be added to the package. It will ensure that the serialized content and/or schema items are installed into the Umbraco database when it's run.


### Installing / uninstalling a package

Installing a NuGet package via the backoffice will require validating the NuGet package as being an Umbraco NuGet package. We will not be allowing installing non-Umbraco .nupgk packages via the backoffice directly (though those will be installed as part of the dependency handling chain). 

The backoffice will only list packages found on Our, like the current backoffice package listing. If a developer manually uploads a .nupgk package it will be validated to ensure that it is an Umbraco based NuGet package based on it's file structure and metadata. 

#### Supporting NuGet dependency handling

One of the biggest benefits and most demanded feature of NuGet packages that Umbraco ZIP packages don't have is dependency handling. When you install a NuGet package in Visual Studio it will automatically install the highest version of the package that is compatible with your current installed packages. If you then later want to update your package to a newer version it will know what the dependencies for that package is and make sure to update them at the same time.

If you try to install a package and it is dependent on a package with a version that conflicts with your current packages it fails.

We want to bring a similar experience into the Umbraco backoffice when you install a package from the package dashboard. When you attempt to install a package Umbraco should restore the dependencies for you, or fail if there are version conflicts.

The specifics for how it would work are not set in stone but ideally NuGet APIs are used since they already deal with dependency resolution, etc.... Arkadiusz Biel has made a [nice POC](https://github.com/umbraco/Umbraco.Packages/issues/19#issuecomment-554763674) we can start from. Due to the complexity and potential edge cases of this there will probably be a few iterations and proof of concepts created. For example, there will be a requirement that a NuGet package has a dependency on one of the Umbraco packages like `UmbracoCms` or `UmbracoCms.Web` or `UmbracoCms.Core` even if the package is only JavaScript. This is because this dependency will be the flag that determines which version of Umbraco is compatible with the package. But unlike installing this package via NuGet in Visual Studio, installing a NuGet package in the backoffice is never going to update the Umbraco package files, the dependency for backoffice installation would only serve as the flag for package compatibility. 

Other challenges of this is about package uninstallation and how to deal with dependencies. If a NuGet package is installed in the backoffice and is dependent on 10x other NuGet packages, when we uninstall the main package we cannot also uninstall the other 10x dependencies. There may be other packages that are dependent on these packages. To keep this manageable, when working with packages in the backoffice we won't uninstall package dependencies when a package is uninstalled, like the way Visual Studio works with NuGet packages with the packages.config approach. A developer would then have the option to manually uninstall dependencies if there are no other packages dependent on it.


#### Uniform install experience

This ties into the above, but we want to ensure that the install experience is the same no matter where you install your package. Whether you drag and drop a .nupkg file into the backoffice, do it through Visual Studio or select a package in the backoffice it should feel and do the same.

We want to make sure that things like the dependency management works in each case as it could otherwise open up for confusion among package consumers.

#### Opt-in to changes

This was mentioned briefly in the package migrations section above, but one thing we want to accomplish with this new package format is to make it more user-friendly. Currently, when you install a package there is no transparency and you have no control over what happens. Installing a package can add files, schema, content, potentially unwanted or malicious things, deletions, not handle personal data properly, etc.

In an ideal world, you would install a package and get full control and insight into what it does, but that would also cause a lot of extra bloat. One thing we want to accomplish right now is to at least allow admin users to opt into when package migrations run.

If the package you install or update has one or more package migrations that haven't been run you will need to authorize them with an admin account. This accomplishes two things - first is that you would have more control over things being run on the database (file changes are a bit easier to manage using version/source control such as Git).

We want to show some sort of summary of what the migrations will do and present them to the user before they sign off on it.

Opting in for migrations will also allow the user to plan when these things run. Can help with slow startup times due to migrations automatically running on startup.

#### Warn about what will be removed on uninstall

One thing that we want to see improved is the process of uninstalling a package. Currently, it can mess up your site if you uninstall a package you have used for content. Right now when you install a package it saves a list of all files and schema and content that it installed initially, and when you then uninstall it removes it all.

So if you start with a starter kit and then build upon document types to make a site, and then remove the package all content built on the starter kit document types would disappear.

We want to make it safer and more transparent what happens when you uninstall a package, we have several ideas but it became a huge project on its own. So, for now, the main idea is to show the user a list of things that will be removed when they try to uninstall. See more ideas under the Out of Scope section below.

### Other

#### NuGet feed on Our

We will create a NuGet feed on the Our website for packages uploaded to Our. This means any package in the new .nupkg package format that is uploaded to Our will be able to be consumed by a new custom NuGet feed. This custom NuGet feed can be consumed within Visual Studio or any development IDE by adding the Our NuGet feed URL to your NuGet.config file. The feed URL is yet to be determined.

If developers wish to upload their packages to NuGet as well that is totally ok. Perhaps in the future there will be a way to automatically publish a package from Our to your NuGet account. There's no problem with having the same package/version in 2 different feeds.

## Drawbacks

Discuss any disadvantages or sideeffects of this RFC.

## Alternatives

- Use proprietary format for backoffice creation, installation and uninstallation, similar to what exists.

## Out of Scope

We have a few ideas that would be awesome to have but will be out of scope for this RFC:

- Automatically check for upgrades and notify users with the right permissions
- Check for references in other content / schema when trying to uninstall something and warn the user
- Add option on uninstall to choose whether to uninstall the Umbraco content & schema
- Use Umbraco Deploy as the serialization/deserialization engine

## Unresolved Issues

This RFC is mostly a feature plan for what we want to achieve with the first iteration of a new package format. Some of the features are on an early idea level and will need to be defined clearer before implementation can start. 

- What packages do we list as installed in the backoffice? If a package is installed in the backoffice and it requires 10x NuGet packages that are not Umbraco NuGet packages but instead normal .NET libraries, do we list all of these? This would essentially mean we'd be listing everything that would exist in a `packages.config` file. If we don't list all of these then the backoffice user will have no way to uninstall these dependencies since we aren't going to uninstall a package's dependencies when it is uninstalled from the backoffice.

## Related RFCs 

* There are no related RFCs

## Contributors

This RFC was compiled by:

* Jesper Mayntzhusen (Umbraco HQ)
* Niels Hartvig (Umbraco HQ)
* Shannon Deminick (Umbraco HQ)
