# RFC Name

Request for Contribution (RFC) 0000 : _Style guide_

## Code of conduct

Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## Intended Audience

The intended audience for this RFC is: 

* Documentation contributors
* HQ developers

## Summary

What kind of voice, language and punctuation rules should 'guide contribution' to the Umbraco documentation.

## Motivation

Provide a consistent writing style and tone of voice to improve readability and ultimately comprehension of information.
Make first-time contributors feel more confident on the approach to creating documentation.
Help curators evaluate contributions consistently.

## Detailed Design

### Usage of language

The used language should be in plain English, should avoid 

### Plain English

The audience for Umbraco documentation includes many where English is not their native language. Therefore documentation should be written with an international audience in mind. The used language should be simple and easy to understand. 

If in doubt, the document [How to write plain English](http://www.plainenglish.co.uk/files/howto.pdf) from plainenglish.co.uk, gives a good idea on what we expect. 

In summary: 

* Use short words
* Use everyday English
* Keep sentence length down
* Prefer active verbs
* Use lists where appropriate
* Annotate screenshots for accessibility

#### Tone of voice

Language should be friendly and direct. But not intimidating.
You can address the reader if you are telling a story, eg as part of a Tutorial - but should avoid this in most parts of the documentation.

Where possible documentation should be gender neutral.

Not all humor/humour translates, not all cultures have the same sense of humor or will understand sarcasm in the same way, therefore documentation should be written neutrally.  

There are different types of documentation, reference, tutorial, etc - leading by 'code examples' is an international approach.

Unless writing a tutorial, we should try to avoid... "you then have to do this, then set this, and then you just do this" try to avoid addressing the reader as 'you' - focus the writing on the words which describe the steps or details of the documentation.
Tutorials are slightly different, here the language needs to take the reader on a journey through the tutorial.

#### Words to avoid

Categories of words we like to avoid are 

* Negative words (like swearing) or words which are known to have a negative context
* Words can make the user feel stupid, therefore try avoiding the following connecting words: simply, just, of course and obviously. 

### Layout rules

Making pages follow a consistent simple layout will mean regular visitors to the documentation will learn the conventions, and improve the experience of interacting with the documentation.
Consider 'different journeys' to access documentation - introduction, deep dive or quick reference lookup.
Embedding of screenshots - consistent annotation approach.
Informative Tips, Warnings styled and delivered consistently

### Navigation

Clear language in navigation structure, leading visitors through the documentation.
Landing pages for sections and topics

#### Code examples

Make sure the examples are marked appropriately in order to have correct syntax highlighting.

* Examples should not contain foreign languages for variables.
* Try to adhere to the [.editorconfig](https://github.com/umbraco/Umbraco-CMS/blob/v8/dev/.editorconfig) style rules from the Umbraco-CMS repository.
* Code examples should not contain any commercial references like company names other than Umbraco

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

There are currently no alternatives... other than continuing without a style guide and manually reviewing submissions for perceived quality.

## Out of Scope

* Discussing tooling, this is an RFC about the content of a style guide.

## Unresolved Issues

The answers that we are hoping to get from the community & Umbraco HQ is:

* Is a style guide a good idea?
* What sort of rules should be enforced?
* What rules should not be enforced?
* If multi-lingual documentation is introduced how could style guides be managed per language?
* Would having a style guide make a difference to whether you would be more or less likely to contribute to the community documentation.

## Related RFCs 

* There are no related RFCs

## Contributors

This RFC was compiled by:

* Jeavon Leopold
* Lotte Pitcher
* Jesper Mayntzhusen
* Damiaan Peeters
* Marc Goodson
* Sofie Toft Kristensen
