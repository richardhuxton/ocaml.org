---
title: Progress in OCaml docs
description:
url: https://jon.recoil.org/blog/2025/05/docs-progress.html
date: 2025-05-29T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---

<h1><a href="https://jon.recoil.org/atom.xml#progress-in-ocaml-docs" class="anchor"></a>Progress in OCaml docs</h1>
<ul class="at-tags"><li class="published"><span class="at-tag">published</span> <p>2025-05-29</p></li></ul>
<ul class="at-tags"><li class="page-tags"><span class="at-tag">page-tags</span> <p><a href="https://jon.recoil.org/tags/odoc.html" title="odoc">odoc</a> <a href="https://jon.recoil.org/tags/docs-ci.html" title="docs-ci">docs-ci</a></p></li></ul>
<p>The docs build is progress well, and we've <i>just about</i> hit 20,000 packages (20,038 to be precise). So at this point I thought it'd be useful to take a look through the various failures to see if there are any insights to be gained.</p>
<p>Odoc requires a built package in order to generate the docs, there are two steps that have to be done before we can begin building the docs. Step one is to figure out the exact set of packages to build - ie, doing an opam solve, and step two is to actually build the packages. These two steps are, to some extent, out of docs-ci's control, and rely on the state of opam repository. While there are efforts to keep this in as good a state as possible, it's still the case that these steps fail much more often than the actual docs build itself. Let's take a look at some of the failures we see in each of these steps.</p>
<h2><a href="https://jon.recoil.org/atom.xml#step-1:-opam-solve" class="anchor"></a>Step 1: opam solve</h2>
<p>There are 2,074 solver failures. A good chunk of these are due to the way docs ci works itself, that it starts with a specific version of OCaml. In order to do this, the solution must have a specific version of OCaml in it, and this is not always the case, for example, all of the <code>conf-*</code> packages fail in this way. This particular class of "failures" is not at all important, as mostly they don't contain useful documentation, but even if they do, if they're actually being used then they will be built as part of another solution. For example, while <code>conf-faad</code> fails with this error, the solution of the <code>faad</code> package pulls it in anyway, so we can build any docs that it includes. Roughly two thirds (685) of the reported failures are due to this, and by checking the resulting HTML docs we can see that we do get docs for 278 of these, so they must be pulled in by other solutions.</p>
<p>The remaining failures are "real" in the sense that we never currently get docs for these packages. In turn, these can be subcategorised. One class of failures happen with platform-specific packages, for example <code>camlkit</code> which provides bindings to Cocoa frameworks, and is only available on macOS, or <code>eio_windows</code> which clearly requires Windows. The current docs-ci setup only builds on Linux, and extending this to other platforms will require a little more work, and is not currently scheduled. These are "fixable" failures.</p>
<p>The third class of failures are those that will "just never work". For example, some early versions of <code>domainslib</code> were released before the OCaml 5.0 APIs were finalised, and so they can only work with alpha versions of OCaml 5.0. We won't be documenting these.</p>
<p>Finally there are some more 'unexplained' failures, such as <code>docteur.0.0.2</code>. This one's particularly interesting as the solve actually succeeds when using the stand-alone tool opam-0install, whereas it's failing in docs-ci, which uses opam-0install as a library! I'm currently suspicious about the 'deprecated' flag, as the failure log contains:</p>
<pre>- git-cohttp-unix -&gt; (problem)
    No usable implementations:
      git-cohttp-unix.3.6.0: Availability condition not satisfied
      git-cohttp-unix.3.5.0: Availability condition not satisfied
      git-cohttp-unix.3.4.0: Availability condition not satisfied
      git-cohttp-unix.3.3.3: Availability condition not satisfied
      git-cohttp-unix.3.3.2: Availability condition not satisfied
      ...</pre>
