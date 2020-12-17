# @signaldevs/slack-orb

[![CircleCI Build Status](https://circleci.com/gh/signaldevs/slack-orb.svg?style=shield "CircleCI Build Status")](https://circleci.com/gh/signaldevs/slack-orb) [![CircleCI Orb Version](https://img.shields.io/badge/endpoint.svg?url=https://badges.circleci.io/orb/signaldevs/slack-orb)](https://circleci.com/orbs/registry/orb/signaldevs/slack-orb) [![GitHub License](https://img.shields.io/badge/license-MIT-lightgrey.svg)](https://raw.githubusercontent.com/signaldevs/slack-orb/master/LICENSE)


A CircleCI orb for sending notifications to a Slack channel. This orb is a light wrapper around the [`circleci/slack`](https://circleci.com/developer/orbs/orb/circleci/slack) orb is order to abstract away some common configuration.

## Resources

* [CircleCI Orb Registry Page](https://circleci.com/orbs/registry/orb/signaldevs/slack-orb) - The official registry page of this orb for all versions, executors, commands, and jobs described.
* [CircleCI Orb Docs](https://circleci.com/docs/2.0/orb-intro/#section=configuration) - Docs for using and creating CircleCI Orbs.

## Issues

We welcome [issues](https://github.com/signaldevs/slack-orb/issues) to be opened, but first, please check out the [CircleCI-Public/slack-orb issues](https://github.com/CircleCI-Public/slack-orb/issues) page as this repo is just a light wrapper around that orb.

## Getting Started

### Adding Commands

 - [Reusable Commands](https://circleci.com/docs/2.0/reusing-config/#authoring-reusable-commands)
 - [Orb Author Intro](https://circleci.com/docs/2.0/orb-author-intro/#section=configuration)
 - [How to author commands](https://circleci.com/docs/2.0/reusing-config/#authoring-reusable-commands)

### Usage Examples

Easily author and add [Usage Examples](https://circleci.com/docs/2.0/orb-author/#providing-usage-examples-of-orbs) to the `src/examples` directory.

Each _YAML_ file within this directory will be treated as an orb usage example, with a name which matches its filename.

Usage examples should contain clear use-case based example configurations for using the orb.

### How to Publish
* Create and push a branch with your new features.
* When ready to publish a new production version, create a Pull Request from fore _feature branch_ to `master`.
* The title of the pull request must contain a special semver tag: `[semver:<segement>]` where `<segment>` is replaced by one of the following values.

| Increment | Description|
| ----------| -----------|
| major     | Issue a 1.0.0 incremented release|
| minor     | Issue a x.1.0 incremented release|
| patch     | Issue a x.x.1 incremented release|
| skip      | Do not issue a release|

Example: `[semver:major]`

* Squash and merge. Ensure the semver tag is preserved and entered as a part of the commit message.
* On merge, after manual approval, the orb will automatically be published to the Orb Registry.


## License

MIT


