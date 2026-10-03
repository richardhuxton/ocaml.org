---
title: Using Forester to turn Foundations of CS into interactive evergreen lectures
description: Porting the FoCS lecture notes to Forester, including transclusions and
  stable section URLs, and a live OCaml toplevel compiled into the browser.
url: https://anil.recoil.org/notes/forester-teaching-notes
date: 2026-08-16T00:00:00-00:00
preview_image: https://anil.recoil.org/images/focs-forest-ss-1.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>I've heard good feedback from my undergrad students about how <a href="https://jonmsterling.com">Jon Sterling</a> publishes his <a href="https://www.jonmsterling.com/00JB/">1A Discrete Maths lectures</a> in a structured online form using his <a href="https://www.forester-notes.org/bafkrmidpuo45tjd55ndfgg2ipcwgfaxby7paf6em5nw7tbfgo377gfm3re.pdf">own Forester tool</a>. As my sabbatical comes to an end, I've started prepping my own <a href="https://anil.recoil.org/notes/focs">Foundations of CS</a> course<sup><a href="https://anil.recoil.org/news.xml#fn:1" class="footnote">[1]</a></sup> for resuming lecturing in October. I ported this course to Jupyter a few years ago along with a nice <a href="https://www.cl.cam.ac.uk/teaching/2425/FoundsCS/focs-202425-v1.6.pdf">printable set</a> of notes with the same content.</p>
<p>Over the last week, I've experimented with porting the Markdown FoCS lecture sources over to use <a href="https://www.forester-notes.org/index/index.xml">Forester</a> instead. I like Jon's perspective on <a href="https://www.forester-notes.org/tfmt-000V/index.xml">evergreen notes</a>, and also the idea of interlinking concepts <em>across</em> lecture courses eventually.</p>
<p>This is now online as a <strong><a href="https://www.cl.cam.ac.uk/~avsm2/fcs/index/index.xml">draft interactive forest of the 2026–27 notes</a></strong>. First, I'll jot down notes on <a href="https://anil.recoil.org/news.xml#how-are-forests-structured">how forests are structured</a>, then on <a href="https://anil.recoil.org/news.xml#making-the-forest-a-live-program">making the forest into a live program</a> by embedding the OCaml compiler into the browser, and finally on <a href="https://anil.recoil.org/news.xml#publishing-the-forest-to-the-web">publishing it all</a> as a static site.</p>
<p><a href="https://www.cl.cam.ac.uk/~avsm2/fcs/index/index.xml"> <img src="https://anil.recoil.org/images/focs-forest-ss-1.webp" alt="%c"> </a></p>
<h2><a href="https://anil.recoil.org/news.xml#how-are-forests-structured" class="anchor" aria-hidden="true"></a>How are forests structured?</h2>
<p>There's a really good <a href="https://www.forester-notes.org/bafkrmidpuo45tjd55ndfgg2ipcwgfaxby7paf6em5nw7tbfgo377gfm3re.pdf">overview talk</a>
from a couple of years ago to get you started. The basic idea behind Forester
is to structure notes as a series of
<a href="https://www.forester-notes.org/007L/index.xml">transclusions</a>.
A document isn't written top-to-bottom but instead assembled from
trees that include other trees by reference.</p>
<p>The syntax to do this is a LaTeX-like <code>\transclude{addr}</code> that splices the
tree at <code>addr</code> into the local note as a section. Headings, numbering and depth are computed
from where the transclusion appears and not where it is written down in the filesystem.</p>
<p>The Forester format is a bit of a departure from Markdown, which
<a href="https://jacobzelko.com/05172024064639-forest-zettelkasten/">others</a> have noted.
<a href="https://patrick.sirref.org">Patrick Ferris</a> maintains a frontend to Forester called
<a href="https://graft.sirref.org/graft-0004/index.xml">Graft</a>, which converts Markdown into the native Forester
format.  I decided to go fully in and commit to the Forester syntax while I find my way around!</p>
<p>My original lecture notes use a traditional linear Markdown format, e.g.:</p>
<pre><code class="language-markdown"># Lecture 3: Lists

## Append: List Concatenation
...etc
</code></pre>
<p>The section numbering you see in the <a href="https://www.cl.cam.ac.uk/teaching/2425/FoundsCS/focs-202425-v1.6.pdf">PDF version</a> (where this is "3.5") was computed later in the LaTeX build, so the Markdown sources have no stable way to refer to a section beyond direct links to the section name.
In the new FoCS forest, every section now has its own <code>tree</code> file. A lecture tree is just
the title plus a list of transclusions; e.g. in <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-lists/"><code>focs-lists.tree</code></a> we have:</p>
<pre><code>\title{Lists}
\taxon{Lecture}
\author{anil-madhavapeddy}

