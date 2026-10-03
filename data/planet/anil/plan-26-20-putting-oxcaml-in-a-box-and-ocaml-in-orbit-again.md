---
title: '.plan-26-20: Putting OxCaml in a box and OCaml in orbit (again)'
description: Consolidating my OCaml trees for easier OxCaml deployment, shipping native
  system packages for OxCaml which then got into space, and remembering Peter Neumann
url: https://anil.recoil.org/notes/2026w20
date: 2026-05-17T00:00:00-00:00
preview_image: https://anil.recoil.org/images/borealis-clustergate2-boot.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>I've had a heads-down week spent consolidating my OCaml trees (pun intended) so
that deploying <a href="https://anil.recoil.org/projects/oxcaml">OxCaml</a> is easier. We've started generating test
<a href="https://anil.recoil.org/projects/tessera">TESSERA</a> embeddings for a forthcoming v1.1 model <a href="https://www.tunbury.org/2026/03/13/oxcaml-inference/">using OxCaml</a>
and so it's quite important to get my <a href="https://anil.recoil.org/notes/aoah-2025">monorepo</a> under control.</p>
<h2><a href="https://anil.recoil.org/news.xml#oxcaml-in-a-box" class="anchor" aria-hidden="true"></a>OxCaml in a box</h2>
<p>One concrete output comes from my investigations about how <a href="https://anil.recoil.org/notes/oxcaml-packages">native system packages for OxCaml</a>
could work.  I had to root around a bunch of instructions on various distro wikis to figure out how system
packagers work these days, but now <code>apt install oxcaml-compiler</code> on Debian/Ubuntu, as well as the
other commands for Fedora, Arch and Homebrew, all work from one <a href="https://tangled.org/anil.recoil.org/oxcaml-pkgs">installer script</a>.</p>
<p>All this trouble is necessary because of how staggeringly complex adding
relocatability to a build system is. The master of builds, <a href="https://www.dra27.uk">David Allsopp</a>, has been
kindly assisting me as I explore this integration in
<a href="https://github.com/avsm/oi">oi</a>. I'm almost ready to start talking about oi as this relocatability for oxcaml is the last thing on my list before a preview release!</p>
<h2><a href="https://anil.recoil.org/news.xml#ocaml-in-orbit-again" class="anchor" aria-hidden="true"></a>OCaml in orbit, again</h2>
<p>I've also been helping <a href="https://github.com/samoht">Thomas Gazagnaire</a> with <a href="https://gazagnaire.org/blog/2026-05-14-borealis.html">Borealis</a>, the
pure-OCaml CCSDS stack now <a href="https://anil.recoil.org/notes/2026w18">flying in low earth orbit</a>. It
hit the <a href="https://news.ycombinator.com/item?id=48147058">front page of Hacker News</a> mid-week, which
is always fun.</p>
<p><a href="https://gazagnaire.org/drafts/46f8e325a7fc.html"> <img src="https://anil.recoil.org/images/borealis-clustergate2-boot.webp" alt="%c" title="OCaml running in space: the proof's in the VT220 console"> </a></p>
<p>I was quite surprised to see in the <a href="https://news.ycombinator.com/item?id=48148343">discussion</a> that
another group ran <a href="https://lambda-diode.com/static/data/GHGSat_OCaml.pdf">OCaml on the GHGSat satellites back in 2016</a>:</p>
<blockquote>
<p>OCaml was very much part of the GHG measurements. On the satellite it was
controlling the cameras, acquiring the images, losslessly compressing them,
encrypting them and transferring them to the platform controller using a
clunky but mandated CSP-based file tranfer protocol. On the ground, OCaml was
running almost the entire data processing chain, including spectroscopy,
image corrections, retrievals and post-retrieval ad hoc bias corrections, as
well as simulations.</p>
<p>I simply used an <code>mmap()</code>'d Bigarrays to do parallel processing (back then
OCaml wasn't multi-core.) At a later stage I replaced a few bits of code
(e.g. some sparse matrix routines) with Fortran. The only processing-related
part that wasn't OCaml (besides the shells scripts to glue the things
together) was the image alignment algorithm which was written by someone else
in C++. I even had a job scheduling system written in OCaml.</p>
<p><cite>-- <a href="https://news.ycombinator.com/item?id=48148560">rho-soul-kg-m3, HN, May 2026</a></cite></p>
</blockquote>
<p>So this turned into a nice chat on HN about many of the advances since <a href="https://anil.recoil.org/papers/2020-icfp-retropar">OCaml 5.0</a>
that make this sort of usecase much easier.</p>
<h2><a href="https://anil.recoil.org/news.xml#digesting-ai-disclosure-feedback" class="anchor" aria-hidden="true"></a>Digesting AI disclosure feedback</h2>
<p>I also posted <a href="https://anil.recoil.org/notes/opam-ai-disclosure-update">an update</a> digesting the
public and private feedback on my <a href="https://anil.recoil.org/notes/opam-ai-disclosure">voluntary AI disclosure proposal</a> from a few months ago.</p>
<p>I'm not entirely convinced myself this will see any adoption, but I felt I had
to make the effort.  I do believe that the three concrete steps I proposed
might be useful (promoting maintenance-intent to a first-class opam field,
better tooling for filtering/merging multiple opam repositories, and revisiting
a <a href="https://blog.tangled.org/vouching/">vouching-based reputation</a> system).</p>
<h2><a href="https://anil.recoil.org/news.xml#in-memorium-peter-g-neumann" class="anchor" aria-hidden="true"></a>In memorium: Peter G. Neumann</h2>
<p>I was extremely sad to hear that Peter passed on Sunday at the age of 93. I worked with him extensively in early days of <a href="https://www.cl.cam.ac.uk/research/security/ctsrd/">CTSRD</a> and <a href="https://www.darpa.mil/research/programs/mission-oriented-resilient-clouds">MRC2</a> which funded a lot of our work on <a href="https://anil.recoil.org/projects/unikernels">MirageOS</a>. Peter was the loveliest, most supportive and eternally jolly collaborator, and visiting him in SRI was <a href="https://bsky.app/profile/anil.recoil.org/post/3mm4uq5z5722z">always joyful</a> and quirky.</p>
<p>The NYT did a lovely <a href="https://www.nytimes.com/2026/05/17/obituaries/peter-g-neumann-dead.html">obituary</a>, and I'm going to spend some time finding secondary sources to refresh his <a href="https://en.wikipedia.org/wiki/Peter_G._Neumann">Wikipedia page</a>. If you have any good secondary sources you'd like included please do let me know!</p><h1>References</h1><ul><li>Madhavapeddy (2026). A Proposal for Voluntary AI Disclosure in OCaml Code. <a href="https://doi.org/10.59350/cxypn-ysv27" target="_blank"><i>10.59350/cxypn-ysv27</i></a></li>
<li>Sivaramakrishnan et al (2020). Retrofitting parallelism onto OCaml. <a href="https://doi.org/10.1145/3408995" target="_blank"><i>10.1145/3408995</i></a></li></ul>