<p>and that flag is the only thing I can immediately see that stands out in <code>git-cohttp-unix</code>. In contrast, the solution given by opam-0install contains <code>git-cohttp-unix.3.6.0</code> as a dependency. I suspect fixing this will cause quite a few more packages to succeed.</p>
<h2><a href="https://jon.recoil.org/atom.xml#step-2:-building-packages" class="anchor"></a>Step 2: building packages</h2>
<p>The next step, once we've got the solutions, is to build the packages. This is using the new method I <a href="https://jon.recoil.org/04/ocaml-docs-ci-and-odoc-3.html" title="ocaml-docs-ci-and-odoc-3">previously wrote about</a>. There are about 1,000 packages that fail to build, and once again we can take a look and categorise some of these failures. There are a wider variety of failures here, and it's quite useful to cross-check with <a href="https://check.ci.ocaml.org/">opam health check</a> to see if it's known to be broken. Unfortunately OHC only builds the latest versions of everything, so we can't check in some cases. The interesting issues are where we're failing to build something that seems to work in OHC.</p>
<h3><a href="https://jon.recoil.org/atom.xml#llvm.18" class="anchor"></a>llvm.18</h3>
<p>This is an interesting type of error, where the build fails because of a missing external dependency. The <code>llvm</code> package depends upon <code>conf-llvm-static.18</code>, which should be able to install the depext. Looking at the package, it does indeed have a depext for Debian:</p>
<pre>depexts: [
  ["llvm@18" "zstd"] {os-distribution = "homebrew" &amp; os = "macos"}
  ["llvm-18"] {os-distribution = "macports" &amp; os = "macos"}
  ["llvm-18-dev" "zlib1g-dev" "libzstd-dev"] {os-family = "debian"}
  ["llvm18-dev"] {os-distribution = "alpine"}
  ["llvm18"] {os-family = "arch"}
  ["llvm18-devel"] {os-family = "suse" | os-family = "opensuse"}
  ["llvm18-devel"] {os-distribution = "fedora" &amp; os-version &gt;= "41"}
  ["llvm-devel"] {os-distribution = "fedora" &amp; os-version = "40"}
  ["llvm18-devel" "epel-release"] {os-distribution = "centos"}
  ["devel/llvm18"] {os = "freebsd"}
]</pre>
<p>However, in Debian 12, they've already updated to <code>llvm-19</code>, so the depext is not available.</p>
<h3><a href="https://jon.recoil.org/atom.xml#camlimages.5.0.5" class="anchor"></a>camlimages.5.0.5</h3>
<p>This one fails due to a linking error. Oddly enough it does seem to work in OHC.</p>
<pre>(cd _build/default &amp;&amp; /home/opam/.opam/4.14/bin/ocamlmklib.opt -g -o freetype/camlimages_freetype_stubs freetype/ftintf.o -ldopt -lfreetype)
# /usr/bin/ld: freetype/ftintf.o: warning: relocation against `Caml_state' in read-only section `.text'
# /usr/bin/ld: freetype/ftintf.o: relocation R_X86_64_PC32 against undefined symbol `Caml_state' can not be used when making a shared object; recompile with -fPIC
# /usr/bin/ld: final link failed: bad value</pre>
<h3><a href="https://jon.recoil.org/atom.xml#ahrocksdb.0.2.2" class="anchor"></a>ahrocksdb.0.2.2</h3>
<p>This one fails in OHC too, but it looks like it's a build failure with more recent gccs, fixed upstream: https://github.com/ahrefs/ocaml-ahrocksdb/commit/e52316b3d30fededac023141bf8b47da79cabfed</p>
<pre># run: gcc -O2 -fno-strict-aliasing -fwrapv -fPIC -pthread  -I/usr/include/rocksdb -I /home/opam/.opam/5.3/lib/ocaml -o /tmp/build_02b340_dune/ocaml-configuratordc5e55/c-test-2/test.exe /tmp/build_02b340_dune/ocaml-configuratordc5e55/c-test-2/test.c -lm -lpthread -lrocksdb
# -&gt; process exited with code 1
# -&gt; stdout:
# -&gt; stderr:
#  | In file included from /tmp/build_02b340_dune/ocaml-configuratordc5e55/c-test-2/test.c:4:
#  | /usr/include/rocksdb/version.h:7:10: fatal error: string: No such file or directory
#  |     7 | #include &lt;string&gt;
#  |       |          ^~~~~~~~
#  | compilation terminated.
# Error: discover error</pre>
<h3><a href="https://jon.recoil.org/atom.xml#alt-ergo.2.2.0" class="anchor"></a>alt-ergo.2.2.0</h3>
<p>Looks like it's trying to write outside the sandbox. The failure only occurs on alt-ergo 1.3.0 - 2.2.0.</p>
<pre># mkdir -p /home/opam/.opam/4.14/man/man1
# cp -f doc/alt-ergo.1 /home/opam/.opam/4.14/man/man1
# mkdir -p /usr/local/lib/alt-ergo/preludes
# mkdir: cannot create directory '/usr/local/lib/alt-ergo': Permission denied
# make: *** [Makefile.users:243: install-preludes] Error 1</pre>
<h3><a href="https://jon.recoil.org/atom.xml#ctypes-foreign.0.18.0" class="anchor"></a>ctypes-foreign.0.18.0</h3>
<p>This one is a much more interesting failure. The logs show:</p>
<pre>[ERROR] No solution for ctypes-foreign.0.18.0:   * Missing dependency:
            - ctypes-foreign -&gt; ctypes
            unknown package</pre>
