---
title: "FOSS activities (July 2026)"
date: 2026-07-24T22:27:34+05:30
---

This post summarises my FOSS activities since the [last one](/posts/foss-activities-april-2026)
I made in April. Due to [various events](/posts/flathub-leave), this
is likely going to be the last one, at least for a long while.

<!--more-->

Unsorted:

- Fixed a bug in the cstate theme that was making dark mode a bit
  unusable for me. This is used for the Flathub status page, so I had
  to interact with it whenever the infra fell down, and it was a bit
  annoying. [Link](https://github.com/cstate/cstate/issues/353)
- Made a PR to OSTree with two trivial fixes I found while I was
  working on Flatpak builder. [Link](https://github.com/ostreedev/ostree/pull/3592)
- Fixed a bug in propcache that caused it to unintentionally bump
  the setuptools constraint through rogue automation. [Link](https://github.com/aio-libs/propcache/pull/225)
- Convinced the pypa/wheel maintainer to update the flit constraint and
  support flit v4. [Link](https://github.com/pypa/wheel/issues/687)
- Fixed two leaks in AppStream found by ASAN while working on
  flatpak-builder. [Link](https://github.com/ximion/appstream/pull/752)

Flathub documentation / policy:

- Added a formal policy on onboarding more team members. At this point
  a couple of new people were trying to join the review team and I felt
  that having a transparent and documented process would help, on
  both sides. [Link](https://github.com/flathub-infra/documentation/pull/606)
- Reviewed two new inclusion policies to curb the LLM slop fallout.
  [Link](https://github.com/flathub-infra/documentation/pull/603).
  I went into more detail about the second one in my other post
  [Some notes on Flathub's LLM policy](/posts/flathub-llm-policy).
- Migrated historical documents from the old GitHub wiki to the docs
  site so that I could properly decommission it.
  [Link](https://github.com/flathub-infra/documentation/pull/608)
- Migrated the YAML manifest style guide from the old wiki too and
  documented a JSON style guide
  based on what was being used and what was being suggested in submission
  reviews. [Link](https://github.com/flathub-infra/documentation/pull/609)
- Added contribution docs to Flathub in general. This felt like a major
  missing piece since people always came to me asking how they could
  help, and there was nothing to link them to for onboarding, etc. I
  also thought it would help bring more contributors in, who would be
  able to help with submission reviews or package maintenance.
  [Link](https://github.com/flathub-infra/documentation/pull/626). A
  link to this page lives in flathub.org footer as well.
- Created an app monetisation policy because some submissions were
  being sneaky / shady. This was a bit tricky since I didn't want to
  block legitimate monetisation. I think it turned out to be ok.
  [Link](https://github.com/flathub-infra/documentation/pull/629)
- Created an issue as a reminder for myself and a way for the team
  to discuss the AI policy and enforce only the reasonable parts. The
  slop fest was still at full force, so my wish was to do this exercise
  at a later point in time when things were calmer and the team had collected
  its thoughts about the effects of the current policy. My plan was also
  to ask the community; that was the reason I unlocked the issue
  (someone had posted a comment mentioning a prominent project was not
  able to submit due to the policy and later deleted it).
  Unfortunately, some people wanted a quicker result in the middle
  of the slop fest... [Link](https://github.com/flathub-infra/documentation/issues/620)
- Created a formal EOL policy. This came up during the Fedora-OBS Flatpak
  situation, and various people from the community wanted one too,
  especially after the licence debacle, as the project was growing a graveyard
  of unmaintained apps. My intention was to slowly nudge and make
  the app maintainers aware, and if they didn't have time, etc., the
  plan was to ask the community for help, arrange QAs, etc.
  [Link](https://github.com/flathub-infra/documentation/issues/621)

Flathub website (backend / frontend):

- Redid the Flathub website's about section. [Link](https://github.com/flathub-infra/website/pull/6660)
- Opened a couple of issues for some investigations that I planned on
  doing. [Link](https://github.com/flathub-infra/website/issues/6774)
- Added UI to show user ID in the settings page. [Link](https://github.com/flathub-infra/website/issues/6756)
- Made various bugfixes to the backend to stop Sentry spam, hid invalid
  entries from the statistics UI in the Flatpak version graphs, created
  a moderation-constants module, added an ignore list to moderation
  so that the moderators weren't being spammed by `x11` disappearing from
  the metadata after Flatpak 1.18.0 was deployed (fallback-x11
  previously used to generate an `x11` entry in metadata, which wasn't
  the case after 1.18.0), made various UI fixes to the admin-client UI,
  and added API and UI to mark apps as manually verified and logic + UI
  to mark them as unverified. [Link](https://github.com/flathub-infra/website/commits?author=bbhtt)

Flathub submissions:

Reviewed ~81 submission PRs during this time. This is according to
GitHub but I feel like it should be more. Maybe it's hiding some in
the contribution page or I missed some.

Vorarbeiter:

- Fixed logspam in vorarbeiter from linter logs. [Link](https://github.com/flathub-infra/vorarbeiter/pull/145)
- Opened an issue to vendor external JS bundles. [Link](https://github.com/flathub-infra/vorarbeiter/issues/165)
- Made various fixes, and added new functionality to detect PRs updating
  or changing runtimes and label them accordingly, automatically. [Link](https://github.com/flathub-infra/vorarbeiter/commits?author=bbhtt&since=2026-05-31&until=2026-06-30)

Flathub packages:

- Made various updates to `org.flatpak.Builder`, `net.sourceforge.liferea`, `com.valvesoftware.Steam`.
- Migrated Flathub infra to Flatpak 1.18.0. [Link](https://github.com/flathub/org.flatpak.Builder/pull/407)
- Merged the 6.11 branch of the QtWebEngine baseapp. [Link](https://github.com/flathub/io.qt.qtwebengine.BaseApp/pull/406)

Flatpak builder lint:

- Dropped the lints that blocked the USB and input device permissions
  since they had no backwards compatibility. This was in preparation for
  Flatpak 1.18.0, which solved the backwards compatibility issue by
  introducing conditional permissions. [Link](https://github.com/flathub-infra/flatpak-builder-lint/pull/1015)
- Added support for linting conditional permissions in preparation
  for Flatpak 1.18.0. [Link](https://github.com/flathub-infra/flatpak-builder-lint/pull/1022)
- Fixed a bug while scanning for submanifests. It was incorrectly
  falling back to glob too early and picking up everything instead
  of the actual submanifests. [Link](https://github.com/flathub-infra/flatpak-builder-lint/pull/1021)
- Made a couple of lints more specific to the corresponding
  permission so that exceptions were more fine-grained. [Link](https://github.com/flathub-infra/flatpak-builder-lint/pull/1025)
- Reviewed and/or merged ~25 exception PRs.

Flathub repro checker:

Added a workaround so that it could work with VLC built on their
own infra. They were planning on making it a direct upload, and I
was [reviewing](https://github.com/flathub/flathub/issues/6387) that
whole thing. [Link](https://github.com/flathub-infra/flathub-repro-checker/pull/7)

Flat manager hooks:

Made an issue to discuss creation of a strictly sandboxed
subset. A lot of subset options were being cargo-culted during
the Fedora-OBS drama, and I felt that most of them were arbitrary and
unreasonable. I felt like making a sandboxed subset would solve some
problems. The explanations are in the issue. [Link](https://github.com/flathub-infra/flat-manager-hooks/issues/141)

Flathub submission checker:

In one of my earlier blog posts I said:

> I've developed and maintain a number of GitHub actions in that
> repository (and in the flathub-infra org) to automate parts of the
> review process. The automation helps, but a large portion of the
> submissions is still difficult to filter automatically and needs to
> be reviewed manually. One of these actions is currently a fairly
> large shell script. At one point, I started converting it to
> Python, but I ended up dropping the ball on it. I intend to pick it
> up again at some point.

and the submission checker action was the culmination of my effort
to port that 300-400 line shell-based action into something
maintainable, written in Python, with proper CI and tests.

I also made two functional changes. The first was that the action
would run instantly once a PR was opened (and could also be triggered
manually). This was based on some feedback I saw that the delay
was not very helpful. The delay in the old action was not a design
choice but a GitHub limitation, as there was no way to safely run an
action on a non-default branch other than by using a scheduled workflow.

The next one was that it would instantly close PRs that followed
extremely AI-generated patterns with a default neutral response. At this
point it was really tedious to post a template response and close 5-10
PRs per day manually; it was making the experience sour for me, and I
saw (or felt) that others weren't liking it either. A neutral automated
response felt like a solution to both.

I thought the detection rate of it was pretty good. There were
a couple of false positives where it would close the PR, and I was
fixing them one by one.

Anyway, I later learnt that this was an unwelcome change, but nobody
contacted me with the feedback beforehand. I also failed to properly
communicate the changes, but if I had received feedback I would have
tried to fix them.

[Link](https://github.com/flathub-infra/flathub-submission-checker)

Flatpak External Data Checker:

- Added a feature to try harder to preserve JSON formatting in
  manifests. Turns out it's hard. [Link](https://github.com/flathub-infra/flatpak-external-data-checker/pull/517)
- Added a fallback for chromiumchecker. [Link](https://github.com/flathub-infra/flatpak-external-data-checker/pull/522/changes/b2283eff0b57d12a779fe70801b053bc6e515db5)
- Added timestamp support to anityachecker. [Link](https://github.com/flathub-infra/flatpak-external-data-checker/pull/528)
- Added support for respecting robots.txt. This was done in response
  to a feature request made by Will, and I thought it would be a nice
  addition. It later became clear that this could never be turned on by
  default, because most of the API URLs x-checker parses have
  unreasonable things in their robots.txt, and that would break the
  checker. So it's opt-in. The details are in the PR. [Link](https://github.com/flathub-infra/flatpak-external-data-checker/pull/512)
- Added support for multiple exclude version constraints in response
  to a very old issue. [Link](https://github.com/flathub-infra/flatpak-external-data-checker/pull/510)
- Merged my potential theoretical fix to a hang condition. [Link](https://github.com/flathub-infra/flatpak-external-data-checker/pull/511)
- Migrated the project to using modern linters / formatters like
  ruff, mypy, and uv. It had been using flake8 and black inside a venv
  in a rather primitive way since dawn. That made the developer
  workflow unreproducible in various cases, as my local black version
  wouldn't match the one in CI, producing different formatting and so
  on. Using uv clearly felt like the better solution. I also took the
  chance to migrate it to ruff, fixed various lints, widened the mypy
  rules, and fixed various type issues. The PR was rather large in the
  end. At this point I was unable to find anyone to review it, and
  merged it on my own.
  [Link](https://github.com/flathub-infra/flatpak-external-data-checker/pull/524)
- The other two large sets of work were adding a huge number of unit
  tests so that I could bump up the coverage to 100% and get a green
  badge. It felt like something to chase at the moment, not sure why.
  The test suite of x-checker was entirely integration tests before
  I took over. They were heavily network-dependent, failed randomly,
  were slow, hard to write or test, and had a lot of other problems.
  There was an [old issue](https://github.com/flathub-infra/flatpak-external-data-checker/issues/325)
  suggesting improvements. The signs were clear that the project needed
  to mock things and write unit tests (which I was already doing). So all
  the tests written to bump the coverage were unit tests and worked
  without network. Some of them were extremely tedious to write
  to get 0.01% coverage up, and I took some [shortcuts](https://github.com/flathub-infra/flatpak-external-data-checker/issues/532)
  along the way. That should be fixed by someone.
  Links: [1](https://github.com/flathub-infra/flatpak-external-data-checker/pull/530),
  [2](https://github.com/flathub-infra/flatpak-external-data-checker/pull/531).

[Link](https://github.com/flathub-infra/flatpak-external-data-checker/commits?author=bbhtt)

Flatpak:

- Fixed a build warning in a test. [Link](https://github.com/flatpak/flatpak/pull/6655)
- Made assorted fixes while trying to prepare to migrate to Flatpak 1.18.0
  in Flathub infra. The project was using a fork with some patches, and I had
  tests disabled. I turned them on and used the exact same build args
  the project used, and found some random test bugs.
  [1](https://github.com/flatpak/flatpak/pull/6713) and [2](https://github.com/flatpak/flatpak/pull/6681)
- Fixed various bugs found from issue reports. [1](https://github.com/flatpak/flatpak/pull/6696), [2](https://github.com/flatpak/flatpak/pull/6687)
- Reviewed ~16 PRs.

Flatpak builder:

- Enabled ASAN and LSAN in CI, and made an assortment of leak fixes, bug
  fixes, security fixes, and a couple of new features. [Link](https://github.com/flatpak/flatpak-builder/commits?author=bbhtt)
- Released 1.4.9, and 1.4.10 shortly after.
- Reviewed ~25 PRs.

I may make a detailed post about this later on.

Flatpak GitHub Actions:

Reviewed and merged 3 PRs. [Link](https://github.com/flatpak/flatpak-github-actions/issues?q=is%3Apr+author%3Aelectricbrass)

Libportal:

The last release of this was over a year ago, and the maintainers
didn't have time to review PRs or do a new release. There was
essentially a pile of random fixes and new features in the PR
queue. Some had already been reviewed and approved by various people,
but they didn't have merge rights. I tried contacting them and
the current maintainer on GitHub but didn't get a reply. At the
same time, people had been asking for a new release for 2 months or
so. I also needed a new release with some Qt 6.9+ fixes made almost
a year ago. So I stepped in, merged whatever I could, and
performed the 0.10.0 release. [Link](https://github.com/flatpak/libportal/commits?author=bbhtt)

Flatpak docs:

Documented the new conditional permissions and made various fixes
to make Sphinx linkcheck work properly. [Link](https://github.com/flatpak/flatpak-docs/commits?author=bbhtt)

Flatpak builder tools:

- Reviewed and merged a rather large PR adding PNPM v11 store support
  to Flatpak Node Generator. PNPM made it especially annoying,
  since the store needed to be an SQLite database in this version, and
  a whole custom msgpack packer had to be used due to various fixed
  caps. Thereafter, merged several follow-up fixes to that. [Link](https://github.com/flatpak/flatpak-builder-tools/pull/537)
  This was a great and very desired contribution.
- Fixed a very old bug reported in the Flatpak CPAN generator.
  [Link](https://github.com/flatpak/flatpak-builder-tools/pull/533)
- Made various test suite fixes to Flatpak Node Generator. This was due
  to a regression in newer Electron, but I snuck in some more
  fixes that made the integration tests easier to run locally.
  [Link](https://github.com/flatpak/flatpak-builder-tools/pull/541)
- Replaced the old README with more information and documented
  de facto contribution guidelines and process. [Link](https://github.com/flatpak/flatpak-builder-tools/commit/d9a48e3fc08576962307688de9aa64bd5572705a)

That should be all.
