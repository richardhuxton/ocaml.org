---
title: 'Neocaml 0.9: A Better REPL, Dune/Opam Completion, and More Robustness'
description: "It\u2019s been a couple of months since the last neocaml release, and
  the reason is simple \u2014 for a while there I was genuinely out of ideas. Back
  when I shipped 0.6 I declared (again!) that I was done with new features, and this
  time I almost meant it. But ideas have a way of creeping back in, and 0.9 turned
  out to be a meaty release. Here are the highlights."
url: https://batsov.com/articles/2026/06/24/neocaml-0-9-repl-completion-and-robustness/
date: 2026-06-24T07:00:00-00:00
preview_image: https://batsov.com/assets/img/og-image.png
authors:
- Bozhidar Batsov
source:
ignore:
---

<p>It’s been a couple of months since the last <a href="https://github.com/bbatsov/neocaml">neocaml</a>
release, and the reason is simple — for a while there I was genuinely out of ideas.
Back when I shipped <a href="https://batsov.com/articles/2026/03/25/neocaml-0-6-opam-dune-and-more/">0.6</a>
I declared (again!) that I was done with new features, and this time I almost meant
it. But ideas have a way of creeping back in, and 0.9 turned out to be a meaty
release. Here are the highlights.</p>

<h2>A much nicer REPL experience</h2>

<p>The biggest chunk of work went into the REPL (toplevel) integration. I’m well aware
that the OCaml toplevel isn’t terribly popular with seasoned OCaml developers — most
of them reach for a proper build and a debugger instead. But I think newcomers get a
lot of mileage out of a REPL, and (no surprise to anyone who’s followed my work) I’m
a Lisper at heart with a real soft spot for interactive development. Clojure and
Emacs Lisp spoiled me, and I want OCaml beginners to taste a bit of that too.</p>

<p>So, what’s new:</p>

<ul>
  <li><strong>A dedicated REPL per project.</strong> The REPL buffer is now named after its project
(e.g. <code class="language-plaintext highlighter-rouge">*OCaml: myproject*</code>), and the send commands route to the current buffer’s
project REPL. You can have several projects running side by side without them
stepping on each other.</li>
  <li>
    <p><strong>Choose your toplevel.</strong> The new <code class="language-plaintext highlighter-rouge">neocaml-repl-flavor</code> lets you pick between
<code class="language-plaintext highlighter-rouge">ocaml</code>, <code class="language-plaintext highlighter-rouge">utop</code>, and <code class="language-plaintext highlighter-rouge">dune-utop</code>. Set it globally, or per project via
<code class="language-plaintext highlighter-rouge">.dir-locals.el</code>:</p>

    <div class="language-emacs-lisp highlighter-rouge"><div class="highlight"><pre class="highlight"><code><table class="rouge-table"><tbody><tr><td class="rouge-gutter gl"><pre class="lineno">1
</pre></td><td class="rouge-code"><pre><span class="p">((</span><span class="nv">neocaml-mode</span> <span class="o">.</span> <span class="p">((</span><span class="nv">neocaml-repl-flavor</span> <span class="o">.</span> <span class="nv">dune-utop</span><span class="p">))))</span>
</pre></td></tr></tbody></table></code></pre></div>    </div>

    <p>The active flavor shows up in the REPL’s mode line, so you always know what you’re
talking to.</p>
  </li>
  <li><strong>Send a phrase and step.</strong> <code class="language-plaintext highlighter-rouge">C-c C-n</code> (<code class="language-plaintext highlighter-rouge">neocaml-repl-send-phrase-and-step</code>) sends
the phrase at point to the REPL and moves on to the next one — perfect for walking
through a file top to bottom while you experiment.</li>
  <li><strong><code class="language-plaintext highlighter-rouge">#require</code> from Emacs.</strong> <code class="language-plaintext highlighter-rouge">neocaml-repl-require</code> loads a <code class="language-plaintext highlighter-rouge">findlib</code> package into the
running toplevel without you having to type the directive by hand.</li>
  <li><strong>Restart on demand.</strong> <code class="language-plaintext highlighter-rouge">neocaml-repl-restart</code> kills and restarts the toplevel when
things get into a weird state.</li>
</ul>

<h2>Completion for Opam and Dune, no LSP required</h2>

