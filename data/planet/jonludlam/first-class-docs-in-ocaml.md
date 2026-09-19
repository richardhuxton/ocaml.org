---
title: First Class Docs in OCaml
description:
url: https://jon.recoil.org/blog/2026/06/first-class-docs.html
date: 2026-06-22T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---


      <p>The quality of reference documentation available in OCaml for libraries has improved enormously over the past few years. Docs are now built for every package in opam, hosted on ocaml.org, with search, source rendering, inline media, a navigation sidebar and many other features.</p>
<p>However, it's still not the case that documentation in OCaml has what I'd call "First Class" status. What do I mean by this? Well, let's give some examples of what I think it <em>should</em> look like.</p>
<ol>
<li>High quality API and package documentation should be installed with every opam package, meaning that all build systems ought to be able to do this, and that docs should be checked by CI systems.</li>
<li>Editors should offer completion in docs, highlight any errors when you're editing with clear actions to fix them, and display rendered docs natively.</li>
<li>It should be possible to read and search through the documentation easily, through your IDE and in the terminal.
So why aren't we there yet? What needs to be done? Let's first start by explaining a little about why it's harder in OCaml than in many other languages. If you already know this, skip to the <a href="https://jon.recoil.org/#what_to_do">what to do about it</a> section!</li>
</ol>
<h2>Why it's tricky</h2>
<p>API docs in OCaml are written using specially formatted comments, much like they are in <a href="https://www.doxygen.nl/">many</a> <a href="https://haskell-haddock.readthedocs.io/latest/markup.html">other</a> <a href="https://doc.rust-lang.org/rust-by-example/meta/doc.html">languages</a>. We usually write our documentation in <code>mli</code> files:</p>
<pre><code class="language-ocaml">type t
(** The docs for the type [t] go here *)

val wrangle : t -&gt; t list
(** Here we can talk about [wrangle] and how it relates to {!t} *)
</code></pre>
<p>What makes OCaml more tricky than many other languages is OCaml's module system. A simple example is this:</p>
<pre><code class="language-ocaml">type t
(** Here are my docs for [t] *)

