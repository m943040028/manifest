# Fuchsia Manifest

Contains the jiri manifests for Fuchsia.

## Prerequisites

On Ubuntu:

 * `sudo apt-get install golang git-all build-essential curl`

On Mac:

 * Install Xcode Command Line Tools
 * `brew install golang`

## Creating a new checkout

Fuchsia uses the `jiri` tool to manage repositories
[https://fuchsia.googlesource.com/jiri](https://fuchsia.googlesource.com/jiri).
This tool manages a set of repositories specified by a manifest.  The bootstrap
procedure requires that you have Go 1.6 or newer and Git installed and on your
PATH.  To create a new Fuchsia checkout in a directory called `fuchsia` run the
following commands. The `fuchsia` directory should not exist before running
these steps.

```
curl -s https://raw.githubusercontent.com/fuchsia-mirror/jiri/master/scripts/bootstrap_jiri | bash -s fuchsia
cd fuchsia
export PATH=`pwd`/.jiri_root/scripts:$PATH
jiri import fuchsia https://fuchsia.googlesource.com/manifest
jiri update
```

## Building Fuchsia

First, you need to build Magenta:

```
(cd magenta; ./scripts/build-magenta-x86-64)
```

Next, you need to build a sysroot:

```
./scripts/build-sysroot.sh -c -t x86_64
```

Finally, you can build Fuchsia using these commands:

```
./packages/gn/gen.py
./buildtools/ninja -C out/debug-x86-64
```

[Googlers only] If you have goma installed, use these alternative commands for faster builds:

```
./packages/gn/gen.py --goma
./buildtools/ninja -j1000 -C out/debug-x86-64
```

You can set target cpu by using --target\_cpu

```
./packages/gn/gen.py --target_cpu=aarch64
./buildtools/ninja -C out/debug-aarch64
```

You can configure the set of modules that `gen.py` uses with the `--modules`
argument. After running `gen.py` once, you can do incremental builds using
`ninja`.

### Running Fuchsia

These commands will create an `out/debug-{arch}/user.bootfs` file. After you've got
Magenta building (see [Magenta's getting started
guide](https://fuchsia.googlesource.com/magenta/+/HEAD/docs/getting_started.md)),
you can boot using `user.bootfs` with the following command:

```
cd magenta
./scripts/run-magenta-x86-64 -x ../out/debug-x86-64/user.bootfs
./scripts/run-magenta-arm64 -x ../out/debug-aarch64/user.bootfs
```

If you want a graphical console, add the `-g` flag.

Then, when Fuchsia has booted and started an MXCONSOLE, you can run programs!

For example, to receive deep wisdom, run:

```
fortune
```

### Working without altering your PATH

If you don't like having to mangle your environment variables, and you want `jiri`
to "just work" depending on your current working directory, there's a shim for that.

```
sudo cp .jiri_root/scripts/jiri /usr/local/bin
sudo chmod 755 /usr/local/bin/jiri
```

That script is just a wrapper around the real `jiri`.  It crawls your parent directory
structure until it finds a `.jiri_root` directory, then executes the `jiri` it finds there.

## Submitting changes

To submit a patch to Fuchsia, you may first need to generate a cookie to
authenticate you to Gerrit.  To generate a cookie, log into Gerrit and click the
"Generate Password" link at the top of https://fuchsia.googlesource.com. Then,
copy the generated text and execute it in a terminal.

Once authenticated, follow these steps to submit a patch to Fuchsia:

```
# create a new branch
git checkout -b branch_name

# write some awesome stuff, commit to branch_name
vim some_file ...
git commit ...

# upload the patch to gerrit
git push origin HEAD:refs/for/master

# once the change is landed, clean up the branch
git branch -d branch_name
```

See the Gerrit documentation for more detail:
[https://gerrit-documentation.storage.googleapis.com/Documentation/2.12.3/intro-user.html#upload-change](https://gerrit-documentation.storage.googleapis.com/Documentation/2.12.3/intro-user.html#upload-change)

## Cross-repo changes

Changes in two separate repos will be automatically tracked for you by Gerrit
if you use the same topic.

```
# make and commit the first change, upload it with topic 'add_feature_foo'
cd fuchsia/bin/fortune
git checkout -b new add_feature_foo
vim foo_related_files ...
git commit ...
git push origin HEAD:refs/for/master%topic=add_feature_foo

# make and commit the second change in another repository
cd fuchsia/build
git checkout -b new add_feature_foo
vim more_foo_related_files ...
git commit ...
git push origin HEAD:refs/for/master%topic=add_feature_foo

# after the changes are reviewed, approved and submitted, cleanup the local branch
cd fuchsia/bin/fortune
git branch -d add_feature_foo

cd fuchsia/build
git branch -d add_feature_foo
```

Multipart changes are tracked in gerrit via topics, will be tested together, and
can be landed in Gerrit at the same time with `Submit Whole Topic`.

## Building toolchain

To build Fuchsia toolchain, you can use the `scripts/build-toolchain.sh`
script as follows:

```
./scripts/build-toolchain.sh
```

After the build finishes, you can find the built toolchain in `out/toolchain`.

Please note that build can take significant amount of time, especially on slower
machines.

When the sources get updated, you can rebuild the toolchain using the following
commands:

```sh
# update
jiri update

# cleanup the artifacts from the previous build and rebuild the
toolchain
./scripts/build-toolchain.sh -c
```
