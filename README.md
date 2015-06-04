# semver-vcs

[![Travis](https://img.shields.io/travis/ajoberstar/semver-vcs.svg?style=flat-square)](https://travis-ci.org/ajoberstar/semver-vcs)
[![Maintainer Status](http://stillmaintained.com/ajoberstar/semver-vcs.svg)](http://stillmaintained.com/ajoberstar/semver-vcs)
[![Download](https://api.bintray.com/packages/ajoberstar/libraries/org.ajoberstar%3Asemver-vcs/images/download.svg)](https://bintray.com/ajoberstar/libraries/org.ajoberstar%3Asemver-vcs/_latestVersion)
[![GitHub license](https://img.shields.io/github/license/ajoberstar/semver-vcs.svg?style=flat-square)](https://github.com/ajoberstar/semver-vcs/blob/master/LICENSE)

## Why do you care?

### Get your version number out of my build file!

Most build tools and release systems require you to hardcode a version number into
a file in your source repository. This results in commit messages like "Bumping
version number.". Even if you don't have to do this manually, your release plugin
probably modifies your build file and commits the new version.

However, your version control system (VCS) already contains tags with a version
number pointing to a specific commit. Git illustrates the power of this with
the `git describe` command that creates a version number based on the
amount of change since the previous tag (e.g. v0.1.0-22-g26f678e).

Your VCS also contains branches that indicate specific stages of development
or indicate maintenance for a specific subset of versions.

With this much information available to the VCS, there's very little the user
should have to provide to get the next version number. And it certainly
doesn't need to be hardcoded anywhere.

### What does this version number mean?

[Semantic versioning](http://semver.org) is the best answer to this question so far.
It specifies a pretty stringent meaning for what a consumer of an API should expect
based on the difference between two versions numbers.

Additionally it describes methods for encoding pre-release and build-metadata and
how those should be sorted by tools.

With that specification and some conventions related to encoding your stage of
development into the pre-release information, you can end up with a very
easy to understand versioning scheme.

For example, this API's scheme includes four stages:

- **final** (e.g. `1.0.0`) the fully-tested version ready for end-user consumption
- **rc** (e.g. `1.1.0-rc.1`) release candidates, versions believed to be ready for release with final testing
- **milestone** (e.g. `1.1.0-milestone.4`) versions containing a significant piece of functionality on the road
to the next version
- **dev** (e.g. `1.1.0-dev.2` or `1.1.0-milestone.4.dev.6`) development versions happening in-between more
formally defined stages (this is a *floating* stage, in semver-vcs parlance)

## What is it?

TBD

### Current Support

**VCSs**

* [Git](http://git-scm.com/) (through [grgit](https://github.com/ajoberstar/grgit)) (in progress)

**Tooling**

* [Gradle](http://gradle.org/) (in progress)

## Usage

**NOTE:** *All* semver-vcs modules require Java 8 (or higher).

* [Release Notes](https://github.com/ajoberstar/semver-vcs/releases)
* [Full Documentation](https://github.com/ajoberstar/semver-vcs/wiki)

### Gradle & Git

Apply the plugin:

```groovy
buildscript {
	repositories { jcenter() }
	dependencies { classpath 'org.ajoberstar:semver-vcs-gradle-grgit:<version>' }
}

apply plugin: 'org.ajoberstar.semver-vcs-grgit'

semver {
	// optionally configure how the version will be calculated
}
```

See [SemverVcsExtension](http://ajoberstar.org/semver-vcs/docs/semver-vcs-gradle-base/groovydoc/org/ajoberstar/semver/vcs/gradle/SemverExtension.html)
for details on the configuration options.

When you run Gradle, pass in any of the following properties to influence the version being inferred:

* `semver.scope` - one of `major`, `minor`, or `patch` to specify which component of the previous release should be incremented
* `semver.stage` - (by default, one of `final`, `rc`, `milestone`, or `dev`) to specify what phase of development you are in
* `semver.base` - for your first version, if you don't want to start from `0.0.0`
* `semver.force` - if you're in a bind and just need a specific version to be used

For example if the previous release was `1.2.4`:

```
./gradlew build -Psemver.scope=minor -Psemver.stage=milestone
Inferred version 1.3.0-milestone.1
...
```

### Direct API Usage

The four basic steps are:

1. Construct a [Version](https://github.com/zafarkhaja/jsemver/blob/master/src/main/java/com/github/zafarkhaja/semver/Version.java) to use as a base if one isn't found in the VCS.
1. Construct a [Vcs](http://ajoberstar.org/semver-vcs/docs/semver-vcs-api/javadoc/org/ajoberstar/semver/vcs/Vcs.html)
using one of the available providers.
1. Build a [Versioner](http://ajoberstar.org/semver-vcs/docs/semver-vcs-api/javadoc/org/ajoberstar/semver/vcs/Versioner.html)
which is a function of `(Version, Vcs) -> Version`. This does all of the work of inferring the version. The
[Versioners](http://ajoberstar.org/semver-vcs/docs/semver-vcs-api/javadoc/org/ajoberstar/semver/vcs/Versioners.html) class
provides some common `Versioner` implementations that can be composed as needed.
1. Call the `Versioner` with the base `Version` and the `Vcs`.

For example:

```java
import com.github.zafarkhaja.semver.Version;
import org.ajoberstar.semver.vcs.*;
import org.ajoberstar.semver.vcs.grgit.GrgitVcs;
import org.ajoberstar.grgit.Grgit;

Version base = Version.forIntegers(0, 0, 0);
Vcs vcs = new GrgitVcs(Grgit.open()));
Versioner versioner = Versioners.forScopeAndStage(Scope.MINOR, Stage.finalStage());
Version inferred = versioner.infer(base, vcs);
```

## Implementing

### Modules

- [semver-vcs-api](http://ajoberstar.org/semver-vcs/docs/semver-vcs-api/javadoc) - Base API that tooling should use.
- [semver-vcs-grgit](http://ajoberstar.org/semver-vcs/docs/semver-vcs-grgit/groovydoc) - Implementation of a grgit backend.
- [semver-vcs-gradle-base](http://ajoberstar.org/semver-vcs/docs/semver-vcs-gradle-base/groovydoc) - Base Gradle plugin that
will calculate the project's version (given a VCS impl).
- [semver-vcs-gradle-grgit](http://ajoberstar.org/semver-vcs/docs/semver-vcs-gradle-grgit/groovydoc) - Extension of the base Gradle plugin to automatically configure a grgit VCS.

### Implementing a Vcs

A `Vcs` implementation will literally just need to implement the
[Vcs](http://ajoberstar.org/semver-vcs/docs/semver-vcs-api/javadoc/org/ajoberstar/semver/vcs/Vcs.html) interface.

See the Grgit module in this repo for further guidance.

### Implementing a Tool

See the example in *Direct API Usage* above for a basic example and the implementation of the
Gradle modules in this repo as further guidance.

## Questions, Bugs, and Features

Please use the repo's [issues](https://github.com/ajoberstar/semver-vcs/issues)
for all questions, bug reports, and feature requests.

## Contributing

Contributions are very welcome and are accepted through pull requests.

Smaller changes can come directly as a PR, but larger or more complex
ones should be discussed in an issue first to flesh out the approach.

If you're interested in implementing a feature on the
[issues backlog](https://github.com/ajoberstar/semver-vcs/issues), add a comment
to make sure it's not already in progress and for any needed discussion.

## Acknowledgements

Thanks to [everyone](https://github.com/ajoberstar/gradle-git/graphs/contributors)
who contributed to previous iterations of this library and to
[Zafar Khaja](https://github.com/zafarkhaja) for the very helpful
[jsemver](https://github.com/zafarkhaja/jsemver) library.
