# Contributing

## Reporting issues

We use GitHub for tracking Vanadium issues:
https://github.com/vanadium/issues/issues

## Developer setup

### Contributor license agreement

Before patches can be accepted, contributors must sign the Google Individual
[Contributor License Agreement][cla] (CLA), which can be done online. The CLA
is necessary since contributors own the copyright to their code, even after it
becomes part of the codebase, so permission is required to use and distribute
that code. Contributors don't have to sign the CLA until after a patch has
been submitted for review and a member has approved it, but the CLA must be
signed before the patch is committed into the codebase.

Before starting work on a large contribution, we recommend that you get in
touch through the [issue tracker] with your idea so that other contributors
and authors can help out and possibly guide you.

Contributions made by corporations are covered by a different agreement than
the one above, the [Software Grant and Corporate Contributor License Agreement][corp-cla].

### Credentials

As a new developer you need to gain access to vanadium.googlesource.com
and request a password for accessing it as follows:

1. Go to https://vanadium.googlesource.com, log in with your identity, click
on "Generate Password", and follow the instructions to store the
credentials for accessing vanadium.googlesource.com locally.

2. Go to https://vanadium-review.googlesource.com and log in with your
identity. This will create an account for you in the code review system.

## Machine setup

Make sure your machine has the credentials as described above.

### Prerequisites

If you are developing on Darwin (OS X), you need to have the Xcode Command Line
Tools package installed. You can install this package from a terminal with:

    # Mac-only prerequisite install
    xcode-select --install

On Darwin (OS X), you'll also need the [Homebrew package manager][brew]
installed. You can install it with:

    # Mac-only prerequisite install
    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

The instructions on this page assume that the following software is installed and in your `PATH`:

* curl
* [git]
* [Go version 1.5 or newer][go-install]

### Repositories

To contribute to Vanadium, you will want to setup up a directory which
contains the entire collection of repositories (projects) which make up the
Vanadium universe of language libraries, command line tools, services,
website, example projects, and development tools like [jiri].

#### JIRI_ROOT environment variable

Set the `JIRI_ROOT` environment variable. Use an absolute path that points to a local file system location (remote file system such as NFS are discouraged for performance reasons and to avoid ENOKEY 'git' errors). **Use a directory that does NOT exist**; the following steps will create it.

```bash
# Edit to taste
export JIRI_ROOT=${HOME}/vanadium
```

Be sure to add the `$JIRI_ROOT` environment variable to your profile.

#### Setup script

Run the initial setup script (this might take several minutes):

```bash
curl https://v.io/bootstrap | bash
```

#### Update your PATH

Add `$JIRI_ROOT/devtools/bin` to your `PATH`.

```bash
export PATH=$PATH:$JIRI_ROOT/devtools/bin
```

Be sure to add the modified `$PATH` to your profile.

#### REQUIRED: profile installation

Extra development profiles such as ARM cross-compilation, support for mobile, or web development can be installer using the `jiri` command. To see a list of available profiles run:

```bash
jiri profile list
```

Profiles can be installed with `jiri profile install <profile>` where `<profile>` is replaced with one of the items from the list.

Invocations of the `jiri profile install` command are idempotent.

##### syncbase profile

In order to successfully install/build the Vanadium Go code you will need to install syncbase dependencies with:

```bash
jiri profile install syncbase
```

**The syncbase profile is required.**

##### Node.js profile

**OS X only:** If you are on Darwin (OS X) be sure to have a full Xcode install via the App Store (not just the command line tools).

In order to install the nodejs profile run:

```bash
jiri profile install nodejs
```

## Development

### Building and testing

#### Go

To compile vanadium go codebase, use the following commands:

```bash
# For the host OS and architecture
jiri go <command> <packages>

# For cross-compilation using default toolchain
GOARCH=<arch> GOOS=<os> jiri go <command> <packages>

# For cross-compilation using development profiles
V23_PROFILE=<profile> jiri go <command> <packages>
```

These commands setup the `GOPATH` and other environment variables so the
Vanadium libraries and binaries are built for the desired architecture.

##### Examples

Check that the Go code builds

```bash
jiri go build v.io/...
```

Run all Go tests

```bash
jiri go test v.io/...
```

Install all the Go binaries

```bash
jiri go install v.io/...
```

#### JavaScript

Ensure that you have the required profile set up by running:

    jiri profile install nacl nodejs

This will install dependencies needed to build and run the Vanadium JavaScript
projects and the [Vanadium Chrome extension]. Darwin (OS X) users should be sure
to have a full install of Xcode.

##### Dependencies

