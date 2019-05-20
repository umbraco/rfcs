# RFC Name

Request for Contribution (RFC) 0000 : _Style guide_

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is: 

* documentation contributors
* hq developers

## Summary

What kind of language and punctation rules should be set on the documentation.

## Motivation

To make sure that there is a consistent writing style and to improve readability we want to include some rules.  
It will make first time contributors feel more confident on the approach to make documentation.

## Detailed Design

This is the main part of the RFC. How do you see this being implemented in Umbraco?

### Usage of language

While there has never been an official guide on the language being used.  The future tooling (docfx) allows us to have multi lingual documentation.  We would need to set guides for translators.  What words to we try to avoid.  What kind of tone do we keep when writng (e.g. friendly, ...).

We try to avoid words like simply, just, ...  We should explain why we would like to avoid these word.  And create a list of words we try to avoid.  And have a list of words we need to avoid.

### Layout rules

We need to make sure, for consistency that we have simple layout rules.   
As a documentation visitor, having the same punctuation and layout improves documentation. 

### Method of enforcement 

With the tooling like [Vale](https://errata-ai.github.io/vale/) we are able to enforce rules. Contributors should be able to checkout the documentation repository for the our documentation and check whether or not it can pass the enforced rules.

The github repo could have (currently in development) a health check on every commit to be sure that commit adheres to the rules.

Next to the tooling, the documentation curator team is able to follow the guides.  They currently have no written notes on what should be accepted or rejected from the PR's.

## Drawbacks

* It might be intimidating if they try to contribute if they get errors after submitting a PR.
* It might be off putting for people where English is not their native language.
* The rules might be so strict it has issues with parts (code examples, ...) of the documenation.
* It might be limiting if there are too many rules to adhere too.

## Alternatives

There are currently no alternatives.  We should know if we need a style guide.

## Out of Scope

* discussing tooling, this is a RFC about the content of a styleguide

## Unresolved Issues

The answers that we are hoping to get from the community & Umbraco HQ is:

* Is a styleguide a good idea?
* What sort of rules should be enforced?
* What rules should not be enforced?

## Related RFCs 

* there are no related RFCs

## Contributors

This RFC was compiled by:

* Jeavon Leopold
* Lotte Pitcher
* Jesper Mayntzhusen
* Damiaan Peeters
