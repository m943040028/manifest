# Fuchsia Manifest

Contains the jiri manifests for Fuchsia.

## Prerequisites

On Ubuntu:

 * `sudo apt-get install golang git-all build-essential curl`

On Mac:

 * `TODO(abarth): Figure out what needs to be installed.`

## Creating a new checkout

Fuchsia uses the `jiri` tool to manage repositories
[https://github.com/vanadium/go.jiri](https://github.com/vanadium/go.jiri).
This tool manages a set of repositories specified by a manifest.  The bootstrap
procedure requires that you have Go 1.6 or newer and Git installed and on your
PATH.  To create a new Fuchsia checkout in a directory called `fuchsia` run the
following commands. The `fuchsia` directory should not exist before running
these steps.

```
curl -s https://raw.githubusercontent.com/vanadium/go.jiri/master/scripts/bootstrap_jiri | bash -s fuchsia
cd fuchsia
export PATH=`pwd`/.jiri_root/scripts:$PATH
jiri import fuchsia https://fuchsia.googlesource.com/manifest
jiri update
```

## Building Fuchsia

Currently you can build Fuchsia using these commands:

```
./packages/gn/gen.py
./buildtools/ninja -C out/debug-x86-64

```
You can set target cpu by using --target\_cpu

```
./packages/gn/gen.py --target_cpu=aarch64
./buildtools/ninja -C out/debug-aarch64

```

You can configure the set of modules that `gen.py` uses with the `--modules`
argument. After running `gen.py` once, you can do incremental builds using
`ninja`.

These commands will create an `out/debug-{arch}/user.bootfs` file.  After you've got
Magenta building (see [Magenta's getting started
guide](https://fuchsia.googlesource.com/magenta/+/HEAD/docs/getting_started.md)),
you can boot using `user.bootfs` with the following command:

```
cd magenta
./scripts/run-magenta-x86-64 -x ../out/debug-x86-64/user.bootfs
./scripts/run-magenta-arm64 -x ../out/debug-aarch64/user.bootfs

```

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
authenticate you to GoogleSource (you know this is necessary if ```jiri cl mail```
prompts you for a username/password).  To generate a cookie, log into
GoogleSource and click the "Generate Password" link at the top of
https://fuchsia.googlesource.com. Then, copy the generated text and execute it
in a terminal.

Once authenticated, follow these steps to submit a patch to Fuchsia:

```
# create a new change (makes a new local branch, checks it out)
jiri cl new branch_name

# write some awesome stuff, commit to branch_name
vim some_file ...
git commit ...

# upload the patch to gerrit
jiri cl mail

# once the change is landed, clean up the branch
jiri cl cleanup branch_name
```

## Cross-repo changes

Changes in two separate repos will be automatically tracked for you by `jiri`
if you use the same branch name.

```
# make and commit the first change
cd fuchsia/bin/fortune
jiri cl new add_feature_foo
vim foo_related_files ...
git commit ...

# make and commit the second change, with the same branch name ('add_feature_foo')
cd fuchsia/build
jiri cl new add_feature_foo
vim more_foo_related_files ...
git commit ...

# upload both changes to gerrit
jiri cl mail

# after the changes are reviewed, approved and submitted, cleanup the local branch
jiri cl cleanup add_feature_foo
```

Multipart changes are tracked in gerrit via topics, are tested together, and
can be landed in Gerrit at the same time with `submit whole topic`.

## Building toolchain

Create a new checkout, while this is similar to the Fuchsia checkout, there are
some notable differences in particular around environment setup.

```sh
curl -s https://raw.githubusercontent.com/vanadium/go.jiri/master/scripts/bootstrap_jiri | bash -s sdk
cd sdk
export PATH=`pwd`/.jiri_root/scripts:$PATH
jiri import sdk https://fuchsia.googlesource.com/manifest
jiri update
```

Setup your environment:

```sh
export PATH=`pwd`/buildtools:`pwd`/buildtools/cmake/bin:`pwd`/.jiri_root/bin:$PATH
```

Now, you can build the toolchain using the following commands:

```sh
# create an output directory
mkdir -p out

# generate the ninja build file
toyen -src . -out out packages/root.bp

# build the toolchain
ninja -C out toolchain
```

After the build finishes, you can find the built toolchain in `out/toolchain`.

Please note that build can take significant amount of time, especially on slower
machines.

When the sources get updated, you can rebuild the toolchain using the following
commands:

```sh
# update
jiri update

# cleanup the artifacts from the previous build
ninja -C out toolchain-distclean

# build the toolchain
ninja -C out toolchain
```

## Building sysroot

To build Fuchsia sysroot for either x86-64 or AArch64, you can use the same
checkout as for building the toolchain.

```sh
curl -s https://raw.githubusercontent.com/vanadium/go.jiri/master/scripts/bootstrap_jiri | bash -s sdk
cd sdk
export PATH=`pwd`/.jiri_root/scripts:$PATH
jiri import sdk https://fuchsia.googlesource.com/manifest
jiri update
```

Setup your environment:

```sh
export PATH=`pwd`/buildtools:`pwd`/buildtools/cmake/bin:`pwd`/.jiri_root/bin:$PATH
```

To build the sysroot for both architectures, you can use the following commands:

```sh
# create an output directory
mkdir -p out

# generate the ninja build file
toyen -src . -out out packages/root.bp

# build the sysroot using ninja
ninja -C out sysroot
```

The sysroots for both architectures end up in `out/sysroot`.

When the sources get updated, you can rebuild the toolchain using the following
commands:

```sh
# update
jiri update

# cleanup the artifacts from the previous build
ninja -C out sysroot-distclean

# build the toolchain
ninja -C out sysroot
```

## Clang based toolchain for building Magenta

The Clang toolchain which is used for building Fuchsia cannot build Magenta at
the moment as the kernel build uses a linker script which relies on a behavior
not (yet) supported by LLD linker, which is the default linker used by the Clang
toolchain.

To work around this limitation, in case you want to build Magenta using Clang,
e.g. for development or testing purposes, you can build Binutils and use the GNU
linker together with Clang. To build binutils, you can use the following targets
in your toolchain checkout:

```sh
ninja -C out binutils
```

Now, you can use the Binutils from `out/binutils-<platform>` for building the
Magenta.
