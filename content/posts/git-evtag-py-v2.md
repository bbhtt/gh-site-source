---
title: "A Python reimplementation of git-evtag"
date: 2026-05-24T12:11:54+05:30
---

I recently tagged v2.0.0 of `git-evtag-py` and I realised I never
introduced it anywhere. I will talk about it a bit here.

<!--more-->

Git uses SHA-1 for pretty much everything, however the algorithm was
[never selected](https://www.youtube.com/watch?v=4XpnKHJAok8&t=3380s)
with security in mind but primarily to ensure reliability and
integrity. The fact that it provided some amount of cryptographic
security for free was entirely coincidental at the time. The rest of
the "security" comes from the fact that Git is a distributed VCS,
primarily used to store source code in the open and any malicious
change in a repository or something that tries to keep the hash
unchanged usually has a high chance of getting spotted. This is
somewhat better than other uses like TLS certificates but still an
inherent weakness considering freestart collisions against SHA1 have
been [known since 2014](https://web.archive.org/web/20151008144309/https://sites.google.com/site/itstheshappening/).

The practical consequence of this for signed git tags is a bit
important. A `git tag -s` creates a signed tag in git. It produces a
GPG (or SSH) signature over the tag object, which in turn points to a
commit. The signature covers that commit hash which again is computed
using SHA1. A collision here means an attacker will be able to produce
two different blob objects with the same hash and can swap out a source
file across revisions that references the blob, without changing any of
the hashes in the tree above it. This completely bypasses the security
guarantees imposed by the tag signature that Git appended.

The original [git-evtag project](https://github.com/cgwalters/git-evtag/#git-evtag),
tries to solve this problem by computing a SHA-512 checksum over the
complete contents of every object reachable from a commit i.e. the
commit itself, its tree, every blob, and recursively over every
submodule. This checksum is then appended to the tag message before
signing with GPG. So verifying the tag now means verifying both the
GPG signature and recomputing and re-verifying the EVTag SHA-512 as
well. The README there explains the Git-EVTag v0 algorithm, but
essentially it feeds every reachable git object's raw header and
body into a single running SHA-512 digest.

Upstream provides a primary C implementation, a Rust and a Python
implementation of this. The Python implementation, however, lacks most
of the functionality available in the other variants and it performs
significantly worse in practice. It also appears to be effectively
unmaintained, and has received no updates in over four years.

I discovered the project while researching the same topic discussed in
the introduction, and as far as I remember it was at a time when I
needed to create a release tag for one of my own projects.

The C implementation was not packaged for my distribution, and it also
depended on `libgit2`  which I am not a
fan of for various reasons. More importantly it was [missing](https://github.com/cgwalters/git-evtag/issues/9)
the `-m` argument at the time to define commit messages on the
commandline. So, I wanted something for my needs and Python was my
natural choice. I wrote the initial version last year, heavily using
Git _as-a-subprocess_ and added the missing features to it. I obviously
saw the upstream throwaway Python implementation and naturally a small
amount of code was inadvertently copied over. So I used the same
licence and added a copyright attribution to the original author. Today
that code lives in the `ChecksumProcessor` and `GitProcessor` classes
and is < 100 LoC of the total ~800 LoC of the script.

The initial version worked quite well and supported the features I
needed to use it. It was almost 10x faster than the upstream Python
implementation on the `torvalds/linux.git` at the `v6.15` tag. But
it was still slightly slower that the upstream C implementation by
a couple of seconds.

The main difference in the initial version compared to the upstream
Python implementation was that I had introduced a batch processing
through the `GitBatchProcessor` class, which kept a single
`git cat-file --batch` process alive for the duration of the walk
rather than spawning a new subprocess per object. This alone
made it faster by more than 10 times. The upstream implementations
also required submodules to be checked out when computing the checksum
which mine didn't as it was using `git submodule` to clone them as
needed.

Since it was written rather quickly in a day and probably a couple of
days more for getting the tests to work in CI with my GPG key, it had
some issues. The submodule cloning and certain other operations could
modify state of a checked-out repository when trying to just print
the EVTag checksum.

Proper subprocess management, and error handling
was also missing. There was a bunch of `raise`-es thrown around
where it mattered and exceptions not managed at all where it didn't.

I really don't like this style of error management when writing certain
types of Python scripts like this. I think functions should try to
handle certain exceptions where it makes sense, log a meaningful error
message, and return something so that the caller can act on it, rather
than letting exceptions pop up or silently swallowing them.

The v2 rewrite I completed last week, completes these wishlist items
I had. I managed to also make it slightly faster than the upstream C
implementation.

It centralises all subprocess management, handles CalledProcessError,
logs meaningful messages including stdout/stderr and propagate failures
cleanly to the caller.

The checksum logic is now split into three classes, namely,
`ChecksumProcessor` which covers the SHA-512 state and per-type
byte/count statistics, `GitBatchProcessor` which is a context manager
wrapping the single `git cat-file --batch` process and the
`GitProcessor` class which does the recursive tree walk, delegating
appropriate duties to the earlier classes.

There are also a couple of micro-optimisations thrown here and there
but the main thing that made it closer and slightly faster than the
upstream C implementation is that the tree entries are parsed directly
from the raw binary object content in the `git cat-file --batch`
stream, rather than spawning a separate `git ls-tree` process per-tree
object. I have profiled it lightly here and there and the current
bottlenecks are entirely from I/O of reading object content
through `read()` and I don't think it can be optimised further at least
with Python.

```
         2663204 function calls (2656485 primitive calls) in 8.389 seconds

   Ordered by: cumulative time
   List reduced from 983 to 20 due to restriction <20>

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     39/1    0.002    0.000    8.389    8.389 {built-in method builtins.exec}
        1    0.000    0.000    8.389    8.389 git-evtag:1(<module>)
        1    0.000    0.000    8.343    8.343 git-evtag:756(main)
        1    0.000    0.000    8.340    8.340 git-evtag:738(run_compute)
        1    0.000    0.000    8.339    8.339 git-evtag:675(compute_checksum)
        1    0.000    0.000    8.309    8.309 git-evtag:552(checksum_repo)
   5895/1    0.059    0.000    8.308    8.308 git-evtag:515(checksum_tree)
    88844    0.117    0.000    7.923    0.000 git-evtag:483(checksum_object)
    94739    0.229    0.000    6.274    0.000 git-evtag:434(get_object)
   189510    4.695    0.000    4.695    0.000 {method 'read' of '_io.BufferedReader' objects}
   189478    0.093    0.000    1.633    0.000 git-evtag:374(update)
   189478    1.524    0.000    1.524    0.000 {method 'update' of '_hashlib.HASH' objects}
    94739    1.104    0.000    1.104    0.000 {method 'readline' of '_io.BufferedReader' objects}
     5895    0.100    0.000    0.170    0.000 git-evtag:346(parse_tree_content)
    94739    0.152    0.000    0.152    0.000 {method 'flush' of '_io.BufferedWriter' objects}
481490/481391    0.046    0.000    0.046    0.000 {built-in method builtins.len}
     61/7    0.000    0.000    0.046    0.007 <frozen importlib._bootstrap>:1360(_find_and_load)
     61/7    0.000    0.000    0.045    0.006 <frozen importlib._bootstrap>:1308(_find_and_load_unlocked)
     51/7    0.000    0.000    0.045    0.006 <frozen importlib._bootstrap>:914(_load_unlocked)
     32/7    0.000    0.000    0.045    0.006 <frozen importlib._bootstrap_external>:753(exec_module)
```

Note that the profiler itself adds ~1s of penalty here.

I haven't looked too much at the C implementation and haven't profiled
it so I have no idea why mine ends up being just slightly faster than
it.

There is also a hidden `--in-place` argument which tries to compute
the checksum on a checked-out and otherwise "ready" repository. It does
not explicitly attempt to modify any state so submodule cloning is
skipped but I am not certain that it wouldn't. So it comes with a
warning while the default being creating a local clone in a tmpdir to
compute the checksum. The signing mode skips this as obviosuly it needs
to modify state of the current repository.

On the same `torvalds/linux.git` at the `v6.15` tag, the in-place mode
of `git-evtag-py` completes the checksum walk in around 7.7 seconds on
average for 10 runs, while the upstream C implementation takes around
8.7 seconds and the upstream Python script takes around 152 seconds.
Without `--in-place`, `git-evtag-py` takes around 26 seconds. At
`v6.15`, the kernel repository contains roughly 90k objects
totalling about 1.5 GB of content.

I've also added a bunch of new unit tests and integration tests that
can work offline (once someone supplies a local copy my GPG key) etc.
with this rewrite. They can also run without network but will require
the GPG key in a file and the tests running inside the unshallowed
repository.

I don't have much use of it, but it was a nice exercise. I have some
plans to do a "pure-Python" version of it using `dulwich` but I am not
really familiar with it and given some other misadventures with that
module I am not very confident in using it either. So let's see.
