Repode: Dependencies as code
============================

If you're developing a software project with third-party dependencies,
you can usually rely on a package manager such as pip, Maven, or Cargo
to install dependencies. If you need to install a modified version of
a dependency, though, this gets suddenly much harder:

 * In ecosystems that provide you with prebuilt packages, such as pip
   (wheels) and Maven (JARs), you now need to handle the process of
   building the packages yourself.
 * Even in ecosystems that always build from source, such as Cargo, 
   you now need to maintain a source repository to build from. If you're
   using a patch from an open pull request, you may not want to point
   directly at a specific VCS branch/commit - you might want to apply
   the patch to the latest stable release, or carry the patch forwards
   as new releases happen. You may also need to run with _multiple_
   small patches.

If you want your build process to be in any way reproducible /
accessible to other developers, you now need to host a package repo and
maintain a source code fork with the patches you want, and you need to
manually rebase the repository and re-upload packages every time an
upstream release happens.

This is a huge discontinuity in difficulty for wanting to run with a
small patch that hasn't yet been released, and it makes open-source
collaboration harder - it effectively provides an incentive to avoid
contributing changes to your dependencies and testing them locally, as
well as to avoid testing changes from others that aren't in an official
release, and at worst it provides an incentive towards local workarounds
/ monkey-patches and pinned versions.

**Repode** aims to solve this problem by letting you write a textual
specification of the dependencies you want. You can specify which
patches you want (by pointing to either a VCS commit / pull request URL
or to a patch file), and Repode will automatically apply those patches
on top of a stable release and rebuild the code. Repode will then
dynamically generate a repository for the dependencies described by the
file, so that your local build can use it.

Developers can also point their interactive sessions or IDEs at a
Repode-generated mirror and easily install the same packages that would
get installed into production builds.

We're initially building Repode for the Python ecosystem (pip and
wheels), for the use case of relatively small projects that have been
happily using a requirements.txt file but find they need a few patches.
We hope to expand to other ecosystems like Java/Maven soon.

Repode is short for "declarative repositories" or "repositories as
code." The "infrastructure as code" philosophy has produced great
results for understandable, reproducible systems, avoiding the problem
of a machine that has all the right software and configuration but that
no one knows how to rebuild. Similarly, by dynamically generating a
repository from a high-level textual specification that can be checked
in and code reviewed alongside your source code, Repode avoids the
problem of a repository that has all the right dependencies but that no
one knows how to rebuild or keep up to date.
