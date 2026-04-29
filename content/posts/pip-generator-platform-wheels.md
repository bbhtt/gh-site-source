---
title: "Platform wheel support in Flatpak PIP Generator"
date: 2026-04-29T15:21:34+05:30
---

This post summarizes some new features I recently added to the
Flatpak PIP generator, along with some updates to other related
generators.

<!--more-->

### Flatpak PIP Generator

Flatpak PIP Generator is a tool that generates JSON or YAML Flatpak
manifests from PyPI modules. It accepts a package name, a requirements
file, or a pyproject.toml, and produces a manifest that can be included
alongside an application’s Flatpak manifest to enable offline builds.

The tool was originally developed by [Patrick](https://github.com/TingPing)
around 2020, at least according to the git history. Despite being the
most widely used generator in the flatpak-builder-tools repository, it
has historically had only a small number of contributors. I have been
contributing to it since 2023, and after taking over maintenance of
the flatpak-builder-tools repository last year, my level of involvement
increased somewhat. One of the larger changes I introduced last year,
was a codebase [modernisation effort](https://github.com/flatpak/flatpak-builder-tools/pull/453)
that added ruff for formatting and linting, along with mypy for type
checking. It previously was not using any formatter, linter or type
checker, and having these is quite essential nowadays to catch
programmer mistakes.

Historically, the PIP generator relied on universal wheels or source
distributions from PyPI. This design choice favored building from
source to avoid incompatibilities and issues with architecture-specific
wheels. However, since then, the ecosystem has evolved, and many Python
packages now include Rust components, often with Cargo.toml or
Cargo.lock files but without vendored dependencies. As a result,
building these packages introduced an additional step of generating
manifests for Rust dependencies using flatpak-cargo-generator, which
I also maintain. This complicates automated updates to the Python
modules, as the sdist needs to be unpacked for the Cargo lockfile, then
flatpak-cargo-generator needs to be pointed at it to regenerate
the Cargo manifest, and finally, both changed manifests need to be
included in the update.

Therefore, since 2022, one of the [most requested features](https://github.com/flatpak/flatpak-builder-tools/issues/296)
has been support for architecture or platform-specific wheels. Another
[related issue](https://github.com/flatpak/flatpak-builder-tools/issues/202)
was that some PyPI packages do not provide sdists or universal
wheels, causing the generator to fail on them.

Since taking over maintenance, I revisited these issues multiple times
but lacked the time and bandwidth to address them. I was also aligned
with the “build-from-source” approach, so I was not particularly
motivated to prioritise it. However, I do believe that the lack of
support for architecture or platform-specific wheels made Flatpak
packaging difficult for newcomers, especially those unfamiliar with
the complexities of Python build systems.

This changed last month as I finally gathered the motivation to
add the [feature](https://github.com/flatpak/flatpak-builder-tools/pull/524)
to the PIP generator. Within hours of implementing it, someone opened a
PR proposing that platform-specific wheels be preferred over universal
ones, as they can include optimized code paths. I was not entirely
convinced by this approach, so instead I introduced an advanced
option to specify the [artifact policy](https://github.com/flatpak/flatpak-builder-tools/commit/b5df3e23c66cade641bbc38372c06e5edb5d167e)
on a per-module basis.

The platform wheel selection logic operates by querying PyPI metadata
for all available artifacts of a given package version and filtering
them based on compatibility with the target Flatpak runtime. A runtime
argument is required to determine the supported platform and ABI tags.
By default, sources are generated for both x86_64 and aarch64. If
the runtime is unavailable for one architecture, the corresponding
platform tags are inferred on a best-effort basis from the other. For
each supported architecture, the resolver constructs a set of valid
wheel tags and determines the associated Python version. Candidate
wheels are then filtered to those matching the target architecture and
evaluated against the runtime’s Python ABI. A strict compatibility
check is attempted first, with a relaxed fallback if no suitable
candidates are found. Among the compatible candidates, wheels are
ranked by the number of matching tags, and the highest-scoring
artifact is selected as the best fit. The resolver could have bugs
or edge cases and PIP's own resolver is far more complex, but it has
worked correctly, at least for the modules I tried and I haven't
received any bug reports.

Someone also opened an issue requesting support for requirements files
with pinned hashes. The `requirements-parser` module used by the PIP
generator is not very actively maintained and lacks several features,
including parsing pinned hashes - it simply ignores them. I had been
considering writing a replacement from scratch, but given time
constraints, starting a new project from scratch seemed daunting.
Instead, I implemented a [parser](https://github.com/flatpak/flatpak-builder-tools/commit/3f2844feb65672ceee37f2660c9d35ab342d5409)
directly within the PIP generator, which now handles pinned hashes.

This approach was consistent with my earlier work, where I had already
added support for [parsing platform markers](https://github.com/flatpak/flatpak-builder-tools/commit/422e9eca6f8f1f4c7e760aea5e902890e6a5eba0)
which `requirements-parser` also lacks.

This burst of activity also led me to fix some bugs and add some
smaller feature which are visible in the git log.

### Flatpak Node Generator

I also took the opportunity to migrate the Flatpak Node Generator, which
I also maintain (this is becoming a bit ridiculous!), to Python 3.10+.
This enabled me to use some modern language features, particularly
standard type hints.

During this process, I fixed an issue where cache filenames could
[exceed the 255-byte filesystem limit](https://github.com/flatpak/flatpak-builder-tools/pull/515),
as they were derived directly from raw URLs. I introduced a hash-based
caching scheme, along with a backwards-compatible migration
to convert existing cache entries to the new format.

The other issue was that the generator hardcoded an outdated
node-gyp installVersion, with a note indicating it should
eventually be detected automatically. As the value drifted out of
sync with the SDK extension, this began causing build failures. This has
now been addressed, and the version is [automatically detected](https://github.com/flatpak/flatpak-builder-tools/commit/168ee2551783d65ed0778ba70cbb9ae733df557d)
from the installed Node SDK extension.

I also reviewed and merged a PR with [support for PNPM](https://github.com/flatpak/flatpak-builder-tools/pull/511),
which was a long-requested feature, along with several follow-up fixes
to that.

I don’t think many people are using these new features, but please
do test them out!