\transclude{focs-list-primitives}
\transclude{focs-head-tail}
\transclude{focs-append}
...
</code></pre>
<p>A leaf with some content like <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-append/"><code>focs-append.tree</code></a> is straightforward:</p>
<pre><code>\title{Append: List Concatenation}
\author{anil-madhavapeddy}

\pre{\startverb
# let rec append xs ys =
    match xs, ys with
    | [], ys    -&gt; ys
    | x::xs, ys -&gt; x :: append xs ys
val append : 'a list -&gt; 'a list -&gt; 'a list = &lt;fun&gt;
\stopverb}

\p{Patterns can be as complicated as we like.  Here, the two patterns
are \code{[], ys} and \code{x::xs, ys}.}
</code></pre>
<p>This means that the same content now renders in two separate contexts.
First, it's section 3.5 inside the <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-lists/">Lists lecture</a>, and also a standalone page
with a <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-append/">stable URL</a> that can be linked, tagged, queried and (eventually)
evaluated.</p>
<p>On the standalone page, Forester automatically adds a "Context" backmatter section
embedding the parent lecture, so a reader always knows where a fragment was transcluded from.
As we'll see later, the live-code machinery uses this feature to also build up a live programming environment even for isolated fragments.</p>
<p>For small trees not worth their own file, we can also use <code>\subtree[addr]{…}</code> to declare an
addressable tree inline. I use this for per-lecture exercises, e.g. in <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-ex-3/"><code>focs-ex-3.tree</code></a>:</p>
<pre><code>\title{Exercises}
\put\transclude/toc{false}

