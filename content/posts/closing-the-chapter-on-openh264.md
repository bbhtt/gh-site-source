---
title: "Closing the chapter on OpenH264"
date: 2025-03-22T18:48:40+05:30
---

People might have noticed me talking about dropping OpenH264 from
[Freedesktop SDK](https://gitlab.com/freedesktop-sdk/freedesktop-sdk).
Here, I'll try to go a bit into the history, the timeline and what led
to the final decision.

<!--more-->

## A bit of an introduction

If you are unfamiliar with the Freedesktop SDK project: it was born out
of the initial [1.6 Flatpak runtime image](https://github.com/flatpak/freedesktop-sdk-images)
created by [Alexander Larsson](https://github.com/alexlarsson) to
provide a host independent "runtime" for Flatpaks. Since then, with the
help of [Codethink](https://www.codethink.co.uk/) and others, it grew
into an independent project maintained by the community that aims to
provide a "A minimal Linux runtime".

Currently that includes the `org.freedesktop.{Sdk, Platform}`
[Flatpak runtimes](https://docs.flatpak.org/en/latest/introduction.html#terminology),
a bunch of other [Flatpak extensions](https://docs.flatpak.org/en/latest/extension.html)
such as the GL (Mesa) extensions, a bootable image stack that includes
things like the Linux kernel, firmware and drivers and a collection
of docker images.

The runtimes (and more broadly the "Flatpak" stack) form the base for
the [GNOME](https://gitlab.gnome.org/GNOME/gnome-build-meta/) and
[KDE](https://invent.kde.org/packaging/flatpak-kde-runtime) Flatpak
runtimes which are collectively used by 2000+ Flatpaks, while the
bootable stack along with other parts forms the base for things like
[GNOME OS](https://os.gnome.org/).

## Some history

H.264 is one of the [most widely used](https://www.cloudflare.com/learning/video/what-is-h264-avc/)
codecs today but unfortunately it is patented with several patents
[still active](https://meta.wikimedia.org/wiki/Have_the_patents_for_H.264_MPEG-4_AVC_expired_yet%3F).
Patents like this are a blocker to shipping software dealing with the
codec in the base runtime (since we want it usable by a wide variety of
vendors and free of any legal grey areas) and unfortunately makes life
difficult for everyone involved.

To workaround this and ship working software to users, 6 years ago, in
2019 the precursor to the OpenH264 extension, called the
[html5-codecs extension](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/commit/1c1d87eb7f1e7952bf79825cfb65225f3a3c3e30)
was added to the Freedesktop runtime by Tom Coldrick. The idea was
simple - it would contain support for FFMPEG's internal H.264 decoder
and would be an optional Flatpak extension since we were unable to ship
it in the runtime itself.

Fast-forward a few months, in June 2019, [Robert McQueen](https://gitlab.com/ramcq)
opened an [issue](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/issues/815)
to include [Cisco's OpenH264](https://github.com/cisco/openh264)
(commonly referred as `libopenh264` too) as an extension to the runtime.

`libopenh264` code is open source but due to the H.264 patents, no
vendor is legally allowed to distribute their own binaries (some vendors
build it themselves and hand it to Cisco but no one made that arrangement
for us). The solution to this was to distribute Cisco's unmodified
binaries directly to the user which would effectively be free of any
royalties but the catch is, the binaries have
[some license restrictions](https://www.openh264.org/BINARY_LICENSE.txt)
on them.

So Endless around that time, added [extra-data](https://docs.flatpak.org/en/latest/module-sources.html#extra-data)
support to Flatpak. This meant that the Flatpak extension metadata
would only contain an URL to Cisco's binaries, checksums and a "recipe"
to make a working extension out of it. The binary would be downloaded
on the end user's machine, inside a sandbox and then the "recipe"
would be executed to create a working extension. This avoids various
"redistribution" restrictions entirely since we aren't shipping the
binary itself.

However, one last catch remained with this approach. Unless the end user
system has downloaded the extension, a part of the ABI will be missing
from the base runtime itself. Additionally, the extension would also
get mounted to a non-standard location inside the Flatpak sandbox. So to
build something against that we needed a "public" library that is
included in the runtime itself.

Endless came up with a stub OpenH264 library called [noopenhh264](https://github.com/endlessm/noopenh264/),
as a solution to this. This would be kept ABI/API compatible to the
actual Cisco's OpenH264 library and would live inside the runtime at
`$libdir` to allow software to link to libopenh264 and provide a
fallback. At runtime, if the user downloads the actual OpenH264
extension, the stub library would get overridden by the library in
extension through Flatpak's ld.so config and voilà, you have a working
OpenH264 setup!

## The start of the openh264 extension

Following the gruelling details above, in August 2019, Tom Coldrick
[again added](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/merge_requests/1638)
this "noopenh264" library to the runtime and set up the extension point
called `org.freedesktop.Platform.openh264`.

Around that time, the development of the "stub" noopenh264 library also
moved under the [Freedesktop SDK umbrella](https://gitlab.com/freedesktop-sdk/noopenh264)
while the development of this extra-data extension was going on in a
[dedicated repository](https://gitlab.com/freedesktop-sdk/openh264-extension).

I talked about a "recipe" of an extra-data Flatpak extension a while
back, which is nothing but a set of commands in a special file called
`apply_extra` to stage and install the source. As an example, if it
was a `.deb` it would be a sequence of `ar` or `bsdtar` commands to
extract it, then install it inside Flatpak's extra-data prefix.

The catch here is that using any utility like that makes the extension
directly dependent on a particular branch of the Flatpak runtime and thus
the API/ABI provided by it.

As every yearly major release of the Freedesktop runtime is a completely
new ABI, we would need to continuously spin off new branches of the
OpenH264 extension i.e. for version 2.1.0 of libopenh264 we would need
`org.freedesktop.Platform.openh264//2.1.0-{18.08, 19.08, 20.08}` branches
for the extension and so on. This would make it a bit of an chore to
update the extension in Freedesktop runtime itself and also for
our downstream runtimes.

As a solution to this, a custom `apply_extra` script was [written](https://gitlab.com/freedesktop-sdk/openh264-extension/-/commit/8a80eaaab8cd6b6aa7e241e681bdbaeed5632e56),
which utilised only a few standard headers and would be [built statically](https://gitlab.com/freedesktop-sdk/openh264-extension/-/commit/98601a6bcfd5a287106653bc69e54b58ce619c9e)
against a fixed toolchain.

The meant a bit of complexity, since we aren't allowed to use most
things that would make this process easier but it also meant the final
extension was independent of the runtime API/ABI and was built on top
of "NoRuntime".

## The flaws start to show up

This entire setup worked quite well for a long time despite some
maintenance issues from Cisco. However there were some flaws.

A while back I told that every vendor is forced to distribute binaries
hosted by Cisco at `https://ciscobinary.openh264.org`. Unfortunately
`ciscobinary.openh264.org` is missing a valid SSL certificate
[at least since 2014](https://github.com/cisco/openh264/issues/909#issue-34658985)
and Cisco neither supplies GPG signatures nor strong checksums like
SHA-256 for the binary releases.

This meant that we have no way to verify the authenticity of Cisco's
binaries, opening us to various classes of MiTM and supply-chain
vulnerabilities. We reached out to Mozilla and via Fedora/RedHat to get
Cisco to fix this, as one can see the comments on the thread but
nothing ever happened.

More recently (around 2024-2025), this SSL issue meant that the domain
started getting [DNS blocked](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/merge_requests/24521#note_2329956598)
and would make the extra-data download fail, botching an install.

Another critical issue was that if upstream did security fixes and then
released them with an ABI break, we couldn't fix it for the Flatpak
runtimes. Distributing binaries directly through extra-data meant there
is no scope for cherry-picking patches and the ABI break meant the new
version would only be able go to a new branch of the runtime.

We didn't really have a choice here. If we dropped this extension
considering the potential flaws, the entirety of the Flatpak user base
would loose H.264 decoding and encoding support, so we had to live with
the setup.

## The start of the codecs-extra extension

Around June 2024 `gdk-pixbuf` due to a series of security issues,
[dropped support](https://discourse.gnome.org/t/change-in-the-gdk-pixbuf-loaders-built-by-default-in-2-42-11/21845)
for a lot of "fringe" loaders. We immediately noticed the change as
we constantly analyse ABI in stable branches.

To compensate for the dropped loaders, we [decided](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/merge_requests/19916)
to add `webp, avif, jxl, heif` support (with pixbuf loaders) to the
runtime. Around the same time, [Will Thompson](https://gitlab.com/wjt)
opened an [issue](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/issues/1704)
to add `libheif` to the runtime with the problematic HEIC decoder as a
runtime extension.

Consequently, we decided that yet another extension just for two libheif
plugins is not worth the maintenance effort and these would rather be
in the `ffmpeg-full` extension (This was the successor to the
`html5-codecs` extension, created around 2019-2020). So I [again](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/merge_requests/20268)
[added](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/merge_requests/19942)
`libx265, libde265` and the corresponding libheif plugins to the
ffmpeg-full extension. After a few months I also [added](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/merge_requests/21970)
`libx264` to it too as we were already shipping `libx265` and Cisco's
OpenH264 had suboptimal encoding performance.

I was happy with the result and all of this shipped in Freedesktop SDK
24.08. But some people were unhappy for valid reasons.

When the `ffmpeg-full` extension was created, due to various concerns
it was never added to the runtime or SDK manifest; it lived as an
optional "app" extension - meaning app developers would need to add a
[short snippet](https://docs.flatpak.org/en/latest/extension.html#loading-existing-extensions)
in their Flatpak manifest to use it.

This was problematic in some ways - sometimes app developers were
unaware of the extension or that they needed it and the app manifest
was missing the snippet that allowed to use the extension. This made
user and developer experience a bit poor. Once we started
shipping `libheif` in `ffmpeg-full`, it became slightly more
problematic as the actual libheif library was in the runtime but without
the extension it cannot decode any HEIF/HEIC image. So the extension
was practically mandatory. [Albert Astals Cid](https://github.com/tsdgeos),
part of the team who maintains KDE Flatpaks on Flathub complained about
this and an [issue was opened by Erick555](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/issues/1825)
to discuss the possibility of making ffmpeg-full a runtime extension.

I was initially reluctant to do this for various reasons and the
situation was again a bit out of our hands (this time due to H.265
patents) but we asked various parties such as Endless and quite
surprisingly it turned out no one really had an objection to making it
a runtime extension or even make it auto-download along with the
runtime.

Along with this change, we decided to rename the `ffmpeg-full` extension
again, this time to `codecs-extra` to better reflect the fact that it
will not only contain FFMPEG with patented parts but other libraries
too that deals with patented codecs. This would sort of be an equivalent
to the various "meta-packages" that distros provide for patented or
restricted codecs.

Thus the seeds were planted and I switched `ffmpeg-full` to
`codecs-extra` a [month ago](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/merge_requests/24620).

This is supposed to ship in Freedesktop SDK 25.08. The major change for
app developers is that there is no longer any need to have something
like

```yaml
add-extensions:
  org.freedesktop.Platform.ffmpeg-full:
    version: '24.08'
    directory: lib/ffmpeg
    add-ld-path: .
```

in the manifest. `org.freedesktop.Platform.codecs-extra` will be
automatically installed by the runtime and will be available to users.

## Dropping the openh264 extension

The above `codecs-extra` change meant that we now didn't really have an
use case for `org.freedesktop.Platform.openh264` since `codecs-extra`
had FFMPEG's internal H.264 decoder and the `libx264` encoder.

So I initially disabled autodownload and considering the various "flaws"
with OpenH264 discussed above, I [opened an issue](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/issues/1834)
for planned retirement in 26.08.

This was completely overturned when a [high severity flaw](https://github.com/cisco/openh264/security/advisories/GHSA-m99q-5j7x-7m9x)
was discovered in libopenh264 affecting versions 2.5.0 and earlier.

The 23.08 branch of the Freedesktop runtime was locked to 2.2.0 and
upgrading to a fixed version was not possible due to multiple ABI breaks
and SONAME bumps upstream. Patching was also not an option as I said
earlier.

We scrambled to mitigate this. For 23.08, we decided to make an
"ABI break" and [drop the extension](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/issues/1846)
as updating was impossible. So I made a bit of an intimidating [announcement](https://discourse.flathub.org/t/upcoming-freedesktop-23-08-runtime-release-will-drop-openh264-extension/9022).

But we still had 24.08 left which was on 2.4.1 and updating to the fixed
version i.e. 2.6.0 was again blocked due to an ABI bump.

However, looking through the commits between 2.4.1 and 2.6.0 and
analysing the library through libabigail we couldn't find an actual
ABI break that was made and considering Cisco in the past made
unnecessary SONAME bumps, we tried patching the 2.6.0 release to
provide the older ABI.

I [messed](https://gitlab.com/freedesktop-sdk/openh264-extension/-/merge_requests/68)
with that venerable apply_extra and wrote a small utility to patch
SONAMEs. But upstream, in time, pointed us to the ABI break
and this idea had to be scrapped entirely. (I later found out why
liabigail was unable to show the ABI break after talking to Dodji
Seketeli, the libabigail maintainer.)

We then [asked upstream](https://github.com/cisco/openh264/issues/3863)
to provide a patch release 2.5.1 instead with the old ABI and to our
surprise they did within a few weeks! This shipped in 24.08.15 fixing
the 24.08 branch of the runtime.

After all this debacle and the extra "headache", none of us felt
comfortable shipping the openH264 extension anymore. Thus it was
[dropped](https://gitlab.com/freedesktop-sdk/freedesktop-sdk/-/merge_requests/24871)
for the master branch, which means there won't be any
`org.freedesktop.Platform.openh264` extension or `noopenh264` for 25.08+.

## Epilogue

Considering all things, I think and hope we made the correct decision
and hopefully the new `org.freedesktop.Platform.codecs-extra` works
out. `libx264, libx265` and others are built from source and there
are no binaries or extra-data involved. So we should theoretically be
able to patch and fix any issues that come up in the future.

Apart from all this, I'm slightly worried at the prospects of legal
issues cropping up with this setup and also that the new extension
contains "too much", but we will have to see where things flow.

As a fun tidbit, I wasn't aware that Fedora was also using our
`org.freedesktop.Platform.openh264` extension. I caused a bit of work
for them as noopenh264 [moved home](https://gitlab.com/freedesktop-sdk/noopenh264/-/commit/2177b9cb3542d0c9ccba4d20e56516118f8e747d)
once again, now to Fedora and they are looking for ways to make their
own OpenH264 extension similar to how we did.

I hope the experience here helps anyone in the future wanting to maintain
such an extension and this will also serve as a reminder to how much
extra work patents like these causes.

Lastly I'd like to thank Endless for giving us not only noopenh264 but
also extra-data support in Flatpak that made all this possible;
[Michael Catanzaro](https://gitlab.com/TheRealMichaelCatanzaro) and [Seppo Yli-Olli](https://gitlab.com/nanonyme)
for maintaining this setup for a long time; [Valentin David](https://gitlab.com/valentindavid)
for helping me in the last few days and everyone else who worked on
this ❤️ 