<p>which is happening because of the optimisation I <a href="https://jon.recoil.org/04/ocaml-docs-ci-and-odoc-3.html" title="ocaml-docs-ci-and-odoc-3">mentioned before</a> where we build a new <code>opam-repository</code> with only the packages we're going to need. In this case, we've somehow missed out the <code>ctypes</code> package. Looking at the opam file for <code>ctypes-foreign</code>, it has a <code>post</code> dependency on <code>ctypes</code>. The <code>post</code> keyword indicates that <code>ctypes</code> should be installed with <code>ctypes-foreign</code>, but that having it as a "normal" dependency would introduce a dependency cycle. Since we require a DAG of dependencies, we explicitly remove any <code>post</code> dependencies from the set of packages to build, but it seems that <code>opam</code> would like to know about it anyway!</p>
<h3><a href="https://jon.recoil.org/atom.xml#others" class="anchor"></a>others</h3>
<p>There are many more. An automatic cross-check with OHC would be really useful, mainly to distinguish between the packages that are broken due to <code>ocaml-docs-ci</code> issues (like <code>ctypes-foreign</code>) and those that are broken for other reasons (like <code>ahrocksdb</code>).</p>
<h2><a href="https://jon.recoil.org/atom.xml#step-3:-building-docs" class="anchor"></a>Step 3: building docs</h2>
<p>Finally, we have the actual docs build. This is where we run <code>odoc</code> and <code>odoc_driver</code> to produce the HTML docs. All the errors here are ones that we should be able to fix!</p>
<p>Firstly, there are the internal errors:</p>
<pre>Uncaught exception: Failure("\"rm\" \"-rf\" \"/var/cache/obuilder/merged/582e973685d380d4c91eadc2611eee02c82c5fe4f8bd732e0080fb22bc4404cd\" \"/var/cache/obuilder/work/582e973685d380d4c91eadc2611eee02c82c5fe4f8bd732e0080fb22bc4404cd\" failed with exit status 1")
2025-05-22 09:30.18: Job failed: Failed: Internal error</pre>
<p>These are some <code>obuilder</code> error that needs fixing. Currently we're just rerunning the job to fix these.</p>
<h3><a href="https://jon.recoil.org/atom.xml#odoc.2.0.0" class="anchor"></a>odoc.2.0.0</h3>
<p>Oops, we can't build our own docs! At least it's an old version :-)</p>
<pre>odoc: internal error, uncaught exception:
      File "src/html/link.ml", line 101, characters 16-22: Assertion failed
      Raised at Odoc_html__Link.href in file "src/html/link.ml", line 101, characters 16-57
      Called from Odoc_html__Generator.internallink in file "src/html/generator.ml", line 108, characters 19-49
