# Creating an RFC

When creating an RFC, ensure that it meets these criteria:

* Most changes, including small features, bug fixes and documentation improvements can be implemented and reviewed via the normal GitHub pull request workflow. __Questions or discussions on more substantial changes should first be done on https://our.umbraco.com until further defined to the RFC format if one is deemed necessary.__
* The RFC should be a proposal, whether this is a technical proposal or an idea, it must propose an outcome.
* All topics for the RFC based on [the template](0000-rfc-template.md) must be known in advance of creating the RFC.

__Important__: _When submitting an RFC, it is the Author's responsibility to keep the RFC updated based on feedback made on the RFC. Therefor the Author must be willing to take on this responsibility before submitting an RFC and/or allow collaboration on their PR with other Authors._

## Instructions

* Fork this repository
* Copy {tempate-name} to /{product}/0000-my-feature.md (where 'my-feature' is descriptive) Don't assign an RFC number yet as this must match the number of the pull request that you will later submit. For example: /CMS/0000-dotnetCore.md
* Fill in the RFC. Put care into the details ensuring to provide information. RFCs that do not present convincing motivation, demonstrate understanding of the impact of the design, or are disingenuous about the drawbacks or alternatives tend to be poorly-received.
* Submit a pull request. As a pull request the RFC will receive design feedback from Umbraco HQ and the community, **and the author should be prepared to revise it in response**.
* (optional) Now that you have submitted a pull request, you can now assign the RFC number in the filename and title. This must always match the PR number to make the PR easy to find after the RFC has been merged. It's OK to skip this as it can be done while the RFC is being merged.