<p>This one I’m particularly happy with. <code class="language-plaintext highlighter-rouge">.ml</code>/<code class="language-plaintext highlighter-rouge">.mli</code> files have <code class="language-plaintext highlighter-rouge">ocaml-lsp-server</code> to
lean on for completion, but the auxiliary file formats have no language server at all.
That’s exactly the kind of gap neocaml is meant to fill, so both <code class="language-plaintext highlighter-rouge">neocaml-dune-mode</code>
and <code class="language-plaintext highlighter-rouge">neocaml-opam-mode</code> now ship a <code class="language-plaintext highlighter-rouge">completion-at-point</code> backend.</p>

<p>In a <code class="language-plaintext highlighter-rouge">dune</code> file you get completion for stanza names, the field names valid for the
<em>enclosing</em> stanza, and library names inside <code class="language-plaintext highlighter-rouge">libraries</code>/<code class="language-plaintext highlighter-rouge">pps</code> fields:</p>

<pre><code class="language-dune">(library
 (name my_lib)
 (libraries str re|))   ; &lt;- completes both your own libraries and installed findlib ones
</code></pre>

<p>The library candidates combine your project’s own libraries with whatever’s installed
in the active Opam switch. And it’s switch-aware — if there’s a project-local switch
(an <code class="language-plaintext highlighter-rouge">_opam/</code> directory), neocaml detects it and queries it via <code class="language-plaintext highlighter-rouge">opam exec --</code>,
without any configuration on your part. The results are cached per project, so it
stays snappy.</p>

<p>In <code class="language-plaintext highlighter-rouge">opam</code> files you get completion for field and section names, and package names
inside <code class="language-plaintext highlighter-rouge">depends</code>/<code class="language-plaintext highlighter-rouge">depopts</code>/<code class="language-plaintext highlighter-rouge">conflicts</code> (sourced from <code class="language-plaintext highlighter-rouge">opam list</code>):</p>

<pre><code class="language-opam">depends: [
  "dune" {&gt;= "3.0"}
  "cmd|"   ; &lt;- completes to cmdliner and friends
]
</code></pre>

<p>Both backends can be toggled off via <code class="language-plaintext highlighter-rouge">neocaml-dune-complete-libraries</code> and
<code class="language-plaintext highlighter-rouge">neocaml-opam-complete-packages</code> if you’d rather not have them.</p>

<h2>Robustness improvements</h2>

<p>A good chunk of this release is the unglamorous but important work of making things
just behave correctly:</p>

<ul>
  <li><strong>Character literals and quoted strings at the syntactic layer.</strong> Tree-sitter
fontifies <code class="language-plaintext highlighter-rouge">'"'</code>, <code class="language-plaintext highlighter-rouge">'('</code>, and <code class="language-plaintext highlighter-rouge">{|raw "strings"|}</code> correctly, but the syntax table
underneath was getting confused — which broke <code class="language-plaintext highlighter-rouge">forward-sexp</code>, <code class="language-plaintext highlighter-rouge">delete-pair</code>, and
<code class="language-plaintext highlighter-rouge">electric-pair-mode</code> around those constructs. The fix was a good old
<code class="language-plaintext highlighter-rouge">syntax-propertize-function</code>. I wrote up the whole story over on Emacs Redux in
<a href="https://emacsredux.com/blog/2026/06/22/tree-sitter-modes-still-need-a-syntax-table/">Tree-sitter Modes Still Need a Syntax Table</a>,
if you’re into mode-writing internals.</li>
  <li><strong><code class="language-plaintext highlighter-rouge">project.el</code> integration.</strong> A directory with a <code class="language-plaintext highlighter-rouge">dune-project</code> file is now
recognized as a project root (even without version control), <code class="language-plaintext highlighter-rouge">_build/</code> and <code class="language-plaintext highlighter-rouge">_opam/</code>
are ignored, and <code class="language-plaintext highlighter-rouge">compile-command</code> defaults to <code class="language-plaintext highlighter-rouge">dune build</code> in dune projects.</li>
</ul>

<p>There’s more in there too — <code class="language-plaintext highlighter-rouge">ocamlformat</code> integration (<code class="language-plaintext highlighter-rouge">C-c C-f</code>), clickable URLs and
bug references in comments, a font-lock level selector, and richer menus across
the modes.</p>

<h2>OCaml 5.5 support, and the ABI 14 balancing act</h2>