...</pre>
<p>The failure points <a href="https://github.com/ocaml/odoc/blob/42190737339d9be4510eeeb0e3c47e84badf4d73/src/html/link.ml#L101">here</a>, an assertion about the common ancestor of two paths. <a href="https://github.com/ocaml/odoc/issues/1345">Issue filed</a>.</p>
<h3><a href="https://jon.recoil.org/atom.xml#ocaml-base-compiler.4.07.0" class="anchor"></a>ocaml-base-compiler.4.07.0</h3>
<p>This one happens because of our "optimisation" to use a base image with OCaml pre-installed. What we <i>actually</i> do is find the major/minor version of OCaml and use the corresponding docker image - so in this case we'll use ocaml/opam:debian-12-ocaml-4.07. Now this image actually contains OCaml 4.07.1, and the format of <code>cmt</code> and <code>cmti</code> files changed between these releases, so we get a failure.</p>
<p>We'll fix this by getting rid of the optimisation and building from an empty switch.</p>
<h3><a href="https://jon.recoil.org/atom.xml#lascar.0.7.0" class="anchor"></a>lascar.0.7.0</h3>
<p>This one is quite interesting. It's another assertion failure in odoc:</p>
<pre>odoc: internal error, uncaught exception:
      File "src/xref2/cpath.ml", line 364, characters 37-43: Assertion failed
      Raised at Odoc_xref2__Cpath.unresolve_resolved_parent_path in file "src/xref2/cpath.ml", line 364, characters 37-49
      Called from Odoc_xref2__Cpath.unresolve_module_path in file "src/xref2/cpath.ml", line 349, characters 28-60
      Called from Odoc_xref2__Tools.fragmap.map_module_decl in file "src/xref2/tools.ml", line 1792, characters 48-80</pre>
<p>It's happening when we 'unresolve' a previously resolved path. We end up having to do this when something about the path has changed, in this case while we're handling a <code>S with module Foo = Bar</code> or similar. Issue <a href="https://github.com/ocaml/odoc/issues/1346">filed</a>.</p>
<h3><a href="https://jon.recoil.org/atom.xml#camlp5" class="anchor"></a>camlp5</h3>
<p>This one actually occurs in <code>odoc_driver</code> rather than in <code>odoc</code> itself.</p>
<pre>odoc_driver_voodoo: [DEBUG] Found cmi_only_lib in dir: /home/opam/.opam/4.08/lib/camlp5
odoc_driver_voodoo: internal error, uncaught exception:
                    Invalid_argument("\"/home/opam/.opam/4.08/lib/camlp5\": invalid segment")
                    </pre>
<p>Here we're trying to add a segment to a path, but rather than a single path segment we've got an entire fully qualified path. Issue <a href="https://github.com/ocaml/odoc/issues/1347">filed</a>.</p>
<h2><a href="https://jon.recoil.org/atom.xml#conclusion" class="anchor"></a>Conclusion</h2>
<p>It's pretty good that we've only got 4 types of error happening at the doc-generation phase. However, as a whole, any error that occurs earlier in the pipeline ends up with a missing documentation tab on the website, and we need to do a bit more so that the actual problem can be tracked down and fixed. This is obviously a more general problem than just the docs, and one that <a href="https://check.ci.ocaml.org">opam health check</a> seeks to highlight. However, the current incarnation of OHC is significantly less efficient than docs-ci, so generalising the approach we've taken with <a href="https://github.com/jonludlam/opamh">opamh</a> should really help with making this more responsive.</p>
<p>In addition, a number of the issues seen here could be addressed with a tool my colleague <a href="https://ryan.freumh.org/">Ryan</a> is working on: <a href="https://ryan.freumh.org/enki.html">Enki</a>. This tool would allow us to run a solve that actually determines not only the set of packages we wish to install, but the platform to install onto - e.g. for <code>eio_windows</code> the solution would be to install on Windows, and for <code>llvm.18-static</code> the solution might be Fedora 40.</p>
