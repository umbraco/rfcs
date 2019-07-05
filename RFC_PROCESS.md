# The Umbraco RFC process

This repository is intended to be used for reviewing submitted RFCs. 

## Process outline

* An RFC [pull request is made](RFC_CREATION.md) by Umbraco HQ and thereby [open for comments](#Reviewing).
* The RFC may may be [updated, closed or restarted if necessary](#Updating) by the Author if they choose to.
* When Umbraco HQ decides to finalize the RFC, it will be tagged for final comments and a deadline will be provided.
* At the provided deadline the [RFC will be accepted or closed](#Acceptance) by Umbraco HQ.

## Reviewing

An RFC will be submitted in the form of a [Pull Request](https://github.com/umbraco/rfcs/pulls). Comments can be made directly on the Pull Request itself while the PR is open.

## Updating

The RFC should always reflect it's current state and the Author(s) may update the RFC to reflect any decisions made while it is open. 

In some cases it will make more sense to close an RFC and start over again if the scope or purpose of the RFC should be radically changed.

## Acceptance

When Umbraco HQ decides to finalize an RFC it will be tagged (`final-comment-period`) as having a final comment period which will generally be one or two weeks. An RFC may be accepted at the end of its final comment period and the associated pull request will be merged. When accepted, the RFC must be up to date and accurate and will represent the final intention of the RFC.

An RFC may be closed (not accepted) after discussion has settled and comments have been made summarizing the rationale for being rejected. An RFC's associated PR is closed if this occurs. Other reasons an RFC may not be accepted is if it doesn't contain the appropriate information or if the RFC was submitted without previous discussion or HQ knowledge about it's intentions. In some cases once an RFC is closed, comments may be locked and if a follow up RFC is proposed, another PR will be submitted. 

## Implementation

When an RFC is accepted, then the process of implementing it may begin. The implementation process could be task(s) created on the project's issue tracker, or it could mean further, more detailed RFCs are created. In some cases tasks for an RFC will be created on the tracker tagged as `up-for-grabs` which means the implementation can be carried out by anyone.

Being accepted does not imply anything about the priority of having it implemented and doesn't necessarily indicate that it's in active development.