> A journey of a thousand miles begins with a single step


# Umbraco RFCs

A request for contribution is a proposal of a change/ new feature that will be reviewed and discussed by Umbraco HQ and community. 

In using RFCs will:
* enable individual contributors to help make decisions.
* allow domain experts to have input in decisions.
* manage the risk of decisions made.
* have a snapshot of context for the future.
* transparency of decision making by Umbraco HQ teams.


Most changes, including bug fixes and documentation improvements can be implemented and reviewed via the normal GitHub pull request workflow. Questions or discussions on potential changes should be on Our until further defined to the RFC format.

Some changes though are "substantial", and we ask that these be put through more of a design process to produce a consensus.
The "RFC" (request for contributions) process is intended to provide a consistent and controlled way for Umbraco HQ and community to collaborate on important decisions on the codebase.

As the name suggests, this is an invitation to contribute to the conversation with suggestions and ideas. Please read and respect the [RFC Code of Conduct](https://github.com/umbraco/rfcs/blob/master/CODE_OF_CONDUCT.md)

## This repository & the RFC process

This repository is intended to be used for reviewing submitted RFCs. The process of submitting RFCs is currently under review so if you wish to submit your own RFC please wait for further guidance.

## WIP...

### Creating an RFC

* Fork this repository
* Copy {tempate-name} to /{product}/0000-my-feature.md (where 'my-feature' is descriptive. Don't assign an RFC number yet). For example: /CMS/0000-dotnetCore.md
* Fill in the RFC. Put care into the details ensuring to provide information. RFCs that do not present convincing motivation, demonstrate understanding of the impact of the design, or are disingenuous about the drawbacks or alternatives tend to be poorly-received.
* Submit a pull request. As a pull request the RFC will receive design feedback from the larger community, and the author should be prepared to revise it in response.
* An RFC may be rejected by the team after public discussion has settled and comments have been made summarizing the rationale for rejection. A member of the team should then close the RFCs associated pull request.
* An RFC may be accepted at the close of its final comment period. A team member will merge the RFCs associated pull request, at which point the RFC will become 'active'.

### The RFC life-cycle

When an RFC is accepted then the process of implementing it may begin in the code repository. Being accepted does not imply anything about the priority of having it implemented and doesn't necessarily indicate that it's in active development.

Modifications to active RFCs can be done in followup PRs. 

## Credits

Umbraco's RFC process is inspired by the [React RFC process](https://github.com/reactjs/rfcs), [Rust RFC process](https://github.com/rust-lang/rfcs), and [IETF RFC](https://www.ietf.org/standards/rfcs/) documents