<p><a href="https://discuss.ocaml.org/t/ocaml-5-5-0-released/18265">OCaml 5.5 was released</a> on
June 19th, so this felt like a good moment to ship 5.5 support in neocaml. The <code class="language-plaintext highlighter-rouge">ocaml</code>
and <code class="language-plaintext highlighter-rouge">ocaml-interface</code> grammars now track tree-sitter-ocaml v0.25.0, which brings the
5.5 grammar along with it.</p>

<p>There’s a wrinkle here that’s worth explaining, because it’s shaped the last few
releases. A tree-sitter grammar gets compiled into a parser that speaks a particular
<strong>ABI version</strong>, and Emacs can only load parsers up to the latest ABI supported by the
<code class="language-plaintext highlighter-rouge">libtree-sitter</code> it was <em>built against</em> — it’s not the Emacs version itself that sets
the ceiling. In practice a lot of Emacs 30 builds out there (notably Homebrew’s on
macOS) are linked against tree-sitter 0.24, which tops out at <strong>ABI 14</strong>; you need an
Emacs built against tree-sitter 0.25+ to load <strong>ABI 15</strong> grammars. The trouble is that
the tree-sitter 0.25 CLI now generates ABI 15 parsers by default, so any grammar
regenerated with current tooling produces something those builds simply can’t load —
you install it and it just errors out. Emacs 31 will ship with newer tree-sitter and
make ABI 15 the common case, but it’s not out yet. (This isn’t a neocaml problem as
such; it’s been biting tree-sitter modes across the ecosystem.)</p>

<p>After a few users ran into exactly this, I’ve made a deliberate decision: <strong>stick to
ABI 14 grammars until Emacs 31 is widely available.</strong> That effort started a couple of
releases back — in 0.8.1 I lowered the ABI requirement from 15 to 14 across the opam,
dune, and ocamllex modes, switched the menhir recipe to
<a href="https://github.com/tmcgilchrist/tree-sitter-menhir">tmcgilchrist/tree-sitter-menhir</a>,
and pinned ocamllex back to v0.24.0, all of which target ABI 14
(<a href="https://github.com/bbatsov/neocaml/issues/42">#42</a>). 0.9 extends that policy to the
core OCaml grammars.</p>

<p>The catch with v0.25.0 is precisely that it generates an ABI 15 parser. Happily, the
5.5 grammar didn’t actually need any ABI 15 features — the bump rode along with the CLI
upgrade — so an ABI 14 regeneration of the very same grammar is a drop-in. Big thanks
to <a href="https://github.com/314eter">314eter</a>, the <code class="language-plaintext highlighter-rouge">tree-sitter-ocaml</code> maintainer, for cutting
a <code class="language-plaintext highlighter-rouge">v0.25.0-abi14</code> tag for exactly this purpose
(<a href="https://github.com/tree-sitter/tree-sitter-ocaml/issues/141">#141</a>). The one snag was
that tagging normally triggers releases to NPM, crates.io, and PyPI, so I sent a small
PR to skip publishing for ABI-suffixed tags
(<a href="https://github.com/tree-sitter/tree-sitter-ocaml/pull/142">#142</a>), and the tag
followed. neocaml now pins both grammars to it.</p>

<h2>Roadmap and docs</h2>

<p>If you’re curious where neocaml is headed, I’ve started keeping a
<a href="https://github.com/bbatsov/neocaml/blob/main/ROADMAP.md">ROADMAP.md</a> with ideas and
guiding principles (short version: tree-sitter first, lean on the LSP stack for
<code class="language-plaintext highlighter-rouge">.ml</code>/<code class="language-plaintext highlighter-rouge">.mli</code>, and own the auxiliary modes that have no language server). The project
also has a proper documentation site now at
<a href="https://neocaml.org">neocaml.org</a>, so there’s a real
home for the details beyond the README.</p>

<h2>Give it a Try</h2>

<p>As always — update from <a href="https://melpa.org/#/neocaml">MELPA</a>, play with it, and let
me know how it goes. The full list of changes is in the
<a href="https://github.com/bbatsov/neocaml/releases/tag/v0.9.0">0.9.0 release notes</a>. Bug
reports, feature requests, and pull requests are all welcome on
<a href="https://github.com/bbatsov/neocaml">GitHub</a>.</p>

<p>That’s all from me, folks! Keep hacking!</p>
