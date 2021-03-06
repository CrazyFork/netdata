# Description of CI build configuration

## Variables needed by travis

- GITHUB_TOKEN - GitHub token with push access to repository
- DOCKER_USERNAME - Username (netdatabot) with write access to docker hub repository
- DOCKER_PASSWORD - Password to docker hub
- encrypted_decb6f6387c4_key - Something to do with package releasing (soon to be deprecated)
- encrypted_decb6f6387c4_iv - Something to do with package releasing (soon to be deprecated)
- OLD_DOCKER_USERNAME - Username used to push images to firehol/netdata # TODO: remove after deprecating that repo
- OLD_DOCKER_PASSWORD - Password used to push images to firehol/netdata # TODO: remove after deprecating that repo

## Stages

### Test

Unit tests and coverage tests are executed here. Stage consists of 2 parallel jobs:
  - C tests - executed every time
  - coverity test - executed only when pipeline was triggered from cron

### Build

Stage is executed every time and consists of 5 parallel jobs which execute containerized and non-containerized
installations of netdata. Jobs are run on following operating systems:
  - OSX
  - ubuntu 14.04
  - ubuntu 16.04 (containerized)
  - CentOS 6 (containerized)
  - CentOS 7 (containerized)
  - alpine (containerized)

### Release

This stage is executed only on "master" brach and allows us to create a new tag just looking at git commit message.
It also has an option to automatically generate changelog based on GitHub labels and sync it with GitHub release.
For the sake of simplicity and to use travis features this stage cannot be integrated with next stage.

Releases are generated by searching for a keyword in last commit message. Keywords are:
 - [patch] or [fix] to bump patch number
 - [minor], [feature] or [feat] to bump minor number
 - [major] or [breaking change] to bump major number
All keywords MUST be surrounded with square braces.
Alternative is to push a tag to master branch.

### Packaging

This stage is executed only on "master" branch and it is separated into 3 jobs:
  - Update Changelog/Create release
  - Nightly tarball and self-extractor build
  - Nightly docker images

##### Update Changelog/Create release

This job is running one script called `releaser.sh`, which is responsible for a couple of things. First of all it 
automatically updates our CHANGELOG.md file based on GitHub features (mostly labels and pull requests). Apart from 
that it can also create a new git tag and a github draft release connected to that tag.
Releases are generated by searching for a keyword in last commit message. Keywords are:
 - `[netdata patch release]` to bump patch number
 - `[netdata minor release]` to bump minor number
 - `[netdata major release]` to bump major number
All keywords MUST be surrounded with square brackets.

Alternatively new release can be also created by pushing new tag to master branch.

##### Nightly tarball and self-extractor build AND Nightly docker images

As names might suggest those two jobs are responsible for nightly netdata package creation and are run every day (in
cron). Combined they produce:
  - docker images
  - tar.gz archive (soon to be removed)
  - self-extracting package

Currently "Nightly tarball and self-extractor build" is using old firehol script and it is planed to be replaced with
new design.

##### Nightly changelog generation

This job is responsible for regenerating changelog every day by executing `generate_changelog.sh` script. This is done
only once a day due to github rate limiter.