module TMap : Map.S with type t = t
(** This module is a map from the standard library *)
</code></pre>
<p>module <code>TMap</code> is a module found within <em>our</em> module, containing lots of types and values that all need docs, but its definition is in <em>another</em> module - in the standard library in this case. The standard library isn't particularly special, and functors aren't the only way this can happen, so to produce the correct docs we need to have access to the docs of <em>all</em> of our dependencies, just in case they turn out to be necessary to fully document our own interfaces.</p>
<p>The information we need is all in the typedtrees - the <code>.cmt</code> and <code>.cmti</code> files - of our own files and all our dependencies, but there's a non trivial amount of work that goes into processing these, so we effectively <em>compile</em> those <code>.cmti</code> files into <code>.odoc</code> files to make this process efficient.</p>
<p>The upshot of this is that in order to produce fully processed docs for any particular package, we need to have the <code>.odoc</code> files for all of the dependency packages available somewhere.</p>
<p>The approaches that work today all build the docs <em>after</em> the package has been installed, which is how tools like like <a href="https://erratique.ch/software/odig">odig</a> and <a href="https://jon.recoil.org/reference/nox-odoc-driver/">odoc_driver</a> work. But what would it take to ensure it happens at package install package, and for <em>all</em> packages?</p>
<h2>What to do?</h2>
<p>Odig exists, of course, and it does a great job of building docs when you ask for them. But it's an optional install, and the odoc files it builds are squirreled away in its own <code>var/cache/odig</code> directory, and not necessarily up to date, by design.</p>
<p>The key thing is to define the standard place in the opam tree where the odoc and odocl files will live, and ensure that packages are responsible for doing this. With this in place, we simply know that, if a package is installed, its libraries will have their compiled module docs installed and its package docs will be installed in their appropriate places. This gives all other tooling a reliable source of <code>.odoc</code>/<code>.odocl</code> files for whatever they need them for.</p>
<h3>Build system and CI integration</h3>
<p>The most-used developer tool to build docs today is dune, and it's woefully short of where we'd like it to be. Without the odoc files from other packages the docs it builds are full of unresolved links and unexpanded modules, and the error messages it gives are almost useless.</p>
<p>We're working on a "local" fix for this, meaning that dune will know how to build docs for the dependency packages. The docs produced are far better, the error messages much more useful and it will greatly improve the standard of self-published docsets and what's published on ocaml.org. However, it would be far better if the docs for the dependency packages were installed already. Then dune would only have to concern itself with the docs of the workspace packages, and the implementation would be simpler.</p>
<p>For a truly first-class docs experience, <em>all</em> OCaml build systems would have to be able to build docs themselves. We've got quite clear instructions on <a href="https://jon.recoil.org/reference/nox-odoc/driver.html">how to drive odoc</a> at a low level, but we need better instructions at a higher level, for which the main source of info right now is the reference driver.</p>
<p>As a stop-gap measure, we could tweak <a href="https://jon.recoil.org/reference/nox-odoc-driver/">odoc_driver</a> and use it as a black box for doing the job. We can use this for all packages that don't yet install their own docs as a post-processing step after the opam build. Clearly this would be significantly less developer friendly than integrating it properly with the build system. For example, it wouldn't be able to do incremental rebuilds.</p>
<p>Once we've got the build systems working, checking things with CI would be pretty straightforward. Even before we're at that point, we could wire up <a href="https://jon.recoil.org/reference/nox-odoc-driver/">odoc_driver</a> which already provides reasonable error messages.</p>
<h3>Editor support</h3>
<p>The main additional features I'd like to see in editors would be:</p>
<ul>
<li>Tab-completion of references. With all the odoc files installed this would be easy enough, but I'm not sure if it would be quick enough right now. If it isn't, it should be straightforward enough to add in an index.</li>
<li>Error reporting. This is a bit trickier, but is not dissimilar from how Merlin does its work - we would rely on the build system to build the odoc files for the rest of the project, then have something that can report errors from a buffer.</li>
<li>Preview of docs. When incremental builds of docs are available, this should be straightforward. The dune rules we've been working on already can do this.</li>
</ul>
<h3>Search</h3>
<p>And, of course, there's search. We currently have search in odoc output powered by <a href="https://jon.recoil.org/reference/nox-sherlodoc/">Sherlodoc</a>. However, Sherlodoc is focused on searching of API functions, and doesn't index mld pages at all. We can augment sherlodoc with search powered by other mechanisms, such as BM25 or using LLM embeddings, as we demonstrated <a href="https://jon.recoil.org/blog/2025/08/ocaml-mcp-server.html">last year</a>, to provide a more holistic search function for package documentation.</p>
<p>So now let's focus on how we get to the point where all the odoc files are installed.</p>
<h2>The path to success</h2>
<h3>What gets installed?</h3>
<p>First we need to define what actually gets put into the filesystem. The docs we've got on driving odoc are great at the level of individual commands, but not anything with a ecosystem-wide view on actually how to lay out the files, where to put package docs vs library docs vs rendered source files, what we definitely need installed, and what's more of a convenience file that's generated while we're building docs.</p>
<p>The starting point for this is probably what <a href="https://jon.recoil.org/reference/nox-odoc-driver/">odoc_driver</a> produces as part of the builds for ocaml.org. We can easily see what these are by just running <code>odoc_driver</code> and supplying output paths for the intermediate files. For example, <code>odoc_driver --odoc-dir _odoc --odocl-dir _odocl</code>. These files are more-or-less what should be installed somewhere in your opam switch directory.</p>
<p>Having figured all this out, we then need to figure out how to make that happen. Let's first consider the ambitious plan - one that might take some time to do.</p>
<h3>The ambitious plan</h3>
<p>Let's upstream the necessary parts of odoc into <a href="https://github.com/ocaml/ocaml">ocaml/ocaml</a>!</p>
<p>If we want packages to install their own docs, and we need odoc available to do that, then we should arrange for odoc to be part of the first package that needs docs - and that would be the compiler itself. Otherwise, the way we work with stdlib will always be different from many other packages.</p>
<p>We don't need <em>all</em> of odoc, just the parts that are essential for other packages to be able to produce <em>their</em> docs correctly. So essentially we need enough to run the compile and link phases, and at least one output format.</p>
<p>All of the tooling for additional uses - search, html output, completion, and so on, can be packaged up in additional tools packages. It's not completely clear exactly where to draw the line, so we'll have to figure that out as a community.</p>
<p>The ocaml repository already contains <a href="https://ocaml.org/manual/5.5/ocamldoc.html">ocamldoc</a> which is used to build the manual for OCaml. We've had support for building the manual with odoc since <a href="https://github.com/ocaml/ocaml/pull/9997">OCaml 4.13</a>, so replacing ocamldoc with odoc isn't too far of a stretch, particularly as odoc has really been the standard doc tool for OCaml for many years now.</p>
<h3>The additional quicker plan</h3>
<p>The ambitious plan will take a while to sort out, but while we're working on it we can explore what life would be like with this all working using a shortcut. If we have a tool that can install the odoc files on behalf of the packages without having to actually modify the packages to do it, we can work on some of the integration tasks and try them out.</p>
<p>Odig obviously does a lot of this already, but it's missing quite a few of the modern odoc 3 features. Odoc_driver is closer to what we'd need, but we need a couple of tweaks to make it really fit.</p>
<p>We can then execute this package-by-package after they're installed, and if we do this using opam's post-install hooks, we can simulate what it would be like had the packages installed the docs themselves. It doesn't <em>quite</em> work, an obvious example of which is that opam's view of what the packages have installed won't contain the odoc files. But it's certainly good enough to try out things like CLI-based doc viewing, or completion of references.</p>
<p>With these changes, the "energy barrier" to writing good docs for your OCaml packages will be significantly reduced, and the ubiquity of the docs in all opam switches will result in tools and workflows that really make the most of this enormously useful, but currently underused resource.</p>

    
