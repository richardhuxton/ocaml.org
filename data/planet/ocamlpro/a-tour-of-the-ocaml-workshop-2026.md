---
title: A tour of the OCaml Workshop 2026
description: This year, for the first time, the Functional Programming Workshops were
  held in Paris, on Inria's, the French National Computer Science Research Institute,
  campus along with a watch party for the ICFP conference at Indianapolis. As each
  year we attended the OCaml Workshop to see the latest advances...
url: https://ocamlpro.com/blog/2026_09_14_ocaml_workshop_2026
date: 2026-09-14T09:41:57-00:00
preview_image: https://ocamlpro.com/assets/img/og_image_ocp_the_art_of_prog.png
authors:
- "\n    Nathan Rebours\n  "
source:
ignore:
---

<p>This year, for the first time, the Functional Programming Workshops were held in Paris, on
Inria's, the French National Computer Science Research Institute, campus along with a watch
party for the ICFP conference at Indianapolis.</p>
<p>As each year we attended the OCaml Workshop to see the latest advances of our
favourite state-of-the-art language, and as usual it was a blast!
People from various companies and labs came to present their latest work in the OCaml ecosystem:
Cambridge University, Inria, LexiFi, Meta, Robur, Sorbonne University and Tarides.
Here's a quick summary of the day's talks.</p>
<h3>A new workflow to build unikernels in OCaml</h3>
<p><em>Romain Calascibetta</em></p>
<p>Romain presented a brand new way to build <a href="https://github.com/Solo5/solo5">Solo5</a> <a href="https://mirageos.org">unikernels</a> in OCaml. It’s based
on two new OCaml components: <a href="https://git.robur.coop/robur/unic">unic</a> and
<a href="https://git.robur.coop/robur/mfetch">mfetch</a>.</p>
<p>This lets you build unikernels without vendoring and recompiling
entire dependency tree, as was the case with the usual <code>opam-monorepo</code> and
<code>mirage</code> approach. With Solo5, in most cases one actually only needs to
recompile C files with the Solo5 toolchain (and any package that depends on
them). <code>unic</code> helps you identify those packages and <code>mfetch</code> sets them up in a
dune <code>vendored_dir</code> so they can be rebuilt with the right toolchain for your
unikernel. The rest of your dependencies, the pure OCaml ones, can just be
installed via opam. No need for dune ports of your entire dependency tree
anymore.</p>
<p><img src="https://ocamlpro.com/blog/assets/img/ocamlworkshop2026_unikernel-unic-mfetch.png" alt="unic and mfetch in action, from Romain's slides"></p>
<p>On the picture above, you can see how they are used to vendor the right set of dependencies when
building a Solo5 unikernel in the
<a href="https://github.com/dinosaure/immuable/blob/690755581e0ffd96425db9398d4c1a11057365e7/GNUmakefile#L1">Makefile</a>
of Romain's <a href="https://github.com/dinosaure/immuable">immuable</a> project. You can
also take a look at the
<a href="https://docs.google.com/presentation/d/1bOXJAPEEjPBx_8i0frThKhLpHd8C7-EPRczEQjvz6JU/edit?slide=id.g3f7268904c5_1_92#slide=id.g3f7268904c5_1_92">slides</a>
(and at Romain's wonderful drawings).</p>
<h3>Compiling FFI-heavy OCaml to WebAssembly: An Experience Report on MOPSA</h3>
<p><em>Reda Boudrouss</em></p>
<p>Reda reported on how they compiled
<a href="https://gitlab.com/mopsa/mopsa-analyzer">MOPSA</a>, a static analysis platform
which depends quite heavily on C/C++ libraries (APRON, GMP, MPFR, LLVM/Clang),
to WebAssembly.</p>
<p>They took a completely different approach to the usual "OCaml in the browser":
no <code>js_of_ocaml</code> or <code>wasm_of_ocaml</code>. They instead compiled the OCaml bytecode
runtime itself, along with all C/C++ dependencies, using
<a href="https://github.com/emscripten-core/emscripten">Emscripten</a>. MOPSA is then
compiled to bytecode and interpreted by the wasm-compiled OCaml runtime.</p>
<p>This approach saved them from rewriting thousands of lines of stubs in
JavaScript or WebAssembly.</p>
<p>This wasn't a walk in the park though: they had to fix some nasty <em>32-bit</em> bugs
to get it to work, and some features don't behave properly because WASM offers
no control over the floating-point rounding mode. But <a href="https://mopsawasm.rboud.com">it's
there</a>!</p>
<p>The result is a not-so-slow web version of MOPSA: 10x slower than native, but
much faster than their existing pure <code>js_of_ocaml</code> version!</p>
<p>This approach can easily be reused to compile any other project with a
completely different set of C dependencies.</p>
<p><img src="https://ocamlpro.com/blog/assets/img/ocamlworkshop2026_wasm-js-mopsa-comparison.png" alt="Javascript vs Wasm comparison, from Reda's slides"></p>
<p>You can read the full article
<a href="https://github.com/rboudrouss/articles/blob/main/Ocaml-Workshop-2026/paper/ocaml-work-2026.pdf">here</a>
if you'd like to know more, or browse the
<a href="https://cdck-file-uploads-us1.s3.dualstack.us-west-2.amazonaws.com/flex020/uploads/ocaml/original/2X/0/0b4801c0377c2e217353284b1e0efe8f192acb52.pdf">slides</a>.</p>
<h3>Security-team address</h3>
<p><em>Edwin Török</em></p>
<p>Edwin gave us an overview of what the <a href="https://ocaml.org/security">OCaml Security
Team</a> did this year. They are the main point of
contact for all security issues in the OCaml world, especially for the compiler
and the ecosystem tools. They published 17 advisories in 2026, across 11
packages; 3 of them were on the OCaml runtime itself (<code>Marshal</code> buffer
over-read, <code>Bigarray.reshape</code> integer overflow, and command injection on
Windows via filename). They are all listed in the <a href="https://github.com/ocaml/security-advisories">security advisories
database</a>. A <a href="https://sympa.inria.fr/sympa/info/ocsf-ocaml-security-announcements">mailing
list</a> is
also available if you want to stay up to date with security advisories. They
also released <a href="https://github.com/hannesm/opam-audit"><code>opam-audit</code></a>, a tool
that audits your current switch against known CVEs. As it is an <code>opam</code> plugin,
you install it once and it is then available on all your switches. The Security
Team holds regular public meetings: the next one is on Tuesday, September 15th
(<a href="https://pad.data.coop/p/RF2XNjlq6">slides</a>).</p>
<h3>A Capability Type System for OCaml</h3>
<p><em>Yoann Padioleau</em></p>
<p>Yoann presented their <a href="https://github.com/aryx/ocaml-caps"><code>Caps</code> library</a>,
inspired by
<a href="https://roscidus.com/blog/blog/2023/04/26/lambda-capabilities/">eio</a>'s
capabilities, which provides fine-grained control over effectful or "dangerous"
resources such as IO, network and system calls, using only existing OCaml
language features.</p>
<p>The principle is that one forbids the use of effectful or dangerous functions
from the standard library or from external libraries in their code, for
instance via <a href="https://semgrep.dev/">Semgrep</a> rules, warnings or any linting
system.</p>
<p>Any function that needs to access those resources must then be granted access
explicitly:</p>
<pre><code class="language-ocaml">val f : &lt; Cap.stdout; Cap.fork; Cap.network; .. &gt; -&gt; int -&gt; int
</code></pre>
<p>If you're unfamiliar with this syntax, the first argument's type is an <a href="https://ocaml.org/docs/objects#immediate-objects-and-object-types">object
type</a>. Just
from the function's signature we know exactly which resources it can access.</p>
<p><code>Caps</code> provides the abstract types for each resource, wrappers around the
standard library and <code>Unix</code> functions such as:</p>
<pre><code class="language-ocaml">val fork : &lt; Cap.fork; .. &gt; -&gt; unit -&gt; int
</code></pre>
<p>and an entry point:</p>
<pre><code class="language-ocaml">val main : (all_caps -&gt; 'a) -&gt; 'a
</code></pre>
<p>which can only be called once in a program and is the only way to access
capabilities. One then downcasts the <code>all_caps</code> object to pass it around, and
the typechecker will ensure no one accesses undeclared resources.</p>
<p>Yoann's slides are available <a href="https://github.com/aryx/ocaml-caps">here</a> if you
want to learn more, or you can even watch the
<a href="https://www.youtube.com/watch?v=4t_2wLz9EOo">talk</a>.</p>
<h3>Runtime Types at LexiFi: Experience Report</h3>
<p><em>Nicolas Ojeda Bär</em></p>
<p>Nicolas presented LexiFi's compiler extension for runtime types, an extension
that grants access to type-directed programming in your OCaml codebase.</p>
<p>They define a type for the type witness of a type <code>'a</code>:</p>
<pre><code class="language-ocaml">type 'a ttype
</code></pre>
<p>Their patched compiler interprets <code>[%t: &lt;type-expr&gt;]</code> and generates a
<code>&lt;type-expr&gt; ttype</code> type witness. This cannot be done by a simple PPX, because
PPXs don't have access to type information; instead, it is done during or after
type checking.</p>
<p>They also have a GADT <code>xtype</code> that describes a type's shape:</p>
<pre><code class="language-ocaml">type 'a xtype =
  | Unit : unit xtype
  | Bool : bool xtype
  | Char : char xtype
  | Int : int xtype
  | Float : float xtype
  | String : string xtype
  | Option : 'b ttype -&gt; 'b option xtype
  | List : 'b ttype -&gt; 'b list xtype
  ...
</code></pre>
<p>and it can be obtained from a type witness with:</p>
<pre><code class="language-ocaml">val xtype_of_ttype : 'a ttype -&gt; 'a xtype
</code></pre>
<p>With that, they can easily generate generic printers, equality functions, JSON
de/serializers, etc.:</p>
<pre><code class="language-ocaml">val print : t:'a ttype -&gt; 'a -&gt; string
</code></pre>
<p>Such generic functions always take the type witness as an extra <code>~t</code> labeled
argument. When this argument is omitted at a call site, their compiler
extension automatically adds it, with the type witness for the inferred type of
the argument, whenever possible.</p>
<p>This also comes with type properties <code>[@t &lt;property-name&gt; = &lt;string&gt;]</code> which
are embedded in the type witness so that generic functions can eventually
interpret them. This makes it possible to customize the behaviour of a generic
function for specific types, e.g. to mark that a record field should use a
different name when serialized to JSON.</p>
<p>Compared to the PPX <code>[@@deriving ...]</code> approach, this might offer lower
performance, as the functions aren't specialized for a specific type, but it
comes with much better ease of use and flexibility: once one has the type
witness for a type, they can pass it to any number of functions.</p>
<p>You can browse Nicolas' <a href="https://www.lexifi.com/blog/ocaml/ocaml-workshop-2026/">slides</a>
and read more about LexiFi's runtime types
<a href="https://www.lexifi.com/blog/ocaml/ocaml-workshop-2026/report.pdf">here</a>.</p>
<h3>PPXs That Reach Their Destinations</h3>
<p><em>Gabriel Radanne</em></p>
<p>Gabriel presented their proposal for exposing a low-level API allowing PPXs to
transform functions into destination passing style (DPS for short).</p>
<p>OCaml 4.14 introduced the <code>[@tail_mod_cons]</code> transformation, which allows one
to mark a function so the compiler can turn a natural-looking recursive
function into a tail-recursive one, provided the recursive call is in tail
position modulo a constructor application. For instance, the following map
implementation for lists:</p>
<pre><code class="language-ocaml">let[@tail_mod_cons] rec map f l =
  match l with
  | [] -&gt; []
  | x :: xs -&gt;
    let y = f x in
    y :: map f xs
</code></pre>
<p>is, roughly speaking, transformed into something that could be written as the
following pseudocode:</p>
<pre><code class="language-ocaml">let rec map_dps f l dst idx =
  match l with
  | [] -&gt; dst.idx &lt;- []
  | x :: xs -&gt;
    let y = f x in
    let dst' = y ::{mutable} Hole in
    dst.idx &lt;- dst';
    map_dps f xs dst' 1
</code></pre>
<p>Destinations can be seen as OCaml values with holes (write-once pointers) that
can be filled later.</p>
<p>The proposal is to expose the following in the <code>Obj</code> module:</p>
<pre><code class="language-ocaml">type 'a dest
val set_dest : 'a dest -&gt; 'a -&gt; unit
</code></pre>
<p>and a compiler-interpreted extension <code>[%ocaml.value_with_holes ...]</code> that would
be replaced by the value, with holes, and a tuple made of all the destinations
one needs to fill them:</p>
<pre><code class="language-ocaml">type t = Foo of int * float array * string

let holed : t * (float Obj.dest * string Obj.dest) =
  [%ocaml.value_with_holes Foo (21, [| [%hole] |], [%hole])]
</code></pre>
<p>This is not meant to be used directly, but rather through PPXs that would
transform functions in contexts where the current compiler transformation
cannot be applied.</p>
<p>You can read the full proposal
<a href="https://github.com/Drup/RFCs/blob/master/rfcs/exposed_dest.md">here</a>, and see
how Gabriel and their team apply this to transform tail calls modulo
Async/Await
<a href="https://icfp26.sigplan.org/details/icfp-2026-icfp-papers/12/Tail-Modulo-Async-Await">here</a>.</p>
<h3>Slipshow: chill coding with OCaml</h3>
<p><em>Paul-Elliot Anglès d'Auriac</em></p>
<p>Paul-Elliot presented new features of his
<a href="https://github.com/panglesd/slipshow">Slipshow</a> project. Slipshow is a tool to
craft slideless presentations: your presentation is a continuously rolling
slip, into which you can integrate handwriting and animations. A console
command gives you full control over your animations, hand-made or imported. You
can even switch between WYSIWYG and WYGIWYS editors!</p>
<p>Slipshow is written 100% in OCaml, and the compiler's guarantees strengthened
it, from safety to speed, making the developer experience much smoother than
with the original JavaScript prototype. Paul-Elliot summed it up by saying that
it's OCaml's exceptional design, editor tooling and robust ecosystem that made
working on this project so <em>chill</em>: guided by the compiler, without fear of
introducing bugs.</p>
<p><img src="https://ocamlpro.com/blog/assets/img/ocamlworkshop2026_slipshow.png" alt="Slipshow overview, from Paul-Elliot's slides"></p>
<p>If you want to see it in action, take a look at Paul-Elliot's
<a href="https://choum.net/panglesd/slides/ocaml-workshop-2026/chill-coding.html">slides</a>.</p>
<h3>When Type Checking Goes Wrong, but Keeps Going</h3>
<p><em>Xavier Van De Woestyne</em></p>
<p><a href="https://github.com/ocaml/merlin">Merlin</a> is a well-known (multi-)editor helper
for OCaml. Where the compiler stops at the first error, <code>merlin</code> displays all
of them, even when the code block is unfinished; Xavier presented the recovery
system that makes this possible. For this, <code>merlin</code> uses a patched version of
the type checker. That system worked for a while, but the maintenance burden of
updating it with every compiler release is not sustainable.</p>
<p>With the help of the compiler team, they <a href="https://github.com/ocaml/ocaml/pull/14241">finally upstreamed this typing
recovery feature</a> into the compiler
itself. The next OCaml 5.6.0 release will offer a new flag <code>-typing-recovery</code>
that can be used by <code>merlin</code> to show <strong>all</strong> the errors found.</p>
<p>Because the recovery mechanism works on incomplete or incorrect code, the
errors that follow the first one might be inaccurate; this mode is therefore
not recommended for regular users, who should rely on the regular compiler type
error reporting.</p>
<p>This will greatly ease support for new compilers in <code>merlin</code>, though there are
still some patches to apply, such as their parsing recovery mechanism.</p>
<p>You can find Xavier's slides
<a href="https://xvw.lol/talks/ocamlworkshop2026-tyrec.pdf">here</a>.</p>
<h3>JSON parsing in OxCaml: fast, but not too fast</h3>
<p><em>Artem Pianykh</em></p>
<p>Artem presented their SIMD JSON parser implementation, written in OxCaml.</p>
<p>They used <a href="https://oxcaml.org/">OxCaml</a> (Jane Street's current fork of OCaml,
which uses the
<a href="https://ocamlpro.com/blog/2024_03_18_the_flambda2_snippets_0/">flambda2</a>
optimisation backend, developed here at OCamlPro) to provide a lightning fast
JSON parsing library, inspired by C++'s
<a href="https://github.com/simdjson/simdjson">simdjson</a>.</p>
<p>Thanks to OxCaml features such as stack allocation, unboxed types and — of
course — SIMD instruction support, plus a bit of elbow grease, they managed to
write a parser up to 9 times faster than
<a href="https://github.com/ocaml-community/yojson">Yojson</a>.</p>
<p>The library exposes two parsing interfaces: one that produces the usual
<code>Json.t</code> variant type, and a lower-level but much faster one.</p>
<p>It is still roughly 3 times slower than <code>simdjson</code>, but there's room for
improvement. Artem mentioned that unboxed variants could drastically improve
the <code>Json.t</code> parser, among other things.</p>
<p>If you'd like to know more, you can read Artem's <a href="https://pianykh.com/blog/posts/2026-08-24-ocaml-workshop-json-parsing.html">detailed
article</a>
or jump straight to the
<a href="https://github.com/artempyanykh/simdjson-oxcaml">code</a>.</p>
<h3>A new implementation of Short-paths</h3>
<p><em>Ulysse Gérard, Paul-Elliot Anglès d'Auriac</em></p>
<p>Ulysse presented a new design to select short paths for printing type error
messages.</p>
<p>Type short-paths are very useful to make types easier to read and understand at
first glance. Instead of displaying <code>Int.t</code>, the compiler shows <code>int</code>; and if
you defined <code>type foo = string * int * float</code>, <code>foo</code> is selected by the
compiler. However, it is also very opinionated, as we saw clearly in the
survey/poll conducted live during the presentation. In more specific cases,
such as defining <code>foo</code> in a module <code>Bar</code>, some prefer to keep <code>foo</code> when
including that module, while others prefer <code>Bar.foo</code> to keep the information
about the origin.</p>
<p>Currently there are 2 ways to compute short paths: one implemented in the
compiler to display error messages with the <code>-short-paths</code> option, and the
other in Merlin. The compiler determines short paths with a lazy breadth-first
search in the environment until it finds an adequate candidate. It is quite
costly, but this is not an issue for the compiler, which shows 1 error at a
time. Merlin's method is a complex engine that explores the possible branches
more deeply when looking for a short path and, to gain performance, cuts some
of them when needed. It results in a much faster and more accurate short-path
selection.</p>
<p>Merlin's short-path detection has a quite high maintenance burden. The idea of
Merlin's team is to have a new short-path mechanism in the compiler itself, one
that would remain maintainable and perform well enough to display all type
error messages. That new process takes advantage of the compiler to gather
information at typing time and build the set of paths used in the source; that
set is reused later, so printing doesn't slow down compilation. From it, a
domain of discourse is built at printing time, following a given set of
inclusion rules: for example, if a module is opened, all its types are
included, and if a module is renamed, all the types defined by the original and
by the substitution are added to the domain of discourse. To select the
shortest path, they build a priority list sorted by cost (by length, with a
malus for the presence of double underscores), then they canonicalise the
first-level path of that list; if the shortest canonicalised path is valid in
the current environment, it is selected. Otherwise, it loops back to the
canonicalisation step.</p>
<p><img src="https://ocamlpro.com/blog/assets/img/ocamlworkshop2026_short-paths.png" alt="The proposed short-path algorithm, from Ulysses's slides"></p>
<p>They tested that algorithm on OxCaml, and their prototype showed much better
accuracy in short-path selection (with respect to their requirements) than both
the compiler and Merlin, but it remains slower than Merlin. There is some
ongoing work to gain performance.</p>
<p>You can find their slides
<a href="https://choum.net/panglesd/slides/ocaml-workshop-2026/short-path.html">here</a>.</p>
<h3>Towards a Benchmarking Service for OCaml</h3>
<p><em>Luis Eduardo de Souza Amorim, Tim McGilchrist</em></p>
<p>Tim presented their new benchmarking framework for OCaml compilers. It consists
of a series of tools to run, orchestrate and visualize several types of
benchmarks on your computer.</p>
<p>Historically, <a href="https://github.com/ocaml-bench/sandmark">Sandmark</a> was used to
benchmark OCaml compilers, but it is more focused on GC performance over a
multitude of small code pieces. In addition to Sandmark's micro-benchmarks,
other <a href="https://github.com/ocaml-bench/macro-benches">macro-benchmarks</a> were
added. The idea is to test real-world applications, configured to have relevant
data, inspired by Java's <a href="https://www.dacapobench.org/">Da Capo</a> benchmark
suite. They selected a dozen OCaml ecosystem tools and wrote scripts to run
them on edge cases that highlight performance data. You can add your own tool
to the benchmarks if you want to track regressions. The projects are built and
run in isolation, with <a href="https://perfwiki.github.io/main/"><code>perf</code></a> and
<a href="https://github.com/sadiqj/runtime_events_tools"><code>olly</code></a> wrapping them to
collect system and OCaml runtime data respectively. You can see the results of
the macro-benchmarks for the OCaml 5.4.1 compiler, comparing the stable release
with a flambda-enabled build.</p>
<p><img src="https://ocamlpro.com/blog/assets/img/ocamlworkshop2026_macro-benchmark-flambda.png" alt="macro-benchmarks for OCaml 5.4.1 with flambda enabled relative to unmodified baseline"></p>
<p>To round out this work, they want to offer these benchmarking tools <em>as a
service</em>, triggered by a PR in CI, by regular jobs for OCaml releases and
compiler options, etc.</p>
<p>We'd love to see Alt-Ergo added to the benchmarks, along with the possibility
of running them for OxCaml with different flambda2 configurations!</p>
<p>You can take a look at Tim's extended
<a href="https://cdck-file-uploads-us1.s3.dualstack.us-west-2.amazonaws.com/flex020/uploads/ocaml/original/2X/8/871cbbaaf1f5e0d3804c000d0144819931024d29.pdf">slides</a>.</p>
<h3>First Class Docs in OCaml</h3>
<p><em>Jon Ludlam</em></p>
<p>Jon proposed a reflection on how to handle the documentation of a project
together with that of its dependencies.</p>
<p><a href="https://github.com/ocaml/odoc"><code>odoc</code></a> is the main tool to generate
documentation in OCaml ecosystem. It is used by external tools to build cross package documentation,
from <a href="https://erratique.ch/software/odig"><code>odig</code></a> that generates a full opam
switch documentation to
<a href="https://github.com/ocurrent/ocaml-docs-ci"><code>ocaml-docs-ci</code></a> that generates the
online documentation for <code>ocaml.org</code> opam repository packages. It is also used
by <code>dune</code> with 2 targets, <code>@doc</code> that generates only the package documentation,
and <code>@doc-new</code> that generates also the documentation for package
dependencies.</p>
<p><img src="https://ocamlpro.com/blog/assets/img/ocamlworkshop2026_odoc-comparison.png" alt="Documenation tools comparison, from Jon's slides"></p>
<p><code>dune @doc</code> and <code>odig</code> still use <code>odoc</code> CLI in version <code>1</code>, missing new
features introduced in newer versions. Looking at how <code>odoc</code> is used in
<code>ocaml-docs-ci</code> and <code>dune @doc-new</code>, the main difficulty is to build dependencies
documentation. As <code>odoc</code> uses <code>.cmt</code> and <code>.cmti</code> artefacts and
reimplements its own module system (in order to retrieve documentation
information dropped by the compiler), it needs to be able to link each <code>.odoc</code>
to an already present <code>.odoc</code> file of each dependency. Jon propose to consider
<code>odoc</code> no longer as an optional package for documentation but a default package
installed in each switch. There is already an opam plugin,
<a href="https://github.com/jonludlam/odd"><code>odd</code></a> that wraps
each package install with documentation generation, allowing easy packages cross-reference.
It provides switch-wide documentation search for direct users or editor completion.</p>
<p>This still has quite an impact on switch size and build times (~5% at the moment) but there is ongoing work to reduce the overhead.</p>
<p>If you want to learn more, you can watch <a href="https://www.youtube.com/watch?v=fqlH5hwN_oQ">Jon's
talk</a>, or browse his
<a href="https://jon.recoil.org/talks/first_class_docs/firstclassdocs.html">slides</a>.</p>
<h3>In the end</h3>
<p>We'd like to thank the organizers and INRIA for hosting this event here in
Paris. It was a great occasion to hear the latest updates from the vibrant
OCaml community in the French capital, and to meet and discuss with its members
in person.
We'd love to see this turn into a yearly recurring event, at least when ICFP's hosted in distant lands.</p>

