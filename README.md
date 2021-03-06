# Yet Another Commit Checker [![Build Status](https://travis-ci.org/sford/yet-another-commit-checker.svg?branch=master)](https://travis-ci.org/sford/yet-another-commit-checker)

- [About](#about)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [FAQ](#faq)
- [Development](#development)

## About

This is an Atlassian Stash plugin that enforces commit message requirements. If a commit violates the
configured policies, the push to the repository will be rejected.

Features:

* Fully configurable!
* Require commit committer name and email to match Stash user
* Require commit messages to match regex
* Require commit message to contain valid JIRA issue ids
* Issue JQL matcher to validate JIRA issue against. Require issues to be assigned, not closed, in a certain project, etc.
The possibilities are endless!
* No extra JIRA configuration is required. Will use existing JIRA Application Link!
* Validate branch names.
* Customizable error messages (header, footer, and specific errors).
* Branch friendly! Only *new* commits are checked. Commits that already exist in the repository will be skipped.

Questions? Comments? Found a bug? See https://github.com/sford/yet-another-commit-checker!

Author: [Sean Ford](https://github.com/sford)

## Quick Start

1. Install YACC plugin into Stash
2. If you want to require valid JIRA issues, configure a JIRA Application Link in Stash
4. Configure YACC in the Hook Settings for a repository

## Configuration

####Require Matching Committer Email

If enabled, committer email must match the email of the Stash user.

####Require Matching Committer Name

If enabled, committer name must match the name of the Stash user.

####Commit Message Regex

If a regex is present, commit message must match regex.

Example,

    [A-Z0-9\-]+: .*

will require commit message to be in the form of:

    PROJ-123: added new feature xyz

#####Multi-line Commit Messages

Multi-line commit messages can be matched by including newlines into the regex (like `(.|\n)*`), or by enabling Pattern.DOTALL using the `(?s)` embedded flag expression. 

####Require Valid JIRA Issue(s)

If enabled, commit messages must contain valid JIRA issue ids. JIRA issue ids are defined as any item that matches
the regex `[A-Z][A-Z_0-9]+-[0-9]+`.

This check requires JIRA to be first linked with Stash using an Application Link. See https://confluence.atlassian.com/display/STASH/Linking+Stash+with+JIRA.

*Note:* This may result in false positives if commit messages contains strings that look like JIRA issue, for example, "UTF-8". Enable `Ignore Unknown JIRA Project Keys` to tell YACC to ignore items that don't contain a valid JIRA Project key.

#####Locating Issues Using a Regex Group

If a regex group is present in the `Commit Message Regex`, only text contained within this group will be examined when extracting JIRA issues.

For example, a `([A-Z0-9\-]+): .*` commit message regex will mean only `PROJ-123` will be checked against JIRA in the following commit message:

    PROJ-123: fixed bug involving UTF-8 support. I deserve a HIGH-5 for this fix!

UTF-8 and HIGH-5 will be ignored because they are not contained within the regex group. Using a regex group can be used as an alternative to `Ignore Unknown JIRA Project Keys` to deal with issue false positives, especially when you want to detect project key typos.

####Ignore Unknown JIRA Project Keys

If enabled, any issue-like items in commit messages that do not contain a valid JIRA project key (such as "UTF-8") will be ignored.

####Issue JQL Matcher

If JQL query is present, detected JIRA issues must match this query.

For example,

     assignee is not empty and status="in progress" and project=PROJ

will require that JIRA issues be assigned, in progess, and from project PROJ.

See [JIRA Advanced Searching](https://confluence.atlassian.com/display/JIRA/Advanced+Searching) for documentation regarding writing and testing
JQL queries.

####Branch Name Regex

If present, pushes to branches that don't match this regex will be blocked.

For example, `master|(?:(?:bugfix|hotfix|feature)/[A-Z]+-\d+-.+)` would enforce that pushes
should be done to branches that follow the Stash Branching Model naming convention.

*Note: This only affects pushes, not branches created through the Stash UI.*

####Exclude Merge Commits

If enabled, merge commits will be excluded from commit requirements.

####Exclude by Regex

If present, commits will be excluded from all requirements except matching committer email/name if part of the commit message matches this regex.

*Example:* `^Revert \"|#skipchecks`

####Exclude Service User Commits

If enabled, commits from service users (ie, using [SSH Access Keys](https://confluence.atlassian.com/display/STASH/SSH+access+keys+for+system+use))
will be excluded from commit requirements.

## FAQ

#### I am getting a JIRA authentication failed message when attempting to push my code or when trying to configure an issue JQL matcher.

This can occur if Stash is configured to use OAuth to authenticate with JIRA and the currently logged in Stash user has
not yet gone through the OAuth authorization process to allow Stash access to JIRA.

To initialize the OAuth tokens, go into the Stash UI and do something that requires access to JIRA. For example, view
the commits for a repository and click on an linked JIRA issue for an existing commit. See the [Stash JIRA Integration](https://confluence.atlassian.com/display/STASH/JIRA+integration#JIRAintegration-SeetheJIRAissuesrelatedtocommitsandpullrequests)
for an example of this.

There might be a better way to do this, but this what has worked for me :-)

#### How do I not pull my hair out if I need to configure YACC on a lot of repositories?

YACC is designed to be configured on a per-repository basis. If you would like to configure a lot of
repositories, or wish to keep repositories in sync, you can use the
[Stash REST API](https://developer.atlassian.com/static/rest/stash/3.8.0/stash-rest.html#idp2993072)
to automate configuring YACC.

#### YACC is rejecting my push complaining that my user name and/or email is wrong but the `Author:` from `git log` is correct!

Or, YACC is still complaining even after I fixed my commit using `git commit --amend --author`.

This is due to the fact that YACC checks the commit's *Committer* information against the Stash user,
not *Author*. These are normally the same; however, will be different when applying patches on behalf
of someone else or cherry picking commits.

You can verify the problem by using `git log --pretty=full` to see a commit's Committer information.

If your git settings where misconfigured, you can fix both the Author and Committer for your last
commit by doing:

    # Fix configuration
    git config user.name Your Name
    git config user.email your@email.com

    # Reset both author and committer
    git commit --amend --reset-author

## Development

Interested in contributing? [Fork me!](https://github.com/sford/yet-another-commit-checker)

Some useful development information:

### Enable Logging

Enabling YACC logging can be done using the Stash REST API. For example, see the following `curl` command which enables logging in the `atlas-run` development environment:

    curl -u admin -v -X PUT -d "" -H "Content-Type: application/json" http://localhost:7990/stash/rest/api/latest/logs/logger/com.isroot/debug

### Atlassian SDK

See `README_ATLASSIAN.txt` for the original Atlassian SDK README that contains some useful SDK commands.

