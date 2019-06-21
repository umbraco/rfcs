# The Umbraco RFC process

This repository is intended to be used for reviewing submitted RFCs. 

## Process outline

* An RFC [pull request is made](RFC_CREATION.md) and thereby [open for comments](#Reviewing).
* The RFC will be [updated, closed or restarted if necessary](#Updating) by the Author based on feedback.
* When Umbraco HQ decides to finalize the RFC, it will be tagged for final comments and a deadline will be provided.
* At the provided deadline the [RFC will be accepted or closed](#Acceptance) by HQ.

## Reviewing

An RFC will be submitted in the form of a [Pull Request](https://github.com/umbraco/rfcs/pulls). Contribution to the conversation will be done via comments on the Pull Request itself while the PR is open.

## Updating

The RFC may need to be updated while the RFC is open to reflect decision made based on current feedback. It is the Author's responsibility to keep the RFC up to date. There may be multiple authors for an RFC which can be achieved by granting other GitHub users contribution access to the submitting PR. 

At any given time, the RFC should reflect it's current status based on feedback. 

In some cases it will make more sense to close an RFC and start over again if the feedback indicates that the scope or purpose of the RFC should be radically changed.

## Acceptance

When Umbraco HQ decides to finalize an RFC it will be tagged (`final-comment-period`) as having a final comment period which will generally be one or two weeks. An RFC may be accepted at the end of its final comment period and the associated pull request will be merged. When accepted, the RFC must be up to date and accurate and will represent the final intention of the RFC.

An RFC may be closed (not accepted) after discussion has settled and comments have been made summarizing the rationale for being rejected. An RFC's associated PR is closed if this occurs. Other reasons an RFC may not be accepted is if it doesn't contain the appropriate information or if the RFC was submitted without previous discussion or HQ knowledge about it's intentions. In some cases once an RFC is closed, comments may be locked and if a follow up RFC is proposed, another PR will be submitted. 

## Implementation

When an RFC is accepted, then the process of implementing it may begin. The implementation process could be task(s) created on the project's issue tracker, or it could mean further, more detailed RFCs are created. In some cases tasks for an RFC will be created on the tracker tagged as `up-for-grabs` which means the implementation can be carried out by anyone.

Being accepted does not imply anything about the priority of having it implemented and doesn't necessarily indicate that it's in active development.