To build and test Vanadium JavaScript library, please ensure Vanadium Go
binaries are installed first (they are required for integration
testing).

Build the Go Vanadium binaries.

```bash
jiri go install v.io/...
```

All other dependencies can be satisfied via the default `make` target.

```bash
cd $JIRI_ROOT/release/javascript/core
make
```

##### Testing

Build and test the JavaScript API.

    make test

Remove all JavaScript build artifacts

    make clean

### Git workflow

All of the individual Vanadium projects use `git` for version control. The
"master" branch of each local repository is reserved for tracking the remote
https://vanadium.googlesource.com counterpart. All Vanadium development should
take place on a non-master (feature) branch. Once your code has been reviewed
and approved, it will be merged into the remote master via our code review
system and brought to your local instance via `jiri update`.

**The only way to contribute to master is via the Gerrit code review process.**

To submit a change for review you will need to squash your feature branch into
a single commit and send the patch to Gerrit for review. The command `jiri cl`
simplifies this process and is described in greater detail below.

#### Creating a change

1. Sync the master branch to the latest version of the project:

        jiri update

2. Create a new branch for your change

        # replacing `<branch>` with your branch name
        jiri cl new <branch>

3. Make modifications to the project source code.
4. Stage any changed files for a commit:

        git add <file1> <file2> ... <fileN>

5. Commit your modifications:

        git commit

6. Repeat steps 3-5 as necessary.

#### Syncing a change to the latest version of the project

1. Update all of the local master branches using the `jiri` command:

        jiri update

2. If you are not already on it, switch to the feature branch that corresponds
to the change you are trying to bring up to date with the upstream:

        git checkout <branch>
        git merge master

3. If there are no conflicts, you are done.
4. If there are conflicts:
   * Manually resolve the conflicted files.
   * Stage the resolved files for a commit with `git add <pathspec>...`
   * Commit the resolved files with `git commit`

#### Requesting a review

1. Switch to the branch that corresponds to the change in question.

        git checkout <branch>

2. Submit your change to Gerrit with the `jiri cl` command.

        # <reviewers> is a comma-seperated list of emails or LDAPs
        # Alternatively reviewers can be added via the Gerrit UI
        jiri cl mail -r=<reviewers>

If you are unsure who to add as a reviewer you can leave off the `-r` flag. Our team periodically scans for CLs and a reviewer will come along. If you would rather not wait, feel free to let us know about your change by filing an issue on GitHub.

#### Reviewing a change

1. Follow the link you received in an email notifying you about a
   review request.
2. Add comments as you see fit.
3. When you are finished, click on the "Reply" button to submit your
   comments, selecting the appropriate score.

#### Addressing review comments

1. Switch to the branch that corresponds to the change in question

        git checkout <branch>

2. Modify and commit code as as described [above](#creating-a-change).
3. Be sure to address each review comment on Gerrit.
4. Once you have addressed all review comments be sure to reply at the
   top of the Gerrit UI for the specific patch.
5. Once you have addressed all review comments, you can update the change with
   a new patch using:

        jiri cl mail

#### Submitting a change

1. Work with your reviewers to receive "+2" score. If your change no longer
   applies cleanly due to upstream changes, the reviewer may ask you to rebase
   it. You will need to follow the steps in the section above: ["Syncing a
   change to the latest version of the
   project"](#syncing-a-change-to-the-latest-version-of-the-project) and then
   run `jiri cl mail` again
2. The reviewer will submit your change and it will be merged into the master
   branch.
3. Optional: Delete the feature branch once it has been submitted:

        git checkout master
        jiri cl cleanup <branch>

#### Useful shortcuts

There are several useful shorcuts you can use for quick access to changes and
issues.

*  [v.io/issues](https://v.io/issues): Takes you to the issues list.
*  v.io/i/[num]: Takes you to a specific issue.
*  [v.io/i/new](https://v.io/i/new): Creates a new issue.
*  [v.io/review](https://v.io/review): Takes you to your review dashboard.
*  v.io/c/[num]: Takes you to the review for a specific change.

[git]: http://git-scm.com/
[go-install]: http://golang.org/doc/install
[brew]: http://brew.sh/
[gerrit]: https://vanadium-review.googlesource.com
[Vanadium Chrome extension]: ../tools/vanadium-chrome-extension.md
[jiri]: ../tools/jiri.md
[cla]: https://cla.developers.google.com/about/google-individual?csw=1
[corp-cla]: https://cla.developers.google.com/about/google-corporate?csw=1
[issue tracker]: https://github.com/vanadium/issues/issues
