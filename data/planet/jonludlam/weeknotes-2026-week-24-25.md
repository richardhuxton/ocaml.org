---
title: Weeknotes 2026 week 24-25
description:
url: https://jon.recoil.org/blog/2026/06/weeknotes-25.html
date: 2026-06-22T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---


      <p>I have now finally finished almost all of the end-of-term duties, incuding marking of 128 Foundations of Computer Science exam questions. Phew! It ended up being quite a week, with the unwelcome additional burden of having my car written off due to a minor prang.</p>
<p>The main things I've been working on have been finalising the new ocaml-docs-ci implementation, and working on a plan for "first class docs" in OCaml. The latter I've written up <a href="https://jon.recoil.org/blog/2026/06/first-class-docs.html">here</a>. The former is currently running on a rebuilt <a href="https://dill.caelum.ci.dev">https://dill.caelum.ci.dev</a>, building docs for all versions of all packages.</p>
<h3>First-class docs</h3>
<p>I've written up a post on what 'First Class Docs' in OCaml might mean, and in parallel I've made a little tool to see how it feels. I've not put it on the main post, but I'll share a little video of it here:</p>
<p><video controls="" src="./odd_demo.mp4"></video></p>
<p>It's a tiny tool that auto-builds docs for your switch, keeping them up to date automatically as packages are added and removed. It's got sherlodoc-based search, it can show docs in markdown in the terminal for any item in any package, completion of odoc-style references, and the completion is integrated with zsh so you can do `odd doc Odoc_&lt;tab&gt;` and it will show you sensible completions.</p>
<p>Next step is to create a skill for LLMs to use this tool. I'm pretty sure they'll find it very helpful!</p>
<p>Source is here: <a href="https://tangled.org/jon.recoil.org/odd">https://tangled.org/jon.recoil.org/odd</a></p>
<h3>Docs CI</h3>
<p>The work on docs-ci was not terribly exciting, but should be leading to a switch over in the next few weeks, once I'm happy that the service is more stable than the current one (which isn't a particularly high bar).</p>
<ul>
<li>The new way of running meant fewer docker containers, so the deployment had to be changed</li>
<li>I added a Caddy webserver rather than nginx, and updated the configuration so that it can serve multiple profiles</li>
<li>I've only enabled the 'quick' profile and the 'full' profile for now - the other two I've been using are 'oxcaml' and 'odoc-master' that do the obvious things.</li>
<li>I've added a few more bits of info when you're browsing the state - things like the <a href="https://dill.caelum.ci.dev/profiles/full/snapshots/2ff3cda8c7e6">latest commit</a> on the opam repositories being used, a <a href="https://dill.caelum.ci.dev/profiles/full/snapshots/a315b93bb320/diff/eabbc6b9a9da">diff view</a> between snapshots, some instructions on <a href="https://dill.caelum.ci.dev/profiles/full/p/raylib/2.1.0">what to do when your package fails</a></li>
<li>Some more minor tidying for "release"
I've now left it running for a while just to see how it behaves. It's looking pretty good, though there were a few issues to iron out. The first was to do with epochs, which is a mechanism to allow deployment of new versions of odoc, sherlodoc, odoc_driver and so on whilst keeping the older version "live" until you want to switch over to the new version - the new epoch. The problem was that it was keying the epoch off the full solves of the tools rather than just the versions of the tools themselves. The consequence was far too rapid cycling of epochs, so I had to fix that. The second was that the switch to the new epoch did a synchronous GC of the older epochs, which led to a very long pause. Then the release of OCaml 5.5 turned out to be very useful, as while all the packages built as expected, none of the docs appeared. We have an override in the profile to be able to select the compiler version used to build odoc_driver and related tools, which was set to 'None' to mean 'latest version'. However, in this case, 'latest version' meant 'pin to the latest version of the compiler' as opposed to 'best version that has a solution'. This constraint, along with js_of_ocaml not yet working with OCaml 5.5, meant that there was no solution for the tools, hence no docs! The quick fix for this was to put a pin in place to OCaml 5.4.1 and the docs popped right out.</li>
</ul>
<p>Some info: A full build took about 12 hours. This is on roughly the same hardware as the current docs ci, where it takes the best part of a week to do the same, so the day10/day11 architecture has made it much faster. Each snapshot builds about 17600 packages, in about 31,000 build layers, and 31,000 doc layers. The total space for one complete snapshot is on the order of 1.1Tb. The OCaml 5.5 release rebuilt most of the packages, so we're currenly up to about 2.3Tb of storage used.</p>

    
