---
title: Updating to the OxCaml 5.2.0-minus39 opam packaging
description: How the OxCaml overlay's guard packages keep incompatible releases out,
  and how to contribute to it with your own packages.
url: https://anil.recoil.org/notes/oxcaml-opam-guards
date: 2026-08-08T00:00:00-00:00
preview_image: https://anil.recoil.org/images/faces/avsm.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>This <a href="https://anil.recoil.org/notes/bushel-lives">website</a> is written using the <a href="https://oxcaml.org">OxCaml</a> language extensions from Jane Street, so I can experiment with <a href="https://anil.recoil.org/notes/oxcaml-httpz">zero allocation</a> frameworks written in OCaml.  Since <a href="https://www.dra27.uk">David Allsopp</a> released <a href="https://github.com/oxcaml/opam-repository/tree/main/packages/oxcaml-compiler/oxcaml-compiler.5.2.0minus39">oxcaml-minus39</a> a few weeks ago, I've updated to that. This first required <a href="https://github.com/oxcaml/opam-repository/pull/59">figuring out the packaging</a> of some of the third-party dependencies that I require.
Using OxCaml is more complex than vanilla OCaml since many existing
packages fail to compile due to it requiring additional type
annotations or eta-expansions.  However, there's a neat set of guard
packages present in the OxCaml opam overlay that make it straightforward to find
compatible versions.</p>
<p>My own need was for <a href="https://github.com/ocaml-multicore/eio">Eio</a>, as Eio &lt;=1.3
didn't build under OxCaml. The Jane Street overlay shipped an <code>eio.1.3+ox</code>
fork built from a patched branch. <a href="https://roscidus.com">Thomas Leonard</a> and I fixed the OxCaml build
issues in <a href="https://github.com/ocaml-multicore/eio/releases/tag/v1.4">Eio 1.4</a>,
but found ourselves <a href="https://notes.roscidus.com/2026/08/07/">blocked from using it</a> due to the system of guard packages.</p>
<p>I figured I'd dig into how this works and make it easier to contribute fixes to Jane Street for third-party packages like mine.
I've opened <a href="https://github.com/oxcaml/opam-repository/pull/59">PR#59</a> on OxCaml's repo, and here's an explanation of <a href="https://anil.recoil.org/news.xml#how-the-package-guards-work">how the OxCaml guard machinery works</a>,
how to <a href="https://anil.recoil.org/news.xml#allowing-newer-packages-through-the-guards">let newer releases through</a>,
and <a href="https://anil.recoil.org/news.xml#whats-new-in-oxcaml-minus39">what's new in ox-minus39</a> itself.</p>
<h2><a href="https://anil.recoil.org/news.xml#how-the-package-guards-work" class="anchor" aria-hidden="true"></a>How the package guards work</h2>
<p>The <a href="https://github.com/oxcaml/opam-repository">OxCaml opam-repository</a> is an
opam overlay that you add alongside the main
<a href="https://github.com/ocaml/opam-repository">ocaml/opam-repository</a>. It supplies
both the <a href="https://anil.recoil.org/projects/oxcaml">OxCaml</a> compiler itself as well as <code>+ox</code> forks of upstream
packages that don't yet compile with the extended compiler.</p>
<p>This overlay repository seeks to ensure that opam can never resolve <em>around</em>
one of the patched packages (e.g. if upstream releases a newer version), and
install the unpatched upstream version instead.  The OxCaml maintainers need to
explicitly test the new version and permit it through the OxCaml package
guards.</p>
<p><a href="https://www.dra27.uk">David Allsopp</a> did this by adding a couple of opam meta-packages per forked package:
For <code>eio</code>, for example, we get:</p>
<ul>
<li><code>oxcaml-eio.guard</code>, meaning "the patched eio is <em>not</em> installed here".</li>
<li><code>oxcaml-eio-patches.enabled</code> meaning "the patched package <em>is</em> installed here".</li>
</ul>
<p>The two packages conflict with each other, and a third <code>oxcaml-patch-guards</code> package
uses a disjunction of dependencies to require one or the other.
Crucially, the guard packages' <code>conflicts:</code> field lists the upstream versions that
are forbidden if we go down the 'unpatched' package route:</p>
<pre><code>conflicts: [ "oxcaml-eio-patches" "eio" {&lt; "1.4"} ]
</code></pre>
<p>This means that every <code>eio</code> version before 1.4 was <a href="https://github.com/ocaml-multicore/eio/pull/898">incompatible</a>
with OxCaml. The solver is then free to look for an <a href="https://notes.roscidus.com/2026/07/24/">eio 1.4+ release</a>
in upstream opam-repository and select that if available.</p>
<h2><a href="https://anil.recoil.org/news.xml#allowing-newer-packages-through-the-guards" class="anchor" aria-hidden="true"></a>Allowing newer packages through the guards</h2>
<p>The nice property of this design is that retiring an OxCaml fork does
not require deleting the <code>+ox</code> package, as someone on an older OxCaml compiler still
needs it.</p>
<p>For mdx, for example, it just requires patching the guard package to allow
the new version through:</p>
<pre><code class="language-diff">--- a/packages/oxcaml-mdx/oxcaml-mdx.guard/opam
+++ b/packages/oxcaml-mdx/oxcaml-mdx.guard/opam
@@ -6,7 +6,7 @@ authors: "David Allsopp"
 license: "CC0-1.0+"
 homepage: "https://oxcaml.org"
 bug-reports: "https://github.com/oxcaml/opam-repository/issues"
