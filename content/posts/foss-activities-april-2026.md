---
title: "FOSS activities (April 2026)"
date: 2026-04-29T16:45:34+05:30
---

This most will summarise my FOSS activities over the last several
months.

<!--more-->

I would suggest reading these two posts for some major work that I did
in this time period.

- [Some updates to Flatpak Builder Lint (April 2026)](/posts/flatpak-builder-lint-updates-april-2026)
- [Platform wheel support in Flatpak PIP Generator](/posts/pip-generator-platform-wheels)

I will try to list below some more activities that do not belong in the
above detailed posts.

- Reported and fixed a reproducibility bug in fwupd. See [PR 10096](https://github.com/fwupd/fwupd/pull/10096).
- Reported and provided a patch for a reproducibility bug in NASM. See [issue 220](https://github.com/netwide-assembler/nasm/issues/220).
- Reported a reproducibility bug in OVMF. See [issue 12360](https://github.com/tianocore/edk2/issues/12360).
- Migrated flatpak-github-actions to Node 24, and released version 6.7. See the [commit log](https://github.com/flatpak/flatpak-github-actions/commits?author=bbhtt).
- Merged several fixes to the Steam Flatpak package. See the [commit log](https://github.com/flathub/com.valvesoftware.Steam/commits?author=bbhtt&since=2026-02-28&until=2026-03-31).
- Addded various Flatpak docs. See the [commit log](https://github.com/flatpak/flatpak-docs/commits?author=bbhtt).
- Fixed pygtk build in shared-modules repo. See [PR 460](https://github.com/flathub/shared-modules/pull/460).
- PR to add more context to reqwest errors to flat-manager. See [PR 165](https://github.com/flatpak/flat-manager/pull/165).
- PR to migrate flathub::manifest key from metadata to the custom tag in flat-manager-hooks. See [PR 134](https://github.com/flathub-infra/flat-manager-hooks/pull/134).
- Several test fixes to flathub-external-data-checker. See the [commit log](https://github.com/flathub-infra/flatpak-external-data-checker/commits?author=bbhtt).
  Added a feature to include branch name in PR title and merged an old PR fixing the checker for rotating load-balanced URLs.
  PR to add support for robots.txt, see [PR 512](https://github.com/flathub-infra/flatpak-external-data-checker/pull/512) which was a bunch of work.
  PR to support multiple `!=` constraints in the versions property, see [PR 510](https://github.com/flathub-infra/flatpak-external-data-checker/pull/510).
  PR to fix a freeze when async tasks do not get cancelled. See [PR 511](https://github.com/flathub-infra/flatpak-external-data-checker/pull/511).
- Reviewed 18 PRs to Flatpak itself and merged some
- 20 merged PRs to flatpak-builder, 4 PRs waiting to be merged, a security fix. I will write more about this later.
- PR to libglnx to silence warnings around g_clear_fd. See the [commit](https://gitlab.gnome.org/GNOME/libglnx/-/commit/dbc0d4f5531d512b83e3b8ba1905d5d91f1fe781). I want to extend
  this to other parts of libglnx too and tests for it. I have two draft PRs for that waiting for feedback.
- Several bug fixes and new features to Flathub backend, see the [commit log](https://github.com/flathub-infra/website/commits?author=bbhtt). Reviewed a couple of PRs here too.
- Several bug fixes and new features to vorarbeiter, see the [commit log](https://github.com/flathub-infra/vorarbeiter/commits?author=bbhtt). Reviewed a PR to add support for the merge command.
- Created a new action to close stale Flathub submission PR, see the [commit log](https://github.com/flathub/flathub/commits?author=bbhtt&since=2026-02-28&until=2026-03-31).
- Updated several Flatpak packages I maintain, and the usual exception PR reviews.
- Usual Freedesktop SDK maintainance, I crossed the 4000 commit mark recently in just 3 years:

```
git shortlog -sn --all --no-merges|head -4

 15884	freedesktop_sdk_updater
  4071	bbhtt
  2673	Seppo Yli-Olli
```

I also wrote several proposals to improve maintainance of the SDK, see the [open issues](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/work_items?sort=created_date&state=opened&author_username=bbhtt&first_page_size=100), submitted a fix to [buildstream-plugins-community](https://gitlab.com/BuildStream/buildstream-plugins-community/-/merge_requests/433)
and joined some arguments with some people!

That should be all.
