# RFC Name

Request for Contribution (RFC) 0000 : _Style guide_

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is: 

* documentation contributors
* HQ developers

## Summary

What kind of voice, language and punctuation rules should 'guide contribution' to the Umbraco documentation.

## Motivation

To provide a consistent writing style and tone of voice to improve readability and ultimately comprehension of information.
It will make first-time contributors feel more confident on the approach to make documentation.
It will help curators evaluate contributions consistently.

## Detailed Design

### Usage of language

The audience for Umbraco documentation includes many where English is not their native language. Therefore documentation should be simple, easy to understand and use short words, use everyday English and keep sentence length down.

While there has never been an official guide on the language being used. What words to we try to avoid. What kind of tone do we keep when writing (e.g. friendly, ...).

We try to avoid words like simply, just and obviously. We should explain why we would like to avoid these words and create a list of words we try to avoid and a list of words that we must avoid.

There are different types of documentation, reference, tutorial, etc - leading by 'code examples' is an international approach.

Not all humor translates, not all cultures have the same sense of humor or will understand sarcasm in the same way, therefore documentation should we written neutrally.  

Unless writing a tutorial, we should try to avoid... "you then have to do this, then set this, and then you just do this" try to avoid addressing the reader as 'you' - focus the writing on the words which describe the steps or details of the documentation. Tutorials are different, here the language needs to take the reader on a journey through the tutorial.

### Layout rules

We need to make sure, for consistency that we have simple layout rules.   
As a documentation visitor, if the documentation has consistent: tone of voice, punctuation, and simple layout paradigms, it improves the ease at which the information can be consumed and understood.

### Method of enforcement 

With the tooling like [Vale](https://errata-ai.github.io/vale/) we are able to enforce rules. Contributors should be able to checkout the documentation repository for the our documentation and check whether or not it can pass the enforced rules.

The Github repo could have (currently in development) a health check on every commit to be sure that commit adheres to the rules.

Next to the tooling, the documentation curator team is able to follow the guides.  They currently have no written notes on what should be accepted or rejected from the PR's.

## Drawbacks

* It might be intimidating if they try to contribute if they get errors after submitting a PR.
* It might be off-putting for people where English is not their native language.
* The rules might be so strict it has issues with parts (code examples, ...) of the documentation.
* It might be limiting if there are too many rules to adhere too.

## Alternatives

There are currently no alternatives.  We should know if we need a style guide.

## Out of Scope

* discussing tooling, this is an RFC about the content of a style guide.

## Unresolved Issues

The answers that we are hoping to get from the community & Umbraco HQ is:

* Is a style guide a good idea?
* What sort of rules should be enforced?
* What rules should not be enforced?
* If multi-lingual documentation is introduced how could style guides be managed per language?

## Related RFCs 

* there are no related RFCs

## Contributors

This RFC was compiled by:

* Jeavon Leopold
* Lotte Pitcher
* Jesper Mayntzhusen
* Damiaan Peeters
* Marc Goodson
* Sofie Toft Kristensen
