---
title: "Some notes on Flathub's LLM policy"
date: 2026-05-31T14:03:42+05:30
---

I reviewed and acked a pretty extreme amendment to Flathub's LLM policy
yesterday and I want to talk about how we got here.

<!--more-->

I started reviewing Flathub submissions around 2022. Back then it was
Bart, Hub, and me. The queue was manageable and if the [PR template](https://github.com/flathub/flathub/blob/master/.github/pull_request_template.md?plain=1)
was filled out, the manifest looked good, and the submission was
otherwise fine and worked, we'd merge it within a few days. There were
no formal review guidelines and we never thought we needed something.

After a while, some spam PRs started appearing. So we [tightened](https://github.com/flathub/flathub/commit/7c17cdc50cd4fc5e773aeb95b02d11be2614c589)
the PR template and added a checkbox requiring a video attachment. The
intent being simple, that spammers wouldn't be able to complete it, and
the video would help with reviewing legitimate submissions. We also
required PRs to target the non-default `new-pr` branch which was
another hurdle spammers consistently failed. I also added a workflow to
automatically close any PR opened against `master` as most of them were
spam.

Then the "Fedora-Flathub" drama happened and we were effectively forced
to ban some entire classes of submissions. That led me to write what
now sits as the [inclusion policy](https://docs.flathub.org/docs/for-app-authors/requirements#inclusion-policy). This was the first time in 6 years Flathub had a concrete review policy and we were somewhat forced to create it.

I don't exactly remember when but all of this was likely before the real wave which was the surge in LLM-generated submissions and
manifests.

It started predictably. PRs against the wrong branch were auto-closed.
Then there were PRs against the correct branch but with the template
removed, ignored, or replaced with some invented template, or manifests stuffed
with entirely fabricated Flatpak Builder properties which we closed
manually.

Around this time or some time before this, Bart also had stepped back
from reviewing after five years, mainly from burnout, understandably.
So it was just me and Hub.

The LLM situation became absolutely atrocious.

I responded by adding new checks in `flatpak-builder-lint` to catch
LLM-generated manifests, and a workflow in the review repository to
catch PRs that skipped the template or were missing required files.

These helped, but the LLMs adapted quickly and automation can only catch so
much and without being annoying to the innocent newcomer.

On some days we were seeing close to 10-15 slop PRs opened. Some of
them were auto-closed, or caught by automation and we had to close
or reject the rest manaully. We got notifications for every one of these. This was truly unbearable.

This led Hub to add the
[first version](https://github.com/flathub-infra/documentation/pull/537)
of Flathub's LLM policy:

> AI-generated submissions are not allowed as these are typically
> low-quality, often violate the submission requirements, and create
> unnecessary work for the reviewers. They also raise ethical and legal
> concerns such as potential licence infringement and resource wastage.
> We reserve the right to reject such submissions, without any review.

It wasn't entirely clear whether this covered the submission itself, the
application being submitted, or both. My reading was that Hub wanted to
ban both but having never used these myself, I had no strong opinion 
either way, at that time and I merged it so we'd have some grounds to close the "slop" PRs.

Shortly after, LLM usage spread enough that some core ecosystem projects
started adopting it, including some people within our own circle. Bart, also wanted a more moderate stance
and I wanted to avoid disagreements amongst ourselves.

So, instead of going with a hardline stance I [amended](https://github.com/flathub-infra/documentation/commit/2f1a16e9d8dcb4550864669b84e8ab9a41037e85)
the policy to:

> Submission pull requests must not be generated, opened, or automated
> using AI tools or agents. Please also do not request review from any AI
> tools in the submission PR. Automated Copilot reviews on GitHub can be
> disabled by the submitter by going [here](https://github.com/settings/copilot/coding_agent)
> and changing `Repository access` to exclude the repo.
>
> Submissions or changes where most of the code is written by or using AI
> without any meaningful human input, review, justification or moderation
> of the code are not allowed.
>
> Submissions or changes having low-quality AI-generated or AI-assisted
> code are not allowed.
>
> Submissions or changes containing copyrighted, license-incompatible, or
> ethically questionable code are not allowed.
>
> These submissions can be rejected without any further review.
>
> Repeatedly violating these policies may result in a permanent ban from
> future submissions and activities.

I made it explicitly cover both the PR and the app being
submitted. I tried to keep it moderate by placing a "burden of proof" on
the LLM users by requiring "meaningful human input, review,
justification or moderation, while still allowing us to reject the
worst offenders. This policy stayed in place for a while but the 'slop' never stopped.

Worse, those specific words became a trap as every rejection
turned into a prolonged argument about whether the submitter's work
qualified as sufficiently "human-reviewed." I thought the language
would deter 'slop' but instead it gave them ammunition to argue and
dispute the rules.

Some of these submitters had "apps" that were a single 8,000-line fully
LLM generated Python script and other times the "app" was a similar
shell script. When we rejected them, they were often adamant and
confident their work was clean, well-reviewed, free of bugs etc. A few
of them became aggressive and started Matrix pings, Discourse threads,
emails to the admin address, GitHub mentions etc. When we closed the PR, they'd open a new one and the same thing would repeat.

This led to me and Hub handing out GitHub bans like candy, probably four or five a day each, just to live and review in peace.

After getting banned on GitHub, they'd continue on Discourse, then on
Matrix and so on. We had to repeat the whole exhausting process on each
platform. Nobody likes handing out bans.

Then when they were out of options they started posting hitpieces on
socials about us and I hope people know what that leads to on the
internet.

Once they'd been removed from all the public channels, some would
start emailing the admin address, which Bart monitored for essential
things. Without full context, he occasionally unblocked people thinking we were too strict which led to more "slop" submissions, some of which fell through the cracks and got accepted, which lead to more
frustration and so on. This eventually led to friction
[between us](https://github.com/flathub/flathub/pull/8472#issuecomment-4318476717)
over who should be banned or unblocked and which submissions to accept.

We kept adding new inclusion policies like the "Software design and
distribution" policy, "Insufficient development history" policy, new categories of impermissible submissions, hoping they would let us reject things easier and faster but it didn't. People found loopholes or disputed every clause
until we had no energy left, and if we ended the conversation, they'd
escalate.

The entire thing was horrible and so so draining. I wrote about
this in my past blogposts multiple times too.

So last week this I think naturally led to [blocking all LLM-assisted and LLM-generated submissions](https://github.com/flathub-infra/documentation/commit/992f57b30de98ddbd5e80959e9672998c83c8c97).

The nuclear response comes from a place of desperation and exhaustion.

The policy now reads:

> This policy applies to both the application being submitted to Flathub
> and the Flathub submission itself, including the manifest, metadata,
> patches, build scripts, and pull request. For the purpose of this
> policy, applications include BaseApps, extensions, and any other
> artifacts that can be produced by flatpak-builder.
>
> Submission pull requests must not be generated, opened, or automated
> using AI tools or agents. Please also do not request review from any
> AI tools in the submission PR. Automated Copilot reviews on GitHub
> can be disabled by the submitter by going here and changing Repository
> access to exclude the repo or disabling the global
> "Automatic Copilot code review" found here.
>
> Applications containing AI-generated or AI-assisted code,
> documentation, or other content are not allowed.
>
> Applications or changes containing copyrighted, license-incompatible,
> or ethically questionable code are not allowed.
>
> These submissions can be rejected without any further review.
>
> Repeatedly violating these policies may result in a permanent ban from
> future submissions and activities.
>
> Exceptions may be granted for mature, well-maintained projects.

I am aware that it's extreme, that it's vague and probably
unenforceable in some cases. But some of the vagueness is somewhat by
design because as soon as we write something specific, people start
looking for the loopholes in it, and we don't have the bandwidth to
catch up or act like lawyers at this point.

The vagueness is also deliberate to leave some of the final judgment with the reviewers as always and the "extreme"-ness is there to empower the reviewers to reject the worst offenders in one shot. I know that sounds 'cruel' but there is no option left for us.

At least this way we won't have to spend all our energy and argue for
days on end to determine whether a 5000 LoC single file LLM generated
Python script constiutes as a valid non-'slop' app submission or not.

I think we tried hard enough to maintain a quality bar and allow
decent apps where authors had used
LLM assistance. However, there seems to
be little decency and charity left if most of the world is indecent.

So here we are.

I certainly neither plan on going on witch-hunts against people with the
new policy and nor would it be applicable to dependencies or transitive
dependencies of an app being submitted. It's also highly unlikely to
be applied if someone merged a couple of lines of AI assisted PR
or used it for icons or a security fix, or used it to write some man
pages.

I'm not an extremist and I don't plan on applying it on anything other
than the source code unless _well_ the 'other things' are looking
absolutely horrendous to the eye.

None of this is directly written anywhere to precisely preserve some of
that "necessary vagueness".

We are also not giving any free passes to proprietary apps, nor to our
"friends", nor to corporate apps, I don't think we never have and at
least, I don't have any "friends" in this space. Also after working
together for more than four years, even if it's online and I never saw any others' faces, I believe none of us are corporate shills. I'm certainly not, while on the contrary, I have the feeling that some of us are quite anti-corporate. I am as well, to some extent. 

Given recent events, I think we will only need to tighten the rules for proprietary apps even further. The exception clause exists purely to give us case-by-case leeway.

People have trusted us for this long, and I think they can trust us a
little more not to misuse it. At least I will not stay quiet if it
gets misused.

Submissions from actual human beings who genuinely tried to follow the
docs and the requirements are as always welcomed, even if they make
mistakes in the process. I absolutely want to merge more submissions
if they look good and I don't like denying people.

But what I do want to prevent is the slopocalypse and every reviewer
or contributor getting drained by LLM users to the point that no one
wants to review anything or contribute to Flathub as a project. That
will be an absolutely horrible future for everyone and the project
itself.

So let's see how all of this goes!
