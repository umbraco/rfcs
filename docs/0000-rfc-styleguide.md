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

The used language should be in plain English, should avoid 

### Plain English

The audience for Umbraco documentation includes many where English is not their native language. Therefore documentation should be  written with an international audience in mind. The used language should be simple, easy to understand. 

If in doubt, the document [How to write plain English](http://www.plainenglish.co.uk/files/howto.pdf) from plainenglish.co.uk, gives a good idea on what we expect. 

In summary: 

* Use short words
* use everyday English
* keep sentence length down
* prefer active verbs
* Use lists where appropriate

#### Tone of voice

Language should be friendly and direct. But not intimidating.
You can address the reader if you are telling a story, but should avoid this in most parts of the documentation.

Where possible documentation should be gender neutral.

Not all humor translates, not all cultures have the same sense of humor or will understand sarcasm in the same way, therefore documentation should we written neutrally.  

There are different types of documentation, reference, tutorial, etc - leading by 'code examples' is an international approach.

Unless writing a tutorial, we should try to avoid... "you then have to do this, then set this, and then you just do this" try to avoid addressing the reader as 'you' - focus the writing on the words which describe the steps or details of the documentation. Tutorials are different, here the language needs to take the reader on a journey through the tutorial.

#### Words to avoid

Categories of words we like to avoid are 

* negative words (like swearing) or words which are know to have a very negative context
* words can make the user feel stupid, therefor try avoiding the following words: simply, just and obviously. 

### Layout rules

As a documentation visitor, if the documentation has consistent: tone of voice, punctuation, and simple layout paradigms, it improves the ease at which the information can be consumed and understood.

#### Code examples

Make sure the examples are marked appropriately in order to have correct syntax highlighting.

* Examples should not contain foreign languages for variables.
* Try to adhere to the [.editorconfig](https://github.com/umbraco/Umbraco-CMS/blob/v8/dev/.editorconfig) style rules from the Umbraco-CMS repository.
* Code examples should not contain any commercial references like company names orther than Umbraco

### Method of enforcement 

With the tooling like [Vale](https://errata-ai.github.io/vale/) we are able to enforce rules. Contributors should be able to checkout the documentation repository for the our documentation and check whether or not it can pass the enforced rules.

The Github repo could have (currently in development) a health check on every commit to be sure that commit adheres to the rules.

In addition to automated tooling, the documentation curator team will use the style guide when reviewing documentation additions.

#### Suggested Vale rules

We have set up some suggested rules based on best practises and limiting the impact they will have for contributors:

- **HeadingsPunctuation**: Warns you when using punctuation at the end of a heading
- **Hyperbolic**: Warns you when using words such as _simple, just, easily and actually_
- **ListStart**: Warns you when starting a list option with lower case
- **SentenceLength**: Warns you when writing sentences with more than 40 words in them
- **Spacing**: Warns you when having more than 1 space in succession in an article
- **Terms**: Warns you when using a blacklisted term and suggests a replacement - could be from _back-office_ to _backoffice_

## Drawbacks

* It might be intimidating if contributors try to submit documentation and receive errors after submitting a PR.
* It might be demotivating if the errors appear to be 'trivial' to the contributor - particularly if the contributor is providing information that is completely missing from the documentation. We don't want the reputation that you have to jump through hoops to contribute - and so how the guide is applied is therefore important.
* It might be off-putting for people where English is not their native language.
* The rules might be too strict - and automated issues are flagged with parts of the contribution that are technical terms eg. errors with code examples...
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