\subtree[focs-ex-3-1]{
\title{Summing a list}
\taxon{Exercise}
\tag{lists}
\tag{recursion}

\p{Code a [recursive function](focs-def-recursion) to compute the sum of
a list's elements. ...}
}
</code></pre>
<p>Despite being defined inline, <code>focs-ex-3-1</code> remains a full Forester citizen and
has <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-ex-3-1/">its own URL</a> and shows up in the "Summing a list" backlinks of the
<a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-def-recursion/">definition trees</a> it links to. Forester in general feels like it has a good
internally consistent model that all this syntax maps to.</p>
<h3><a href="https://anil.recoil.org/news.xml#transclusions-allow-forester-to-number-sensibly" class="anchor" aria-hidden="true"></a>Transclusions allow Forester to number sensibly</h3>
<p>Notice the titles above carry no numeric titles like "3.5". Just like in LaTeX,
Forester numbers transclusions contextually, so the same tree renders as (e.g.)
"3.5" inside its lecture and unnumbered on its own page.</p>
<p>I needed an exception for the exercises, as numbering each individually was too
noisy. This was easy enough; I gave them each a title and then just marked them
as requiring a collapsed rendering by default so the ToC can focus on the
content and not exercises.</p>
<pre><code>\put\transclude/toc{false}                % keep out of the table of contents
\scope{\put\transclude/expanded{false}
  \transclude{focs-ex-3}}                 % render collapsed
</code></pre>
<h3><a href="https://anil.recoil.org/news.xml#adding-tags-and-metadata-to-forests" class="anchor" aria-hidden="true"></a>Adding tags and metadata to forests</h3>
<p>My original Markdown lectures had no metadata at all, which made converting them
to a richer format difficult. With Forester, every section and exercise tree now
carries a few tags that describes that section a bit better. I'm going to match
these to our syllabus tags (which are also used in examination rubrics) once the
editing settles down.</p>
<pre><code>\title{Append: List Concatenation}
\author{anil-madhavapeddy}
\tag{lists}
\tag{recursion}
\tag{complexity}
</code></pre>
<h3><a href="https://anil.recoil.org/news.xml#cross-references-refer-to-stable-ids" class="anchor" aria-hidden="true"></a>Cross references refer to stable IDs</h3>
<p>One very cool aspect of Forester is that inline cross-references aren't
too distracting from the main prose. We can do HTML-style wrapping very
easily.</p>
<p>For example, the Markdown version couldn't deep link easily due to not knowing
what the output format was:</p>
<pre><code class="language-markdown">Write a version of function `power` (Lecture 1) using `while`
instead of recursion.
</code></pre>
<p>In <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-ex-11-2/">the Forester version</a>, this uses the stable id:</p>
<pre><code>\p{Write a version of function \code{power} (\ref{focs-intro}) using
\code{while} instead of [recursion](focs-def-recursion).}
</code></pre>
<p>The Forester renderer reads the target's <code>\taxon</code> and contextual number and stays correct even if lectures are reordered.</p>
<h3><a href="https://anil.recoil.org/news.xml#making-definition-trees-and-glossaries" class="anchor" aria-hidden="true"></a>Making definition trees and glossaries</h3>
<p>We can then do further tagging to add useful index pages for core concepts:</p>
<pre><code>\title{Tail recursion}
\taxon{Definition}
\tag{recursion}

\p{A recursive function whose computation does not nest is called
\em{iterative} or \em{tail-recursive}: ...}
</code></pre>
<p>All of the mentions across the course then link to <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-def-tail-recursion/">this definition</a>, and Forester's automatic backlinks give us the reverse index for free.
The <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-glossary/">glossary page</a> uses a fancy <a href="https://www.forester-notes.org/013E/index.xml">datalog query engine</a> that appeared in Forester 5.0:</p>
<pre><code>\title{Glossary}
\query{\datalog{?x -: {\rel/has-taxon ?x '{Definition}}}}
</code></pre>
<p>The same query method also builds a "<a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-all-exercises/">collected exercises</a>" page (<code>has-taxon Exercise</code>) and
a <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-topics/">topic index</a> (one <code>has-tag</code> query per tag). These pages update themselves as trees are added just as you'd expect with any other database driven query.</p>
<h2><a href="https://anil.recoil.org/news.xml#making-the-forest-a-live-program" class="anchor" aria-hidden="true"></a>Making the Forest a live program</h2>
<p>The Markdown's code blocks are <a href="https://github.com/realworldocaml/mdx/pull/124">mdx-checked toplevel transcripts</a>, which means that the output of some OCaml code is actually compiled and verified. This was formerly done on the server side, but we can compile OCaml into the browser very easily to embed the compiler into the output Forest!</p>
<p><img src="https://anil.recoil.org/images/focs-forest-ss-2.webp" alt="%c" title="There is a live OCaml toplevel now throughout the Forest output"></p>
<p>First, the syntax is a little different and more LaTeX-like.  The original mdx Markdown was:</p>
<pre><code class="language-markdown">```ocaml
# let x = [3; 5; 9]
val x : int list = [3; 5; 9]
```
</code></pre>
<p>and the corresponding Forester looks like this:</p>
<pre><code>\pre{\startverb
# let x = [3; 5; 9]
val x : int list = [3; 5; 9]
\stopverb}
</code></pre>
<p>Forester outputs XSL which is rendered into HTML via browser stylesheets.
In order to make this work with Js_of_ocaml, we first need a basic toplevel.</p>
<p>This is very straightforward, with a dune file that has <code>linkall</code> specified
to stop unused modules being dropped, since we need everything in the Stdlib
available in an interactive toplevel.  However, it occurred to me that it might
actually be useful to drop OCaml Stdlib modules as well for the purposes of FoCS,
since we build up the standard library for ourselves through the course. Something
for the next iteration!</p>
<pre><code>(executable
 (name focs_toplevel)
 (modes byte)
 (link_flags (-linkall))
 (libraries js_of_ocaml js_of_ocaml-toplevel))

(rule
 (targets focs-toplevel.js)
 (action (run %{bin:js_of_ocaml} --toplevel %{dep:focs_toplevel.bc} -o %{targets})))
</code></pre>
<p>The toplevel itself is extremely straightforward, as we just need to register a
JavaScript callback and ensure we also preserve the compiler error messages
for the toplevel:</p>
<pre><code class="language-ocaml">let execute code =
  let buf = Buffer.create 256 in
  let fmt = Format.formatter_of_buffer buf in
  Sys_js.set_channel_flusher stdout (Buffer.add_string buf);
  Sys_js.set_channel_flusher stderr (Buffer.add_string buf);
  JsooTop.execute true fmt code;
  Buffer.contents buf

let () =
  JsooTop.initialize ();
  Js.Unsafe.set Js.Unsafe.global (Js.string "focsExecute")
    (Js.wrap_callback (fun s -&gt; Js.string (execute (Js.to_string s))))
</code></pre>
<p>Then, running <code>js_of_ocaml --toplevel</code> embeds the stdlib's cmi files so the typechecker
works in the browser. It's not lightweight, about ~9 MB raw and ~2 MB gzipped but it's lazily loaded the first time a toplevel is clicked.</p>
<p>As a quick hack (aka '<a href="https://en.wikipedia.org/wiki/Hydration_(web_development)">hydration</a>' in JavaScript parlance), there's a script that scans for <code>&lt;pre&gt;</code> lines that start with a hash and marks those as editable. This involves rewriting that HTML with a component that
has an editable code area, a Run button, and an output pane pre-filled
with the OCaml's expected output. Although an anachronism these days, this
technique also lets browsers with JS disabled still work reasonably.</p>
<h3><a href="https://anil.recoil.org/news.xml#allow-snippets-to-work-as-well" class="anchor" aria-hidden="true"></a>Allow snippets to work as well</h3>
<p>This all works with notebook-like semantics when viewed from the main page.
Clicking on any item runs a block and first replays any previous codeblocks
to fill its environment with relevant type and function definitions.</p>
<p>However, Forester supports transclusions, which means that we might not be
viewing the page as one giant list of sections! And indeed, clicking through
to one of the subpages showed that the OCaml toplevels broke as they couldn't
find their old function definitions.</p>
<p>To get around this, we can declare metadata tags so that the Forest tree declares its
dependencies in the forest source; e.g. the <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-bst-lookup/">tree lookup section</a> needs
the earlier <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/focs-binary-trees/">binary tree definitions</a> in its toplevel environment:</p>
<pre><code>\title{Lookup: Seeks Left or Right}
\tag{dictionaries}
\meta{ocaml-deps}{focs-binary-trees}
</code></pre>
<p>Forester carries these <code>\meta</code> tags into the output page XML as
<code>&lt;fr:meta name="ocaml-deps"&gt;focs-binary-trees&lt;/fr:meta&gt;</code>, so the JavaScript
can look through its own source and recursively resolve
each dependency's transcripts for the toplevel.</p>
<p>This did require me to annotate trees with <code>ocaml-deps</code> incrementally,
but this is good hygiene anyway and could be automated if I used a build
system in the future for the OCaml snippets.</p>
<h3><a href="https://anil.recoil.org/news.xml#good-web-standards-from-forester" class="anchor" aria-hidden="true"></a>Good web standards from Forester</h3>
<p>One interesting thing about Forester 5.0 is that it emits XML rendered by the browser via XSLT, and not HTML directly.
This is very elegant, and it's just as easy to add the output <code>js_of_ocaml</code>.
There's a theme directory for the forest, so the integration is a couple of
lines in the <code>tree.xsl</code> that does the transform:</p>
<pre><code class="language-xml">&lt;script type="module" src="{/f:tree/@base-url}forester.js"&gt;&lt;/script&gt;
&lt;script type="module" src="{/f:tree/@base-url}ocaml-live.js"&gt;&lt;/script&gt;  &lt;!-- added --&gt;
</code></pre>
<p>The last time I <a href="https://lists.horde.org/archives/doc/Week-of-Mon-20010205/000202.html">hacked on DocBook XSLT</a> was
back in 2001, so this was a bit of a blast from the past...</p>
<p>Fortunately, when Chrome finally finishes its strongarming of web standards and <a href="https://developer.chrome.com/docs/web-platform/deprecating-xslt">removes XSLT</a> later this year, the same stylesheet can still be compiled into HTML at build time via <code>xsltproc</code>.</p>
<h2><a href="https://anil.recoil.org/news.xml#publishing-the-forest-to-the-web" class="anchor" aria-hidden="true"></a>Publishing the Forest to the web</h2>
<p>All the Forester output is a static website, so I just uploaded it to <a href="https://www.cl.cam.ac.uk/~avsm2/fcs/index/index.xml">my Computer Lab account</a>.
One mini gotcha is that in the <code>forest.toml</code> configuration, the site URL must end with a trailing slash
or else the output is corrupted (it bakes in the absolute URL but I've not checked why):</p>
<pre><code class="language-toml">[forest]
trees  = ["trees"]
assets = ["assets"]
url    = "https://www.cl.cam.ac.uk/~avsm2/fcs/" 
</code></pre>
<p>This has all worked out pretty well. The only thing left is to discuss how to handle our <a href="https://www.cst.cam.ac.uk/teaching/exams/hod-notice/part-ia">tick system</a> with <a href="https://jon.recoil.org">Jon Ludlam</a>. I'm going to have a go at porting over <a href="https://realworldocaml.org">Real World OCaml</a> to this as well, to see if it makes managing the refresh of that book a little easier...</p>
<p>You can find the <a href="https://tangled.org/jonmsterling.com/ocaml-forester">Forester source on Tangled</a> as well as the Markdown frontend <a href="https://patrick.sirref.org/graft/index.xml">Graft</a> if you want to try this out for yourself.</p>
<p><small class="credits"> <em>(Update 16th Aug 2026: Both Patrick and Jon <a href="https://amok.recoil.org/@jonmsterling@mathstodon.xyz/117105480422489671">corrected me</a> that Graft isn't a fork of Forester, but a frontend that converts from Markdown syntax to Forester syntax. If I'd understood that before doing my native Forester port I could probably have saved a bunch of time. Mea culpa!)</em> </small></p>
<div class="footnotes"><ol><li><p></p><p>I took over Foundations of CS from the great <a href="https://lawrencecpaulson.github.io/">Larry Paulson</a> back in 2018 or so. My notes are ported from his original course!</p>
 <a href="https://anil.recoil.org/news.xml#fnref:1" class="reversefootnote">↩</a><p></p></li></ol></div><h1>References</h1><ul><li>Madhavapeddy (2025). Foundations of Computer Science. <a href="https://doi.org/10.59350/qms3q-ymn65" target="_blank"><i>10.59350/qms3q-ymn65</i></a></li></ul>
