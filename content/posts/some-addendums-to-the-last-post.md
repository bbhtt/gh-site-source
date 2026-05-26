---
title: "Some addendums to the last post"
date: 2026-05-22T18:28:08+05:30
---

I wrote the last post in a rush and I wanted to explain my side of
some of the situations I mentioned in there. I think it is missing
some context.

<!--more-->

I wrote an introduction about Freedesktop SDK [earlier](/posts/closing-the-chapter-on-openh264/#a-bit-of-an-introduction)
but it is essentially minimal "distro"
(as in it provides _distributables_) built from source with
elements (_packages_ if you will) ranging from gcc, glibc, bash,
coreutils to gtk3 and mesa. The main repository is structured around
3 branches - `master` and two `release/*` branches.

Each element defines an element kind, which is essentially a BuildStream
plugin that specifies how to process a particular build definition. For
example, a `meson` kind may define commands such as `meson setup`,
`ninja`, and `ninja install`, while a `pyproject` kind may define a
series of `python -m build` commands and so on.

Elements also contain source definitions for the software being built.
which again uses a BuildStream plugin to define how the sources are
fetched, processed, and updated. For example, a `git_repo` source with
a `track: v*` means that when `bst source track` is invoked, the plugin
checks the upstream repository for newer tags matching `v*`. If a newer
matching tag exists, the element's `ref:` field is updated to the
corresponding git commit hash.

Source processing can also be more complex such as plugins may process
`Cargo.lock` or `go.mod` and automatically convert their dependency
information into the YAML syntax understood by BuildStream and
embedded directly into the element definition. This enables offline
builds since BuildStream will first read the element definition, fetch
all the Cargo packages, stage them and then start building the software.

This also means, an element can be updated to a newer upstream software
version automatically by simply invoking `bst source track` on it.

The workflow is quite nicely integrated and can be easily automated
to open MRs for element source updates. gitlab-merge-request-generator
is one such tool that runs `bst source track`, commits, pushes and opens
MRs.

The main Freedesktop SDK repository uses this in all the 3 branches
using daily scheduled workflows. It opens 4 MRs for the `master` branch
and 6 for every supported release branch every day. The limit is
intentionally set to low due to CI resource limitations.

Unlike a distro package, any change to an element or its sources will
cause a change in its cache key, causing a cascading rebuild of any of
its dependent elements, while the rest which were unaffected will
be fetched from the artifact cache server to speed things up. For
example a low level element like GCC which is essentially a dependency
of every element will cause a 8+ hour rebuild, while something like
libnotify which is only used by a handful of other elements will take
much less time. This way, however annoying or CI-costing may it be,
makes sure the entire project stays buildable at all times given any
update thrown at it. This also means a GCC 15 to 16 migration cannot
"just happen" in isolation for the project unless every element part
of the project is buildable with the new GCC, in whatever way that is
achieved.

Also, unlike traditional distributions, where packages are typically
maintained by dedicated maintainers or teams, the entire set of 1000+
elements here is maintained by a handful of 3–4 people, voluntarily
and spread across multiple timezones.

The workflow works reasonably well, but handling 14–15 MRs per day with
only a few maintainers is challenging, especially since some cannot, for
obvious reasons, review things daily or even sometimes are away for
more than a week.

Now elements frequently fail to build because the upstream project
added new dependencies, changed build options, switched build systems,
or broke their historical tagging pattern and created a `v1.38.0-rc` tag.
`gitlab-merge-request-generator` being naive, would open an MR with the
rc update in the last case.

The next step for the maintainers is to then review these bot MRs and
fix the resulting issues, ranging from trivial adjustments such as
excluding `rc` tags to more general build fixes. The repository
operates MRs on a single approval basis and while in an ideal world
every human-made fix to a bot MR would be a separate human-created
MR so that it gets approved by another maintainer, it is hardly possible
or practical to do in a daily basis for 10+ elements. Human beings
naturally gravitate towards "shortcuts" such as directly pushing
the fix to the MR opened by the bot. Again in an ideal world every
such push is supposed to reviewed by another maintainer, but again
this is not practical for the exact same reasons as above.

I am also of the opinion that the “maintainer” role here is effectively
equivalent to the “packager” role in traditional distributions, where in
most cases packagers directly push updates to the git repositories
containing the package definitions. Also, in most cases, there is very
little to review by a second person, beyond routine changes to build
flags, patches being rebased or removed and trivial fixes such as
excluding an `rc` tag when the bot is being "dumb".

As such, it should be reasonable for the project to trust someone to
approve their own build fixes, especially when they are already trusted
to review and push changes directly.

This, the review bottlenecks and often the trivial nature of the changes
were exactly why I approved my own MRs. I am aware that it was a
"shortcut", but I never used it for non-trivial changes. Perhaps the
fact that the number of merged MRs authored by me crossing 999+ in the
GitLab UI — the UI display limit for the counter, illustrates it.

Also - why not? The project has no formal documented policy that
every commit pushed needs to reviewed by a second person, there is
precedent for this, for example some large projects use the exact same
workflow of "self-approving" or "self-merging" and it is the exact same
freedom distro package maintainers enjoy. There is also no "partial
trust", you either trust the person you granted write access to the
repository fully or you don't, in that case, it is better to not grant
them any permission. I think any policy in this regard should take this
points into consideration, otherwise, it has the potential to be really
annoying for projects of this specific category.

There goes the first part about "self-approving" and the next part will
be about the "plugin" controversy.

Many of the BuildStream plugins I described earlier were developed
during early days as part of Freedesktop SDK itself and used to live
in the same repository. As the project grew, they were turned into
generic and reusable and moved to the BuildStream plugin repository -
buildstream-plugins-community which uses `dulwich` for a bunch
of plugins. Now `dulwich` often makes breaking changes between
minor versions, or introduces bugs which heavily breaks the
BuildStream plugins. Such a break often renders the 3 scheduled
pipelines that posts the bot update MRs completely broken and therefore
stalling the entire workflow of the main Freedesktop SDK repository
for days to months.

This had happened last year during 25.08 major release and I had
opened an [MR](https://gitlab.com/BuildStream/buildstream-plugins-community/-/merge_requests/392)
to the plugin repo to fix the issue. After patiently waiting for two
months for a review and sometimes lightly pinging the maintainers for
a review on Matrix, I naturally got annoyed as it meant I had to
manually create large update MRs (cosplaying as the bot) just to keep
things going.

After two months, I got a reply on Matrix saying they would look into
it when they had the chance. Meanwhile, other MRs for plugin
repositories were being merged, while the main FDSDK repository
remained effectively stalled and my fix unreviewed. Perhaps seeing
the angry back and forth on Matrix, nanonyme stepped in and generously
merged my MRs and did a release.

The episode left a bad taste in my mouth. Another issue was that my MR
to the plugin repository triggered a broken CI job in my namespace,
causing me to exhaust the entire monthly quota of 300 minutes of CI.
I had no progress in the MR and no way to trigger CI for my own
projects. I don't understand how this can not be annoying for anyone!

Fast forward some time and various other frictions here and there, I
[discovered](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/merge_requests/31020#note_3140096907)
that through various changes to the plugin and bst, the `manifest.json`
inside Flatpak runtimes is now essentially inconsistent JSON. In
stable branches of Freedesktop SDK, ABI is kept stable in a
bidirectional manner, and while the `manifest.json` file is also
included in Flatpak runtimes, I doubt anyone thought it would ever
be "broken" like this.

The problem here is that the JSON needs to
stay consistent because people often parse it to know what software
is included inside a Flatpak runtime. I also had documented a `jq`
incantation to do that in Flatpak docs earlier. Any break in the JSON
structure inside a stable branch has the potential to break these
usecases.

I was naturally annoyed that this happened sort of behind our "backs".
The plugin maintainers who did the work were aware that this would happen
but I wasn't as it was not written in any changelog and we could
not pin the plugin to prevent these. The latter again being due to
`dulwich` or other dependency changes that would break things and would
need a plugin update to fix.

I think these two examples are enough to show that I was really
annoyed at how the plugins were being maintained and I think it was
justified. The last time this review delay happened, I asked to create
a fork of the plugin in the Freedesktop SDK namespace so that I can
push my fixes there and use them to progress on FDSDK work.

For whatever reason, not made clear to me or anywhere in the plublic,
one of the plugin maintainers, really did not want me switching the
plugin to a fork where I am pushing fixes, which was
somewhat annoying to me. I think the entire sequence is exactly how
FOSS is intended to work. Someone didn't have time or bandwidth to
review, I took "charge" to move the thing I am maintaining and used a
fork for the time being. Not letting me use a fork here was essentially
saying "I don't have time to review or merge and I won't let anyone
else move things either!". Nevertheless once my MR got eventually
merged, I agreed to switch to upstream plugins again and I think I
even approved the MRs making the switch from fork to upstream.

But again a few months ago, a bot-generated update MR broke CI due to
a plugin issue. So I again submitted an upstream MR and it again
got no review for months. Now this time the CI for the upstream plugin
repo was entirely broken and was failing all the time. To make things
worse, people would push to my MR branch to rebase it, the CI would
fail again and spam my mailbox - all the while the fix itself was not
going anywhere.

So I was really annoyed and given the previous misadventures I had a
short outburst and closed the MRs. I pushed the fix to the FDSDK plugin
fork, updated the bot MR to use that fork and fix the build at the
same time, and merged it. From my perspective, this was an entirely
trivial set of changes made solely to fix the build and get things
moving.

I had even announced on the bot MR that I was switching the plugin
and waited a day for someone else to post a review on an MR that was
already delayed over a month, but no one came.

So, from my perspective, the all-caps shouting linked in the other post
was directed specifically at me because they specifically did not want
me using a fork where I was pushing fixes. The other points posted there
had absolutely no relevance whatsover to the situation.

It can be seen in the comments that other people also "self-approve",
and it had mentions of "review bottlenecks" - the exact same problem
I described earlier. `bst-to-lorry` being broken is not "their"
problem. It runs in the [mirroring-config repo](https://gitlab.com/freedesktop-sdk/infrastructure/mirroring-config)
out-of-band and as a downstream script. I am the maintainer of
mirroring-config too! So if I break something in the main FDSDK
repository I would know in less than a week and I would have to clean
it up myself anyways! It has no relevance other than to highlight
something I "broke" and cause the blow up.

I am afraid the nuances and the entire general, historical and
technical context of this is not really expressible in comments and was
lost after people, who are unaware of the full picture probably took
the initial comments by their face value.

I wanted to explain my side of the situation a bit in these posts. My
next post is probably going to be about something I enjoy doing and
thinking about these or even writing this post was not it, at all.
