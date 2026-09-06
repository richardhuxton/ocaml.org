---
title: 'OCaml 5.5 Summary: What''s New in the Latest OCaml Update'
description: What's new in OCaml's latest 5.5 update? New features, performance improvements,
  and bug fixes.
url: https://tarides.com/blog/2026-06-23-ocaml-5-5-summary-what-s-new-in-the-latest-ocaml-update
date: 2026-06-23T00:00:00-00:00
preview_image: https://tarides.com/blog/images/camels-sunset-1360w.webp
authors:
- Tarides
source:
ignore:
---

<p>The newest version of OCaml is officially out! The latest update comes with several improvements. Of note are Relocatable OCaml, Garbage Collection Pacing changes boosting overall performance, polymorphic function parameters, and several bug fixes.</p>
<p>This post will take you through some of the biggest changes, so let’s get started!</p>
<h2>Polymorphic Function Parameters</h2>
<p>Prior to 5.5, OCaml’s functions were already polymorphic, but their parameters couldn’t be. If you passed a function as an argument to another function, the type checker would assume your function was monomorphic. You couldn’t call it at multiple different types in the same function body. With OCaml 5.5, function parameters can carry explicit polymorphic type annotations.</p>
<p>This is a feature upstreamed from Jane Street’s <a href="https://oxcaml.org">OxCaml</a> branch of OCaml, which gives the community access to their production compiler and experimental extensions. Some of the extensions, like polymorphic parameters, get upstreamed to mainline OCaml if there is enough community support.</p>
<p>The example that has been used both in the <a href="https://github.com/ocaml/ocaml/pull/13806">5.5 PR</a> and in the <a href="https://oxcaml.org/documentation/miscellaneous-extensions/polymorphic-parameters/">OxCaml documentation</a> is a good illustration. Let’s look at creation functions from <code>ppx_typed_fields</code> – given a record definition:</p>
<pre><code><span class="ocaml-keyword-other">type</span><span class="ocaml-source"> </span><span class="ocaml-source">t</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-source">{</span><span class="ocaml-source"> </span><span class="ocaml-source">a</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-support-type">string</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-source">b</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-support-type">int</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-source">}</span><span class="ocaml-source">
</span></code></pre>
<p><code>ppx_typed_fields</code> gives you a type representing the fields indexed by their type:</p>
<pre><code><span class="ocaml-keyword-other">type</span><span class="ocaml-source"> </span><span class="ocaml-storage-type">'a</span><span class="ocaml-source"> </span><span class="ocaml-source">field</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">A</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-support-type">string</span><span class="ocaml-source"> </span><span class="ocaml-source">field</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">B</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-support-type">int</span><span class="ocaml-source"> </span><span class="ocaml-source">field</span><span class="ocaml-source">
</span><span class="ocaml-source">  
</span></code></pre>
<p>A useful function to provide for these types is one that can create a <code>t</code> if given a function that returns a value for each of the fields. This could look like:</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">create</span><span class="ocaml-source"> </span><span class="ocaml-source">f</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-source">{</span><span class="ocaml-source"> </span><span class="ocaml-source">a</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">f</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">A</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-source">b</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">f</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">B</span><span class="ocaml-source"> </span><span class="ocaml-source">}</span><span class="ocaml-source">
</span></code></pre>
<p>But, <strong>without</strong> polymorphic parameters, the compiler would give the following error:</p>
<pre><code>Line 3, characters 21-22:
3 |     { a = f A; b = f B };;
                         ^
Error: This expression has type int field
       but an expression was expected of type string field
       Type int is not compatible with type string
</code></pre>
<p>What has happened is that you want to apply <code>f</code> to two types a <code>string field</code> and an <code>int field</code>. While the compiler has inferred a monomorphic type for f <code>int field</code>.  The function needs to be polymorphic!</p>
<p>From OCaml 5.5, you can annotate <code>f</code> as polymorphic:</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">create</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-source">f</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-storage-type">'a</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source"> </span><span class="ocaml-storage-type">'a</span><span class="ocaml-source"> </span><span class="ocaml-source">field</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-storage-type">'a</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-source">{</span><span class="ocaml-source"> </span><span class="ocaml-source">a</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">f</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">A</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-source">b</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">f</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">B</span><span class="ocaml-source"> </span><span class="ocaml-source">}</span><span class="ocaml-source">
</span></code></pre>
<p>Which will give <code>create</code> the following type:</p>
<pre><code><span class="ocaml-keyword-other">val</span><span class="ocaml-source"> </span><span class="ocaml-source">create</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-storage-type">'a</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source"> </span><span class="ocaml-storage-type">'a</span><span class="ocaml-source"> </span><span class="ocaml-source">field</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-storage-type">'a</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-source">t</span><span class="ocaml-source">
</span></code></pre>
<p>Which can be called on an appropriate function directly:</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">forty_two</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-keyword">type</span><span class="ocaml-entity-name-function-binding"> a</span><span class="ocaml-source">) </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-source">a</span><span class="ocaml-source"> </span><span class="ocaml-source">field</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-source">a</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">function</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">A</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-string-quoted-double">"</span><span class="ocaml-string-quoted-double">forty two</span><span class="ocaml-string-quoted-double">"</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">B</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">42</span><span class="ocaml-source">
</span><span class="ocaml-source">
</span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">r</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">create</span><span class="ocaml-source"> </span><span class="ocaml-source">forty_two</span><span class="ocaml-source">
</span></code></pre>
<p>Thus, polymorphic parameters let you use polymorphism with minimal code, without workarounds like defining fresh types or wrapping functions in types.</p>
<h2>GC Pacing</h2>
<p>Another one of the major updates is a series of changes that improve the Garbage Collector's performance, making it equivalent to OCaml 4.14! This was achieved over several PRs:</p>
<ul>
<li>
<p>Introduced a sweep-only phase at the start of the major GC cycle to reduce latent-garbage delay, or the time between a block becoming unreachable and it becoming available for allocation. This improves GC performance by recycling memory more quickly, often leading to reduced memory usage. This improvement was originally created for OxCaml before being upstreamed.  The PR is <a href="https://github.com/ocaml/ocaml/pull/13580">#13580</a> by Stephen Dolan and Nick Barnes with review by KC Sivaramakrishnan.</p>
</li>
<li>
<p>The <a href="https://github.com/ocaml/ocaml/pull/13616">sweeping patch</a> changed how the Garbage Collector tracked the blocks of free memory in the shared heap. By using a run-length encoding, the GC can efficiently skip large chunks of free memory. Consequently, the duration of the sweep is now proportional to how much of the heap is used, rather than to its entire size.</p>
<p>Some micro-benchmarks show substantial speedups for sparsely populated heaps. In larger applications, the improvement is likely to be smaller.</p>
</li>
<li>
<p>Added an idle phase to the GC between Sweep and Mark. In this phase, the GC does not do any work while the mutator is allocating as usual. This has improved performance on small heaps. The work is documented in PR <a href="https://github.com/ocaml/ocaml/pull/14365">#14365</a> by Damien Doligez with review by Stephen Dolan and Nick Barnes.</p>
</li>
</ul>
<h2>Relocatable OCaml</h2>
<p>Originally, the OCaml compiler required the Standard Library to be stored at a fixed location, which had been specified when the compiler was compiled. In other words, the install path was woven into the compiler binaries at build time.</p>
<p>Per the <a href="https://github.com/ocaml/ocaml/pull/14014">test harness PR</a>, the RFC defines ‘relocatable’ as when: the compiler binaries are identical regardless of the installation prefix or the working directory in which the compiler was built; the compiler binaries can be used from any disk location on the system without any further alteration to the binaries; and without further alteration to the user’s shell environment.</p>
<p>The most noticeable impact of this change for users is that creating a new <code>opam switch</code> is significantly faster. Instead of waiting for <code>opam</code> to build OCaml from sources, the switch can simply be cloned (‘relocated’). These speed-ups also benefit Dune package management.</p>
<p>Furthermore, it gives OCaml developers a way to reliably distribute pre-compiled binaries, knowing that if you clone and build OCaml in two separate locations, the resulting compiler binaries will be identical.</p>
<p>A relocatable compiler also has ecosystem-wide benefits. It’s good for <a href="https://discuss.ocaml.org/t/volunteers-to-review-the-relocatable-ocaml-work/16667">Windows support, reproducible builds, caching, and more</a>. It also opens up possibilities for new toolchains that distribute pre-built compilers via binary packages.</p>
<p>The Relocatable project is spread out over three main PRs:</p>
<ul>
<li>
<p>The first PR behind Relocatable is <a href="https://github.com/ocaml/ocaml/pull/14243">#14243</a>, which laid the groundwork for relocation by making <code>ld.conf</code> path relative rather than path absolute, while also tidying up several long-standing inconsistencies in how stub libraries were discovered at runtime.</p>
<p>Changes included adding support for explicit relative paths to <code>ld.conf</code>, which are interpreted relative to the directory the file was loaded from, and making relative paths the default; a new <code>--with-stublibs</code> configuration option; merging multiple <code>ld.conf</code> sources allowing the runtime to load and merge the file from all relevant locations rather than just the first one found; unifying the C and <code>ocamlc</code> implementations of the <code>ld.conf</code> parsing logic; and making <code>ld.conf</code> line-ending agnostic. The PR was created by David Allsopp and reviewed by Jonah Beckford, Damien Doligez and Hugo Heuzard.</p>
</li>
<li>
<p>Second, <a href="https://github.com/ocaml/ocaml/pull/14244">#14244</a> allowed for the absolute location of the Standard Library to be removed from both the C runtime and <code>compiler-libs</code> library.</p>
<p>It added the new <code>%standard_library_default</code> primitive which lets the compiler determine the value of <code>config.standard_library_default</code> each time a program is linked; a new configure option <code>--with-relative-libdir</code> that lets the compiler expect to find the Standard Library in a location relative to where the compiler is; new <code>--set-runtime-default</code> command line option to allow the <code>%standard_library_default</code> primitive tobe overridden when linking an executable; and using options for the C compiler to make the compiler’s artefacts more reproducible. This PR was created by David Allsopp, with review by Jonah Beckford, Antonin Décimo, Damien Doligez, Samuel Hym and Vincent Laviron.</p>
</li>
<li>
<p>Thirdly, <a href="https://github.com/ocaml/ocaml/pull/14245">#14245</a> enabled bytecode executables to find and verify the correct interpreter without hardcoded paths.</p>
<p>A new <code>-launch-method</code> command line option allowing for dynamic selection by <code>ocamlc</code> of either the shebang line or an executable-stub launcher for bytecode executables; new <code>-runtime-search</code> lets bytecode executables search for <code>ocamlrun</code> rather than executing it from a fixed location; a new name-mangling scheme for shared libraries and bytecode interpreter executables allowing multiple OCaml installations to safely coexist on <code>PATH</code> and <code>LD_LIBRARY_PATH</code>; and new configure options, <code>--enable-runtime-search</code> and <code>--enable-runtime-search-target</code>, which control whether the compiler's own bytecode binaries and the binaries it produces respectively can locate their runtime after relocation. The PR is by David Allsopp with review by Damien Doligez and Samuel Hym.</p>
</li>
</ul>
<h2>Standard Library Improvements</h2>
<p>OCaml’s standard library has received several improvements with the 5.5 release, particularly for the String module, which gains a wealth of new utility functions:</p>
<ul>
<li><a href="https://github.com/ocaml/ocaml/pull/14381">searching</a> (<code>find_first</code>, <code>find_last</code>, <code>find_all</code>, <code>includes</code>),</li>
<li><a href="https://github.com/ocaml/ocaml/pull/14437">splitting</a> (<code>split_first</code>, <code>split_last</code>, <code>split_all</code>),</li>
<li><a href="https://github.com/ocaml/ocaml/pull/14436">replacing</a> (<code>replace_first</code>, <code>replace_last</code>, <code>replace_all</code>),</li>
<li><a href="https://github.com/ocaml/ocaml/pull/14352">trimming</a> (<code>drop</code>, <code>take</code>, <code>cut</code> variants),</li>
<li>plus tweaks like <a href="https://github.com/ocaml/ocaml/pull/14438"><code>is_empty</code></a> and <a href="https://github.com/ocaml/ocaml/pull/14440"><code>of_char</code></a>.</li>
</ul>
<p>Beyond strings, there are new additions across many other modules:</p>
<ul>
<li><a href="https://github.com/ocaml/ocaml/pull/13343"><code>Array.stable_sort_sub</code></a> for sorting subarrays,</li>
<li><a href="https://github.com/ocaml/ocaml/pull/14185"><code>List.split_map</code></a> and <a href="https://github.com/ocaml/ocaml/pull/14227"><code>List.filter_mapi</code></a>,</li>
<li>new <a href="https://github.com/ocaml/ocaml/pull/13920"><code>Option</code> helpers</a> (<code>product</code>, <code>blend</code>, <code>for_all</code>, <code>exists</code>, <code>Syntax</code>),</li>
<li><a href="https://github.com/ocaml/ocaml/pull/14060"><code>Hashtbl.find_and_replace</code>/<code>find_and_remove</code></a>,</li>
<li><code>Set</code>/<code>Map</code> gaining <a href="https://github.com/ocaml/ocaml/pull/14118"><code>is_singleton</code></a>,</li>
<li>plus <code>Dynarray</code> getting <a href="https://github.com/ocaml/ocaml/pull/12877">reverse iteration</a>.</li>
</ul>
<p>For numerics, <code>Int</code>, <code>Int32</code>, <code>Int64</code>, and <code>Nativeint</code> all <a href="https://github.com/ocaml/ocaml/pull/14432">receive floor/ceil/Euclidean division</a> and a suite of bit-counting functions.</p>
<p>Concurrency sees improvements too, with <a href="https://github.com/ocaml/ocaml/pull/14086"><code>Domain.count</code></a>, better <a href="https://github.com/ocaml/ocaml/pull/14363">backtrace preservation</a> on domain exceptions, <a href="https://github.com/ocaml/ocaml/pull/14393"><code>Lazy.Mutexed</code></a> for thread-safe lazy values.</p>
<p>Rounding things out are <a href="https://github.com/ocaml/ocaml/pull/10177"><code>Seq.delay</code></a>, <a href="https://github.com/ocaml/ocaml/pull/13728"><code>Sys.runtime_executable</code></a>, <a href="https://github.com/ocaml/ocaml/pull/13101"><code>Fun.todo</code></a>, and new <a href="https://github.com/ocaml/ocaml/pull/13372">heterogeneous-list <code>printf</code></a> functions in <code>Format</code> and <code>Printf</code>.</p>
<p>As you can tell, there are a lot of PRs responsible for this effort, which invariably means there are a lot of contributors! We don't have space to mention them all, but they include Daniel Bünzli, François Pottier, Gabriel Scherer, KC Sivaramakrishnan, Leonardo Santos, Jeremy Yallop, David Allsopp, Nicolás Ojeda Bär, Kate Deplaix, Sacha-Élie Ayoun, Émile Trotignon, Basile Clément, Xavier Leroy, and Pavlo Khrystenko.</p>
<h2>… And Many More!</h2>
<p>There are, of course, many more improvements and changes coming in OCaml 5.5 than those described above. Let’s take a quick look at a few more, and as always, check out the <a href="https://github.com/ocaml/ocaml/blob/trunk/Changes">changelog</a> for an exhaustive list.</p>
<ul>
<li>The 5.5 update comes with a new type kind, <code>Type_external</code>, which can be used to distinguish external types from other types and from each other. You can now declare an abstract type with a name by using <code>type t = external “t”</code>. The change turns primitive types into external types, and also means that the old method of distinguishing abstract types as defined in the current module. The PR is <a href="https://github.com/ocaml/ocaml/pull/13712">#13712</a> by Takafumi Saikawa and Jacques Garrigue with review by Richard Eisenberg.</li>
<li>With a new way for C threads to register themselves to the OCaml runtime with <code>caml_c_thread_register_in_domain</code>, PR <a href="https://github.com/ocaml/ocaml/pull/14275">#14275</a> adds the domain’s unique ID as a parameter for where C threads can register. Before this change, all C threads would automatically be registered in domain 0. The PR is by Jack Nørskov Jørgensen with reviews from Gabriel Scherer and Guillaume Munch-Maccagnoni.</li>
<li>Replacing the <code>winpthreads</code> library with modern concurrency primitives using WinAPI for the MSVC and MinGW-w64 ports aims to reduce the maintenance burden and barrier to entry by making the code easier to inspect. PR <a href="https://github.com/ocaml/ocaml/pull/13416">#13416</a> removes <code>winpthreads</code> and implements <code>caml_plat_*</code> and <code>st_*</code> to abstract over <code>pthreads</code> and Windows concurrency APIs. It’s by Antonin Décimo with reviews by Samuel Hym, Gabriel Scherer, Miod Vallat,  B. Szilvasy, and Nicolás Ojeda Bär.</li>
<li>Introduced generational scanning of stack frames for ARM 64-bit, POWER, and RISC-V. Generational scanning means that when stack frames are scanned, scanning stops once it reaches a frame that has not changed since the previous minor GC. Generational scanning was not included in OCaml 5 from the start since OCaml 5 originally only ran on x86, an architecture that does not support this feature. With OCaml 5’s generous stack limits, generational scanning is a very useful feature that reduces minor GC work in the presence of those deep call stacks. The PRs involved are <a href="https://github.com/ocaml/ocaml/issues/13574">#13574</a> and <a href="https://github.com/ocaml/ocaml/pull/13594">#13594</a>, by KC Sivaramakrishnan and Xavier Leroy, with review by Miod Vallat, Gabriel Scherer, and Olivier Nicole; alongside multiple commits and discussions by community members elsewhere.</li>
<li>Another upstreamed feature from Jane Street’s production compiler, <a href="https://github.com/ocaml/ocaml/pull/14029">#14029</a> allowed <code>%identity</code> extensions to be treated as nonexpansive by the typechecker. By enabling the compiler to recognise that primitives like <code>Obj.magic</code> are not applications and therefore do not need to be subject to value restrictions. The original PR was opened by Stephen Dolan on the Jane Street compiler, and the upstreamed version was opened by Olivier Nicole. Reviews by Hugo Heuzard, Jacques Garrigue, Jeremy Yallop, and Gabriel Scherer.</li>
<li>Changed infix extension points and attributes, which meant that they attach to the AST node of the corresponding structure item when they appear in local modules/exceptions/opens rather than the enclosing let expression. This change makes the syntax more intuitive and is consistent with how they work for global structure items. PR <a href="https://github.com/ocaml/ocaml/pull/14009">#14009</a> by Nicolás Ojeda Bär with review by Gabriel Scherer.</li>
<li>This change extended what items are allowed in <code>let</code> expressions. For example, <code>let type t = … in …</code> is valid syntax after this update. The PR in question is <a href="https://github.com/ocaml/ocaml/pull/14040">#14040</a> opened by Nicolás Ojeda Bär with review by Valentin Gatien-Baron.</li>
</ul>
<h2>Bug Fixes</h2>
<p>Of course, we can’t wrap things up without looking at a few bug fixes!</p>
<ul>
<li>
<p><a href="https://github.com/ocaml/ocaml/pull/14210">#14210</a>: fixed a data race, identified by TSan,  in the weak pointers runtime. This effort was by Gabriel Scherer and Damien Doligez, report by Olivier Nicole, and review by KC Sivaramakrishnan. Alongside #14210, several other TSan-related bug fixes were included with this release, including <a href="https://github.com/ocaml/ocaml/pull/14213">#14213</a>, <a href="https://github.com/ocaml/ocaml/pull/14255">#14255</a>, and <a href="https://github.com/ocaml/ocaml/pull/14332">#14332</a>.</p>
</li>
<li>
<p><a href="https://github.com/ocaml/ocaml/pull/13777">#13777</a>, <a href="https://github.com/ocaml/ocaml/pull/14347">#14347</a>, <a href="https://github.com/ocaml/ocaml/issues/14348">#14348</a>, <a href="https://github.com/ocaml/ocaml/pull/14421">#14421</a>, <a href="https://github.com/ocaml/ocaml/pull/14498">#14498</a>, <a href="https://github.com/ocaml/ocaml/pull/14526">#14526</a>: A series of PRs that tested that C++ compilers can link with the OCaml runtime and include its headers. Work by Antonin Décimo with reviews by David Allsopp and Gabriel Scherer.</p>
</li>
<li>
<p><a href="https://github.com/ocaml/ocaml/pull/14495">#14495</a>: Fixed an infix-tag bug in the minor collector that could result in SEGVs in multi-domain programs. The fix was by Nick Barnes with review by Gabriel Scherer.</p>
</li>
<li>
<p><a href="https://github.com/ocaml/ocaml/pull/14519">#14519</a>: Fixed a segfault that would occur when using <code>Runtime_events</code> with a certain set-up, due to bad error checking when calling <code>mmap()</code>. Fix by Mark Elvers with review by Nicolás Ojeda Bär.</p>
</li>
<li>
<p><a href="https://github.com/ocaml/ocaml/issues/14644">#14644</a>, <a href="https://github.com/ocaml/ocaml/pull/14647">#14647</a>: Fixed a segfault in bytecode that would happen when the <code>caml_make_unhandled_effect_exn</code> exception was raised.  PRs by Vincent Laviron and report by Thibaut Mattio, with reviews by Nicolás Ojeda Bär, Stephen Dolan and Olivier Nicole.</p>
</li>
<li>
<p><a href="https://github.com/ocaml/ocaml/issues/14349">#14349</a>, <a href="https://github.com/ocaml/ocaml/pull/14718">#14718</a>, <a href="https://github.com/ocaml/ocaml/pull/14722">#14722</a>: Another runtime fix, which addressed orphaned ephemerons in several steps. PRs by Gabriel Scherer, with reviews by Olivier Nicole and Damien Doligez, and a report by Jan Midtgaard.</p>
</li>
</ul>
<h2>Stay In Touch!</h2>
<p>Tried out the new update? Connect with us on <a href="https://bsky.app/profile/tarides.com">Bluesky</a>, <a href="https://mastodon.social/@tarides">Mastodon</a>, <a href="https://www.threads.net/@taridesltd">Threads</a>, and <a href="https://www.linkedin.com/company/tarides">LinkedIn</a> or sign up for our mailing list to stay updated on our latest projects. You can also connect with other OCaml users on <a href="https://discuss.ocaml.org/">Discuss</a>. We look forward to hearing your thoughts!</p>

