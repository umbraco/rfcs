# RFC Name

Request for Contribution (RFC) 0020 : _Install Process under .NET Core_

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is technical users and developers.

## Summary

This RFC is a child of [Project UniCore - Introduction & Strategy](https://github.com/umbraco/rfcs/blob/master/cms/0001-project-unicore-intro.md)

In moving the Umbraco project to .NET Core, there will be changes needed in how a new Umbraco project is started. Existing methods for starting a new project won’t work with changes that have been made to NuGet. This RFC focuses on what those changes are, how they affect the Umbraco install process, and how we can continue to make the install process easy for installers.



## Motivation

The change from .NET Framework to .NET Core as the targeted framework means that we will need to reference NuGet packages using the `PackageReference` syntax instead of the legacy `packages.config` file. This change brings with it certain restrictions that Microsoft has imposed on the `PackageReference` dependencies. In particular, scripts are not supported cross-platform and install scripts and config file transformations are no longer executed at all.

Not being able to transform configuration files and not being able to run install scripts means that Umbraco does not have a simple way to set up all the deep interactions it needs with the ASP.NET Core website.

## Detailed Design

We suggest introducing a `dotnet new` template. These templates are used by Visual Studio, other IDEs, and the `dotnet` CLI to create new projects for .NET Core applications. Custom templates are supported, and allow for completely customizing the initial project structure and configuration.

The template would include the `UmbracoCMS` NuGet package and configure the project settings and initial files to allow for a user to start developing.

The templates allow for parameters to be provided to customize the installation process. For the initial implementation, we will support a project name as the only available parameter. Other parameters can be added in the future to further customize the Umbraco project.

This is the main part of the RFC. How do you see this being implemented in Umbraco?

## Drawbacks
The primary drawback is that the template needs to be installed before it can be used for the first time. This is an additional step before users can get started.

Another drawback is that configuration changes needed during upgrades would have to be handled in a different manner, since the `PackageReference` syntax no longer allows for install scripts, and .the `dotnet new` templates would only apply to new projects. 

Discuss any disadvantages or side-effects of this RFC.

## Alternatives

The only alternative we can come up with is to include a script that will do the configuration transformation and other needed things when executed. If we were to provide a script, it would need to be run manually after installing the NuGet package.
 
## Out of Scope

* The advanced configuration of the project template is out for the scope of the first edition.

## Unresolved Issues

The answers that we are hoping to get from the community are:

1. Are you aware of any better alternatives?
1. Are you aware of any possible way to achieve the existing workflow?

## Related RFCs 

* [RFC - Introduction & Strategy](https://github.com/umbraco/rfcs/blob/master/cms/0001-project-unicore-intro.md)

## Contributors

This RFC was compiled by:

* Benjamin Carleski (Umbraco Community)
* Bjarke Berg (Umbraco HQ)
* Emma Garland (Umbraco Community)
* Steve Temple (Umbraco Community)
* Andy Butland (Umbraco Community)
* Yvo Linssen (Umbraco Community)
* The Umbraco Unicore Team
