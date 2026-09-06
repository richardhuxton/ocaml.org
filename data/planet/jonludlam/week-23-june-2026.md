---
title: Week 23, June 2026
description:
url: https://jon.recoil.org/blog/2026/06/weeknotes-23.html
date: 2026-06-08T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---

<h1><a href="https://jon.recoil.org/atom.xml#week-23,-june-2026" class="anchor"></a>Week 23, June 2026</h1>
<ul class="at-tags"><li class="published"><span class="at-tag">published</span> <p>2026-06-08</p></li></ul>
<p>...and previous weeks.</p>
<h2><a href="https://jon.recoil.org/atom.xml#current-topics" class="anchor"></a>Current topics</h2>
<ul><li><a href="https://jon.recoil.org/atom.xml#general" title="general">General odoc directions</a></li><li><a href="https://jon.recoil.org/atom.xml#ocaml_docs_ci" title="ocaml_docs_ci">OCaml Docs CI</a></li><li><a href="https://jon.recoil.org/atom.xml#dune_odoc_rules" title="dune_odoc_rules">Dune Odoc rules</a></li><li><a href="https://jon.recoil.org/atom.xml#compsci" title="compsci">Exams!</a></li></ul>
<h2><a href="https://jon.recoil.org/atom.xml#general" class="anchor"></a>General odoc directions</h2>
<p>While docs are, in general, better than they were a few years back, it's definitely not "done". I've been considering what life would look like if docs were "first class". I'm writing this up, but for now I'll at least point at the first few bullet points.</p>
<ul><li>High quality API and package documentation should be installed with every opam package. This implies that every build system is capable of producing them, and CI should check them.</li><li>Editors should offer completion in docs, highlight errors when you're editing, and offer a way to display docs natively.</li><li>Searching for docs, in editor or via the CLI, should be straightforward and useful.</li></ul>
<p>The last point is particularly useful for LLMs.</p>
<p>Obviously there's a lot of work required here! I'm currently thinking about a pathway that gets us most of the benefits ASAP, with a longer term outlook of doing it without "hacks". I'd like to get the 'ASAP' tooling published so that other people can have a go and get some of these benefits.</p>
<h3><a href="https://jon.recoil.org/atom.xml#cmt_vs_odoc" class="anchor"></a>.cmt vs .odoc</h3>
<p>As part of this, I wanted to take a good look at the cmt/cmti format to see what's in there and why, and where the odoc file format differs. They both are based on the underlying source tree, so it's quite reasonable to ask where the similarities and differences are, especially if I'm suggesting installing odoc files as well as cmti!</p>
<p>I realised in doing this that while we've got docs for <em>users</em> of odoc, we've not got great coverage for <em>developers</em>. So I've spent some time this week working on this. I initially started writing this for this blog, but quickly realised that it really should be in the odoc repo, so that'll be coming soon. I'll try to get this done without spending too much time polishing it.</p>
<h3><a href="https://jon.recoil.org/atom.xml#ident_thoughts" class="anchor"></a>Idents vs Identifiers</h3>
<p>In the interests of seeing closer parallels between odoc files and cmt/i files, I got Claude to spend some time investigating using local idents instead of identifiers in the odoc files, and switching to identifiers when writing out the odocl files. This would remove one big source of difference between the formats, and potentially make the code more easier to read for people used to OCaml's module implementation. A side benefit could also be an improvement in performance, as we spend quite a lot of time converting between these two types as odoc runs. Unfortunately this proved a little too tricky for Claude and while it made some progress, it seems I'll need to spend a lot more time describing the change, and/or working through it with more supervision.</p>
<h2><a href="https://jon.recoil.org/atom.xml#ocaml_docs_ci" class="anchor"></a>OCaml Docs CI</h2>
<p>This has been dragging. So I'm currently on a sprint to get this sorted by cutting out any nice-to-haves. It's already in a state where it's better than current ocaml-docs-ci, so my focus now is on doing the absolutely necessary bits to have it in a sensible state for ocaml.org. These are:</p>
<ol><li>Deployability - fix up the docker files / ansible / ocurrent-deployer such that getting a new version running is as simple as pushing to github.</li><li>Wire in the epoch management mechanism. This has proven invaluable with the current ocaml-docs-ci, and was relatively simple to put in place.</li><li>Ensure docs are good enough for others to debug/maintain/fix it.</li></ol>
<p>So the other neat bits of it are deprioritised - building oxcaml packages, building latest odoc master branch, local override repositories, etc. Let's get it out and being used so it's no longer dragging me down!</p>
<h2><a href="https://jon.recoil.org/atom.xml#dune_odoc_rules" class="anchor"></a>Dune Odoc rules</h2>
<p>Paul-Elliot and Arthur have been doing a good job cutting up my branch into smaller chunks, but it seems to have hit a got a bit mired in the mud. There are a few good small commits, and a few have been upstreamed, but there's going to be a big chunk that's still quite hard to put into manageable commits. I'll be looking over what's been done so far to see if I can figure out a pathway forward.</p>
<h2><a href="https://jon.recoil.org/atom.xml#compsci" class="anchor"></a>Exams!</h2>
<p>The exam for the Foundations of Computer Science course I lectured this year is coming up this week, so I'll be spending a lot of time marking for a while!</p>
