Configuration
=============

Repode uses a YAML configuration file and a cache file, which we'll call
`repode.yaml` and `repode.cache.yaml`.  `repode.yaml` is human-writable,
and `repode.cache.yaml` is human-readable (suitable for code review) but
generally not intended to be manually edited. Both of these files should
be checked in.

This discussion will focus on Repode for the Python (pip) use case, but
the concepts should map to other ecosystems.

`repode.yaml` primarily consists of a list of packages you want to make
available in your repository, in the form that you could list in a
`requirements.txt` file (that is, including version specifiers).
You can also just include `requirements.txt` files. When Repode runs, it
will download the specified packages and create a repository that pip
can use (either as a directory path, or by serving it with an HTTP
server).

Repode needs to handle dependency locking / pinning, because if you ask
it to patch or rebuild anything, it will need to know which version to
apply the patch to. By default, locking is disabled (matching the
behavior of `pip install -r requirements.txt`). If you want to lock your
dependencies (matching the behavior of e.g. `pip-tools`), you can set a
flag in the YAML file, which will store the current dependency versions
into `repode.cache.yaml` and reuse those until you instruct Repode to
upgrade.

Patches and rebuilds
====================

To patch a package, annotate it with a list of patches you want to
apply, which can be
 * a path to a patch file
 * a Git repository URL and ref name, which means to take that specific
   commit
 * a URL for a GitHub pull request or GitLab merge request, either on
   github.com / gitlab.com or a private instance, which means to take
   all the changes from that PR/MR

We encourage you to use the second or third option - contribute patches
upstream when you can, or at least set up a local fork to make it easy
to review the patch in context. This also means developers don't have to
manage patch files by hand and can use tools they're familiar with.

Note that Repode will attempt to apply the changes onto the base stable
release (i.e., the source release it would try to use if there are no
patches), so you don't have to keep rebasing your Git branch just to
keep up with upstream updates.

If the patches fail to apply, the Repode command will exit with an error.
You can fix the issue (probably by rebasing your local branch) or pin an
older version of the source code. This way, even if you are keeping a
patch that isn't accepted upstream, you're not disconnected from
upstream's development processes, and you'll get notified if your patch
is incompatible with upstream changes.

You can also specify an alternative source for Repode to use - for
instance, you may actually want to test against the development branch
of a certain dependency instead of manually picking commits, or you may
have an existing fork with changes. If you specify a source, Repode will
use that instead of looking for an sdist from the upstream repository.

Finally, you can request that Repode rebuild a package from source, even
if there are no changes. You might want to do this if your local build
environment is different (e.g., you might want to recompile NumPy
against a specific BLAS/LAPACK implementation or with particular
compiler flags).

Repode does Python builds using a virtualenv, but using the system's
compiler, native libraries, etc. The virtualenv consists of just the
package's build dependencies, and it is itself installed through Repode,
so that you can customize build dependencies if needed. Packages that
are needed as build dependencies will be recorded in
`repode.cache.yaml`, even if they are not explicitly listed in
`repode.yaml`, which means that enabling package locking also locks
build dependencies like `setuptools` to ensure a reproducible build.

We plan to offer more powerful and self-contained build options (e.g.,
remote builds or containerized builds with declarative system
dependencies) in the future, but using the host environment should be
enough for many use cases.

Caches
======

Patches from external sources are copied into `repode.cache.yaml`, so
that builds can continue working even if the remote source is
unavailable, and so that patches can be reviewed as part of the final
code review if desired.

Repode also uses a per-user cache directory `~/.cache/repode` for
storing downloaded sources and built artifacts, including fetched
prebuilt artifacts from an external repository as well as locally built
artifacts. Caches are keyed off the inputs to the build, i.e., the
portion of the Repode configuration files that describe how to fetch or
build the artifact. This means that the same cache directory can be used
by differing configuration files in different branches (or even two
unrelated projects using Repode), and they will reuse builds whenever
possible.

We intend to make it possible to share or distribute Repode caches
between machines in the future.
