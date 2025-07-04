---
title: Readarr Contributing (Retired)
description: 
published: true
date: 2022-09-26T15:56:42.368Z
tags: readarr, development, contributing
editor: markdown
dateCreated: 2021-05-25T19:21:59.324Z
---

# Announcement: Retirement of Readarr

We would like to announce that the [Readarr project](https://github.com/Readarr/Readarr) has been retired. This difficult decision was made due to a combination of factors: the project's metadata has become unusable, we no longer have the time to remake or repair it, and the community effort to transition to using Open Library as the source has stalled without much progress.

Third-party metadata mirrors exist, but as we're not involved with them at all, we cannot provide support for them. Use of them is entirely at your own risk. The most popular mirror appears to be [rreading-glasses](https://github.com/blampe/rreading-glasses).

Without anyone to take over Readarr development, we expect it to wither away, so we still encourage you to seek alternatives to Readarr.

## Key Points

- Effective Immediately: The retirement takes effect immediately. Please stay tuned for any possible further communications.
- Support Window: We will provide support during a brief transition period to help with troubleshooting non metadata related issues.
- Alternative Solutions: Users are encouraged to explore and adopt any other possible solutions as alternatives to Readarr.
- Opportunities for Revival: We are open to someone taking over and revitalizing the project. If you are interested, please get in touch.
- Gratitude: We extend our deepest gratitude to all the contributors and community members who supported Readarr over the years.

Thank you for being part of the Readarr journey. For any inquiries or assistance during this transition, please contact our team.

Sincerely,
The Servarr Team

# How to Contribute

We're always looking for people to help make Readarr even better, there are a number of ways to contribute.

# Documentation

Setup guides, [FAQ](/readarr/faq), the more information we have on the [wiki](https://wiki.servarr.com/readarr) the better.

# Development

Readarr is written in C# (backend) and JS (frontend). The backend is built on the .NET6 (and _soon_ .NET8) framework, while the frontend utilizes Reactjs.

## Tools required

- Visual Studio 2022 or higher is recommended (<https://www.visualstudio.com/vs/>). The community version is free and works (<https://www.visualstudio.com/downloads/>).

> VS 2022 V17.0 or higher is recommended as it includes the .NET6 SDK
{.is-info}

- HTML/Javascript editor of choice (VS Code/Sublime Text/Webstorm/Atom/etc)
- [Git](https://git-scm.com/downloads)
- The [Node.js](https://nodejs.org/) runtime is required. The following versions are supported:
  - **20** (any minor or patch version within this)
{.grid-list}

> The Application will **NOT** run on older versions such as `18.x`, `16.x` or any version below 20.0! Due to a dependency issue, it will also not run on `21.x` and is untested on other verisons.
{.is-warning}

- [Yarn](https://yarnpkg.com/getting-started/install) is required to build the frontend
  - Yarn is included with **Node 20**+ by default. Enable it with `corepack enable`
  - For other Node versions, install it with `npm i -g corepack`

## Getting started

1. Fork Readarr
1. Clone the repository into your development machine. [*info*](https://docs.github.com/en/get-started/quickstart/fork-a-repo)

> Be sure to run lint `yarn lint --fix` on your code for any front end changes before committing.
For css changes `yarn stylelint-windows --fix` {.is-info}

### Building the frontend

- Navigate to the cloned directory
- Install the required Node Packages

     ```bash
     yarn install
     ```

- Start webpack to monitor your development environment for any changes that need post processing using:

     ```bash
     yarn start
     ```

### Building the Backend

The backend solution is most easily built and ran in Visual Studio or Rider, however if the only priority is working on the frontend UI it can be built easily from command line as well when the correct SDK is installed.

#### Visual Studio

> Ensure startup project is set to `Readarr.Console` and framework to `net6.0`
{.is-info}

1. First `Build` the solution in Visual Studio, this will ensure all projects are correctly built and dependencies restored
1. Next `Debug/Run` the project in Visual Studio to start Readarr
1. Open <http://localhost:8787>

#### Command line

1. Clean solution

```shell
dotnet clean src/Readarr.sln -c Debug
```

1. Restore and Build debug configuration for the correct platform (Posix or Windows)

```shell
dotnet msbuild -restore src/Readarr.sln -p:Configuration=Debug -p:Platform=Posix -t:PublishAllRids
```

1. Run the produced executable from `/_output`

## Contributing Code

- If you're adding a new, already requested feature, please comment on [GitHub Issues](https://github.com/Readarr/Readarr/issues) so work is not duplicated (If you want to add something not already on there, please talk to us first)
- Rebase from Readarr's develop branch, do not merge
- Make meaningful commits, or squash them
- Feel free to make a pull request before work is complete, this will let us see where its at and make comments/suggest improvements
- Reach out to us on the discord if you have any questions
- Add tests (unit/integration)
- Commit with \*nix line endings for consistency (We checkout Windows and commit \*nix)
- One feature/bug fix per pull request to keep things clean and easy to understand
- Use 4 spaces instead of tabs, this is the default for VS 2022 and WebStorm

## Pull Requesting

- Only make pull requests to `develop`, never `master`, if you make a PR to `master` we will comment on it and close it
- You're probably going to get some comments or questions from us, they will be to ensure consistency and maintainability
- We'll try to respond to pull requests as soon as possible, if its been a day or two, please reach out to us, we may have missed it
- Each PR should come from its own [feature branch](http://martinfowler.com/bliki/FeatureBranch.html) not develop in your fork, it should have a meaningful branch name (what is being added/fixed)
  - `new-feature` (Good)
  - `fix-bug` (Good)
  - `patch` (Bad)
  - `develop` (Bad)
- Commits should be wrote as `New:` or `Fixed:` for changes that would not be considered a `maintenance release`

## Unit Testing

Readarr utilizes nunit for its unit, integration, and automation test suite.

### Running Tests

Tests can be run easily from within VS using the included nunit3testadapter nuget package or from the command line using the included bash script `test.sh`.

From VS simply navigate to Test Explorer and run or debug the tests you'd like to examine.

Tests can be run all at once or one at a time in VS.

From command line the `test.sh` script accepts 3 parameters

```bash
test.sh <PLATFORM> <TYPE> <COVERAGE>
```

### Writing Tests

While not always fun, we encourage writing unit tests for any backend code changes. This will ensure the change is functioning as you intended and that future changes dont break the expected behavior.

> We currently require 80% coverage on new code when submitting a PR
{.is-info}

If you have any questions about any of this, please let us know.

# Translation

Readarr uses a self hosted open access [Weblate](https://translate.servarr.com) instance to manage its json translation files. These files are stored in the repo at `src/NzbDrone.Core/Localization`

## Contributing to an Existing Translation

Weblate handles synchronization and translation of strings for all languages other than English. Editing of translated strings and translating existing strings for supported languages should be performed there for the Readarr project.

The English translation, `en.json`, serves as the source for all other translations and is managed on GitHub repo.

## Adding a Language

Adding translations to Readarr requires two steps

- Adding the Language to weblate
- Adding the Language to Readarr codebase

## Adding Translation Strings in Code

The English translation, `src/NzbDrone.Core/Localization/en.json`, serves as the source for all other translations and is managed on GitHub repo. When adding a new string to either the UI or backend a key must also be added to `en.json` along with the default value in English. This key may then be consumed as follows:

> PRs for translation of log messages will not be accepted
{.is-warning}

### Backend Strings

Backend strings may be added utilizing the Localization Service `GetLocalizedString` method

```dotnet
private readonly ILocalizationService _localizationService;

public IndexerCheck(ILocalizationService localizationService)
{
  _localizationService = localizationService;
}
        
var translated = _localizationService.GetLocalizedString("IndexerHealthCheckNoIndexers")
```

### Frontend Strings

New strings can be added to the frontend by importing the translate function and using a key specified from `en.json`

```js
import translate from 'Utilities/String/translate';

<div>
  {translate('UnableToAddANewIndexerPleaseTryAgain')}
</div>
```