-conflicts: [ "oxcaml-mdx-patches" "mdx" ]
+conflicts: [ "oxcaml-mdx-patches" "mdx" {&lt; "2.6.0"} ]
 messages: ["WARNING! An older version of OxCaml is being installed" {oxcaml:version = "archived"}]
 depends: [
   ("oxcaml-merlin" {post} | "oxcaml-merlin-patches" {post})
</code></pre>
<p>So this now has mdx working, but then I <a href="https://anil.recoil.org/notes/2026w30">needed</a>
to add the <code>unix</code> dependency that mdx 2.6 now needs in the dune rules.
<a href="https://github.com/ocaml/dune/pull/15592">That fix</a> shipped in
the <a href="https://github.com/ocaml/dune/releases/tag/3.24.2">dune 3.24.2</a> release,
so I've added that into the diff too.</p>
<p>Is this a good system then? It's certainly an ingenious use of the opam solver,
but the packaging encoding itself is difficult to parse at first glance (although made
much easier with agents; I used my <a href="https://anil.recoil.org/notes/language-integrated-llms">local deepseek agent</a>
to help me out. I'm hopeful that as <a href="https://ryan.freumh.org">Ryan Gibb</a> advances his <a href="https://anil.recoil.org/papers/2026-package-calculus">package management calculus</a>, we'll
be able to improve the syntax UI (and the error messages!) in time.</p>
<h2><a href="https://anil.recoil.org/news.xml#whats-new-in-oxcaml-minus39" class="anchor" aria-hidden="true"></a>What's new in OxCaml minus39?</h2>
<p>I updated my <a href="https://anil.recoil.org/notes/aoah-2025-25">Claude OCaml Marketplace</a> skill, which you can <a href="https://github.com/avsm/ocaml-claude-marketplace/tree/main/plugins/ocaml-dev/skills/oxcaml">find here</a>. A few highlights from minus31-39 that caught my eye are:</p>
<ul>
<li>Domain preemption has an initial implementation, so a tight compute loop can be interrupted at a poll point instead of starving everything else sharing the domain.</li>
<li>Arrays of unboxed elements now have their own packed representations (<code>int8 array</code>, <code>float32 array</code>, <code>vec512 array</code> etc), so an array of small numbers isn't a word per element.</li>
<li>The <code>borrow_</code> operator lets you pass a <code>unique</code> value where an <code>aliased</code> one is wanted and still own it uniquely afterwards.</li>
<li>Runtime metaprogramming's quotes and splices (<code>&lt;&lt;e&gt;&gt;</code> and <code>$(e)</code>) only need <code>-extension-universe beta</code>, and <code>[%eval]</code> is now an ordinary <code>Eval.eval</code> function.</li>
<li>There's an LLDB language plugin to ease debugging oxidised binaries.</li>
<li>Implicit kinds are enabled so fewer kind annotations need writing out by hand.</li>
<li>Mode syntax improved so that <code>(e : @ modes)</code> parses directly, removing the need for the <code>: _ @ modes</code> workaround.</li>
<li>Any two-constructor variant can now be <code>[@@or_null]</code> giving custom <code>Nope | Yep of 'a</code> types the same non-allocating null encoding as the built-in <code>'a or_null</code>.</li>
<li><code>-O4</code> exists, being <code>-O3</code> plus the mysterious "reaper" pass, which I need to investigate.</li>
<li>Some things were deleted, such as the block-index array syntax (<code>.(0)</code>, <code>.:(0)</code>, <code>.L(i)</code>), float record indices, and the <code>_internal</code> kind escape hatches. I never used these so wasn't affected.</li>
</ul>
<p>This website is now running on minus39 using <a href="https://github.com/avsm/oxmono/tree/minus39">my bleeding edge monorepo</a>. If you're interested in trying OxCaml yourself, the compiler's stable, but the packaging is still fluid and needs some expertise in how opam works for you to add your own overrides. Hopefully this post will help you navigate that a bit more easily!</p><h1>References</h1><ul><li>Madhavapeddy (2026). Language integrated LLMs as an OCaml function. <a href="https://doi.org/10.59350/61cdd-r5a25" target="_blank"><i>10.59350/61cdd-r5a25</i></a></li>
<li>Madhavapeddy (2026). My (very) fast zero-allocation webserver using OxCaml. <a href="https://doi.org/10.59350/9c6bz-kb659" target="_blank"><i>10.59350/9c6bz-kb659</i></a></li>
<li>Madhavapeddy (2025). Arise Bushel, my sixth generation oxidised website. <a href="https://doi.org/10.59350/0r62w-c8g63" target="_blank"><i>10.59350/0r62w-c8g63</i></a></li>
<li>Gibb et al (2026). Package Managers à la Carte: A Formal Model of Dependency Resolution. <a href="https://doi.org/10.1145/3828699" target="_blank"><i>10.1145/3828699</i></a></li></ul>
