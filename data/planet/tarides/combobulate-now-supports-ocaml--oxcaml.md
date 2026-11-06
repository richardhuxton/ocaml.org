---
title: Combobulate Now Supports OCaml & OxCaml
description: Combobulate now supports OCaml and OxCaml, bringing tree-sitter structural
  navigation and editing to Emacs users working in either language.
url: https://tarides.com/blog/2026-10-01-combobulate-now-supports-ocaml-oxcaml
date: 2026-10-01T00:00:00-00:00
preview_image: https://tarides.com/blog/images/blue-lamp-computer-1360w.webp
authors:
- Tarides
source:
ignore:
---

<p>Code navigation is one of those aspects of programming that can either make your experience significantly better, or be such a pain. Most of the time we navigate code as text, i.e, searching with <code>regexp</code>, jumping by lines or moving word by word. But code isn't text. It has syntactic structure, and being aware of that structure when moving and editing opens up a different way of working. This is what <strong>structural navigation and editing</strong> means: operating on the actual constructs of a program; expressions, bindings, match arms, module definitions, etc,  instead of characters and lines.</p>
<p>We have recently improved navigation in OCaml/Oxcaml with <a href="https://github.com/mickeynp/combobulate">Combobulate</a> support, and this post will get you up-to-speed on what’s new, how it works, and where to try it out!</p>
<h2>What Has Structural Navigation in OCaml Looked Like Until Now?</h2>
<p>OCaml already has a substrate of structural navigation through Merlin (and by extension OCaml-LSP). The <a href="https://github.com/ocaml/merlin/blob/master/doc/dev/PROTOCOL.md"><code>jump</code> command</a> gives you a limited form of structural movement such as jumping to the next <code>let</code>, <code>match</code>, <code>module</code>, and a few other constructs. It is useful, but it's a small subset of what structural navigation could be. Previously, this limitation motivated the <a href="https://kirancodes.me/pdfs/gopcaml-presentation-ocaml21.pdf">GopCaml</a> project, which took a more ambitious approach to structural editing for OCaml by working directly with the compiler's AST.</p>
<p>More recently, <a href="https://tree-sitter.github.io/tree-sitter/">tree-sitter</a> has introduced a generic abstraction over syntax. Given a tree-sitter grammar for a language, you get an incremental parser that produces a concrete syntax tree you can query and traverse. OCaml has a tree-sitter grammar which is already used in, for example, <a href="https://github.com/chsasank/neocaml-mode">neocaml-mode</a> where it provides syntax highlighting.</p>
<p>Tree-sitter can be seen as the syntactic counterpart to LSP: where LSP standardizes semantic features, Tree-sitter provides a common protocol for syntax. Much like TextMate grammars provided a generic way to handle syntax highlighting across editors, Tree-sitter gives editors syntactic tools such as highlighting and navigation.</p>
<p><a href="https://github.com/mickeynp/combobulate">Combobulate</a> by <a href="http://www.masteringemacs.org">Mickey Petersen</a> takes tree-sitter in a different direction: it uses the syntax tree for structural navigation and editing. It's a minor mode for <a href="https://www.gnu.org/software/emacs/">Emacs</a> that supports many languages, and it now supports OCaml.</p>
<h2>Why Does Combobulate Matter for OCaml?</h2>
<p>OCaml code nests very deeply. Modules contain structures, structures contain let bindings, let bindings contain match expressions, and match cases can contain further match expressions. Type declarations can define records, variants, and GADTs in a single <code>type ... and ...</code> block. Many of these constructs can recurse into each other with no fixed limit; this is part of what makes OCaml expressive, but it also means that even a small OCaml file produces a deep and wide tree-sitter parse tree.</p>
<p>Implementing structural navigation for OCaml is harder than for most languages precisely because of this: the procedures that tell Combobulate how to pick the right node at any point have to account for potentially infinite nesting at every level. This is also why line-based movement becomes incredibly slow and unreliable. Jumping to the next <code>let</code> with an incremental search won’t help when there are six of them nested within each other. This is why structural navigation, which helps us move by the structure of the code, and the relationships between different nodes in the tree, feels natural and makes a real difference.</p>
<p>Combobulate is an important addition to the OCaml ecosystem because it perfectly complements tools like Merlin and OCaml-LSP. While Merlin is great for semantic intelligence, type checking, autocomplete, and jumping to definitions, its structural navigation features (like the <code>jump</code> command) are limited. By letting Combobulate handle the purely syntactic, structural movement and editing, the two tools work together to provide a comprehensive editing experience: Merlin understands what your code <em>means</em>, while Combobulate understands its <em>shape</em>.</p>
<h2>Navigating OCaml with Combobulate</h2>
<p>Once Combobulate is active in your OCaml buffer, you should see a <code>©</code> in the mode line. There is a Magit-style <a href="https://docs.magit.vc/transient/Introduction.html">transient</a> UI bound to <kbd>C-c o o</kbd> that lists every binding, which is handy while you're learning. To inspect the full keymap directly, run <code>M-x describe-keymap RET combobulate-key-map</code>.</p>
<p>With Combobulate, you have different commands to navigate your code in a variety of ways: jumping between siblings, jumping between occurrences of words, traversing the node tree sequentially, and more.</p>
<h3>Navigation Commands</h3>
<div role="region"><table>
<tbody><tr>
<th>Binding</th>
<th>Summary</th>
<th>What it does</th>
</tr>
<tr>
<td><kbd>C-M-u</kbd> / <kbd>C-M-d</kbd></td>
<td>Up/Down into list</td>
<td>Move in/out to the parent/child node.</td>
</tr>
<tr>
<td><kbd>C-M-n</kbd> / <kbd>C-M-p</kbd></td>
<td>Forward/Backward sibling</td>
<td>Move to the next/previous sibling at the current level.</td>
</tr>
<tr>
<td><kbd>M-e</kbd> / <kbd>M-a</kbd></td>
<td>Logical next/previous</td>
<td>Jump to the next/previous logical node, regardless of nesting.</td>
</tr>
<tr>
<td><kbd>M-n</kbd> / <kbd>M-p</kbd></td>
<td>Sequence navigation</td>
<td>Move between paired sequence points (e.g., jumping from the word <code>let</code> to the next occurrence of <code>let</code>).</td>
</tr>
<tr>
<td><kbd>C-M-a</kbd> / <kbd>C-M-e</kbd></td>
<td>Move to the start/end of defun</td>
<td>Move to the beginning/end of defun. This is based on best-effort. In nested let bindings, it doesn't work very well.</td>
</tr>
</tbody></table></div><h4>Navigation Examples</h4>
<p>Combobulate primarily handles code navigation in terms of two axes:</p>
<ul>
<li><strong>Vertical/Hierarchical (Parents and Children):</strong> Moving "up" (<kbd>C-M-u</kbd>) leaves the current node for its enclosing parent, while moving "down" (<kbd>C-M-d</kbd>) descends into the child node at the cursor.</li>
<li><strong>Horizontal (Siblings):</strong> Moving forward (<kbd>C-M-n</kbd>) or backward (<kbd>C-M-p</kbd>) hops between sibling nodes at the same syntactic level, such as adjacent match cases, list elements, or record fields.</li>
</ul>
<p>When hierarchical or sibling navigation isn't enough, Combobulate also offers <strong>logical</strong> navigation (<kbd>M-e</kbd> / <kbd>M-a</kbd>). Rather than being constrained to direct parent-child or sibling relationships, logical navigation moves sequentially across nodes in their logical reading order—allowing you to cross operator boundaries or escape deeply nested subtrees.</p>
<h5>Simple Examples</h5>
<ul>
<li>
<p><strong>Navigating down into a body (<kbd>C-M-d</kbd>)</strong></p>
<p>"Down" means entering whatever node the cursor is sitting on. The clearest case is descending from a module declaration into its contents:</p>
<pre><code><span class="ocaml-keyword-other">module</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Counter</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">struct</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">value</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">0</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">bump</span><span class="ocaml-source"> </span><span class="ocaml-source">x</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">x</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">+</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">1</span><span class="ocaml-source">
</span><span class="ocaml-keyword-other">end</span><span class="ocaml-source">
</span></code></pre>
<p>Place the cursor on <code>module</code>. Press <kbd>C-M-d</kbd> thrice and the cursor moves to <code>let value = 0</code>. Press <kbd>C-M-d</kbd> again and you descend further, into the binding itself.</p>
<p><img src="https://tarides.com/blog/images/2026-05-06.combobulate/down~f2q5ViE02M0u9h98ebFsuw.gif" alt="Down navigation"></p>
</li>
<li>
<p><strong>Navigating up to the parent (<kbd>C-M-u</kbd>)</strong></p>
<p>"Up" is the inverse: leave the current node and land on its enclosing parent. Suppose the cursor is on the number <code>100</code> inside a record:</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">player</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">{</span><span class="ocaml-source"> </span><span class="ocaml-source">name</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-string-quoted-double">"</span><span class="ocaml-string-quoted-double">Ada</span><span class="ocaml-string-quoted-double">"</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-source">score</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">100</span><span class="ocaml-source"> </span><span class="ocaml-source">}</span><span class="ocaml-source">
</span></code></pre>
<p><kbd>C-M-u</kbd> jumps to the whole field <code>score = 100</code>. Press it again to land on the record <code>{ ... }</code>. To move from <code>100</code> directly to the <code>let</code> keyword, use <kbd>C-M-a</kbd>.</p>
<p><img src="https://tarides.com/blog/images/2026-05-06.combobulate/up~VdvGwiHdfl66AfzUhCZ8Pw.gif" alt="Up navigation"></p>
</li>
<li>
<p><strong>Navigating siblings (<kbd>C-M-n</kbd> / <kbd>C-M-p</kbd>)</strong></p>
<p>Siblings are nodes at the same level, like match cases, tuple components, record fields, and array elements. Take a <code>match</code> expression:</p>
<pre><code><span class="ocaml-keyword-other">match</span><span class="ocaml-source"> </span><span class="ocaml-source">shape</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">with</span><span class="ocaml-source">
</span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Circle</span><span class="ocaml-source"> </span><span class="ocaml-source">r</span><span class="ocaml-source">    </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-source">pi</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">*.</span><span class="ocaml-source"> </span><span class="ocaml-source">r</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">*.</span><span class="ocaml-source"> </span><span class="ocaml-source">r</span><span class="ocaml-source">
</span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Square</span><span class="ocaml-source"> </span><span class="ocaml-source">s</span><span class="ocaml-source">    </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-source">s</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">*.</span><span class="ocaml-source"> </span><span class="ocaml-source">s</span><span class="ocaml-source">
</span><span class="ocaml-keyword-other">|</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Triangle</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-source">b</span><span class="ocaml-keyword-other-ocaml punctuation-comma punctuation-separator">,</span><span class="ocaml-source"> </span><span class="ocaml-source">h</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-&gt;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-float">0.5</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">*.</span><span class="ocaml-source"> </span><span class="ocaml-source">b</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">*.</span><span class="ocaml-source"> </span><span class="ocaml-source">h</span><span class="ocaml-source">
</span></code></pre>
<p>Place the cursor on the first match arm (<code>Circle r -&gt; ...</code>). <kbd>C-M-n</kbd> moves to <code>Square s -&gt; ...</code>. Again to <code>Triangle ...</code>. <kbd>C-M-p</kbd> walks back.</p>
<p><img src="https://tarides.com/blog/images/2026-05-06.combobulate/sibling~rjqWhW3Q4Mp4zEflVLLFzQ.gif" alt="Sibling navigation"></p>
</li>
</ul>
<h5>Complex Examples</h5>
<p>Using only parent-child or sibling navigation is not always sufficient to navigate OCaml code efficiently. Because OCaml's deep nesting can lead to highly nested concrete syntax trees, you need a few more tools in your belt to avoid getting stuck.</p>
<ul>
<li>
<p><strong>Example 1: Using next-sequent (<kbd>M-n</kbd>) and prev-sequent (<kbd>M-p</kbd>)</strong></p>
<p>In subsequent <code>let...in</code> bindings, parent-child/sibling navigation is insufficient and unreliable due to how <code>let...in</code> is represented as deeply nested subtrees in the tree-sitter grammar. Each successive binding is actually a child of the one before it, meaning <kbd>C-M-p</kbd> won't walk backwards up the chain. Instead, use sequence navigation to hop directly from one <code>let</code> to the next and back.</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">emit_string_table_section</span><span class="ocaml-source"> </span><span class="ocaml-source">fmt</span><span class="ocaml-source"> </span><span class="ocaml-source">section_name</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-source">(</span><span class="ocaml-source">table</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Dwarf_write</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">string_table</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">buf</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Buffer</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">create</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">64</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">contents</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">Buffer</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">contents</span><span class="ocaml-source"> </span><span class="ocaml-source">buf</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">i</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">ref</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">0</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">len</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">String</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">length</span><span class="ocaml-source"> </span><span class="ocaml-source">contents</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">while</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">!</span><span class="ocaml-source">i</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">&lt;</span><span class="ocaml-source"> </span><span class="ocaml-source">len</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">do</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">start</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">!</span><span class="ocaml-source">i</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword-other">while</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">!</span><span class="ocaml-source">i</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">&lt;</span><span class="ocaml-source"> </span><span class="ocaml-source">len</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">&amp;&amp;</span><span class="ocaml-source"> </span><span class="ocaml-source">contents</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">[</span><span class="ocaml-keyword-operator">!</span><span class="ocaml-source">i</span><span class="ocaml-source">]</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">&lt;&gt;</span><span class="ocaml-source"> </span><span class="ocaml-string-quoted-single">'</span><span class="ocaml-constant-character-escape">\x00</span><span class="ocaml-string-quoted-single">'</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">do</span><span class="ocaml-source">
</span><span class="ocaml-source">      </span><span class="ocaml-source">incr</span><span class="ocaml-source"> </span><span class="ocaml-source">i</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword-other">done</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">s</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-capital-identifier">String</span><span class="ocaml-keyword-other-ocaml punctuation-other-period punctuation-separator">.</span><span class="ocaml-source">sub</span><span class="ocaml-source"> </span><span class="ocaml-source">contents</span><span class="ocaml-source"> </span><span class="ocaml-source">start</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-keyword-operator">!</span><span class="ocaml-source">i</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">-</span><span class="ocaml-source"> </span><span class="ocaml-source">start</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">in</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-source">emit_asciz</span><span class="ocaml-source"> </span><span class="ocaml-source">fmt</span><span class="ocaml-source"> </span><span class="ocaml-source">s</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">    </span><span class="ocaml-keyword-other">if</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">!</span><span class="ocaml-source">i</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">&lt;</span><span class="ocaml-source"> </span><span class="ocaml-source">len</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">then</span><span class="ocaml-source"> </span><span class="ocaml-source">incr</span><span class="ocaml-source"> </span><span class="ocaml-source">i</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other">done</span><span class="ocaml-source">
</span></code></pre>
<p>If we want to move from the let-binding on line 3 to the let-binding on line 6, sequence commands <kbd>M-n</kbd> and <kbd>M-p</kbd> let you jump forward and backward easily.</p>
<p><img src="https://tarides.com/blog/images/2026-05-06.combobulate/sequent~Cl3xjgV89HTi4FFrsybpdg.gif" alt="Sequence navigation"></p>
</li>
<li>
<p><strong>Example 2: Using logical-next (<kbd>M-e</kbd>) and logical-prev (<kbd>M-a</kbd>)</strong></p>
<pre><code><span class="ocaml-keyword-other">if</span><span class="ocaml-source"> </span><span class="ocaml-source">(</span><span class="ocaml-source">x</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">1</span><span class="ocaml-source">)</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">then</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-boolean">true</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">else</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-boolean">false</span><span class="ocaml-source">
</span></code></pre>
<p>When the cursor is on <code>if</code>, you can do <kbd>C-M-d</kbd> to go to the parenthesis <code>(</code>, then <kbd>C-M-d</kbd> again to enter <code>x</code>, or <kbd>C-M-n</kbd> to go to <code>then</code> and <code>else</code>.</p>
<p>However, if we have the same code <em>without</em> the parenthesis:</p>
<pre><code><span class="ocaml-keyword-other">if</span><span class="ocaml-source"> </span><span class="ocaml-source">x</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">1</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">then</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-boolean">true</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">else</span><span class="ocaml-source"> </span><span class="ocaml-constant-language-boolean">false</span><span class="ocaml-source">
</span></code></pre>
<p>There is no direct sibling relationship to go from <code>x</code> to <code>then</code> using <kbd>C-M-d</kbd> or <kbd>C-M-n</kbd>. In this case, we use <code>logical-next</code> (<kbd>M-e</kbd>) to cross the operator boundary and jump directly to the <code>then</code> branch.</p>
<p>Logical next/prev allows you to move to the next node in the tree irrespective of their parent/sibling relationships. It is also incredibly helpful for passing over <code>-&gt;</code>, <code>=</code>, and other operators.</p>
</li>
<li>
<p><strong>Example 3: Escaping Deep Subtrees</strong></p>
<p>If you are at the end of a long top-level item and want to navigate to the beginning of the next top-level item, use logical-next (<kbd>M-e</kbd>). If you try to use forward sibling navigation (<kbd>C-M-n</kbd>) from the end of the item, the cursor won't move at all since you are deep inside a nested subtree with no siblings to your right. Using <kbd>M-e</kbd> lets you jump out of the subtree instantly to the next top-level construct.</p>
</li>
</ul>
<h2>Editing Commands</h2>
<p>Because Combobulate's editing commands are built on top of its navigation primitives, particularly sibling navigation, they all work in OCaml without any extra configuration. If you can navigate between two nodes, you can edit them.</p>
<div role="region"><table>
<tbody><tr>
<th>Binding</th>
<th>Summary</th>
<th>What it does</th>
</tr>
<tr>
<td><kbd>C-c o e</kbd></td>
<td>Envelope prefix</td>
<td>Apply a code template (envelope) at the cursor. Press <kbd>C-h</kbd> after to see what's available in this context.</td>
</tr>
<tr>
<td><kbd>M-h</kbd></td>
<td>Expand region</td>
<td>Mark the current node. Repeat to expand the region to the parent iteratively.</td>
</tr>
<tr>
<td><kbd>C-M-h</kbd></td>
<td>Mark defun</td>
<td>Mark the current enclosing defun. Repeat to expand to the next enclosing defun iteratively.</td>
</tr>
<tr>
<td><kbd>M-N</kbd> or <kbd>M-S-n</kbd></td>
<td>Drag forward</td>
<td>Swap the current node with its next sibling, preserving formatting.</td>
</tr>
<tr>
<td><kbd>M-P</kbd> or <kbd>M-S-p</kbd></td>
<td>Drag backward</td>
<td>Swap the current node with its previous sibling.</td>
</tr>
<tr>
<td><kbd>C-c o c</kbd></td>
<td>Clone node dwim</td>
<td>Duplicate the node at cursor. If ambiguous, you cycle through candidates with a live preview (the <em>carousel</em>).</td>
</tr>
<tr>
<td><kbd>C-c o t</kbd></td>
<td>Place cursors</td>
<td>Place multiple cursors (or field-editor fields) at every related sibling; e.g. each element of an array, each field in a record.</td>
</tr>
</tbody></table></div><h3>Editing Examples</h3>
<ul>
<li>
<p><strong>Expanding the region (<kbd>M-h</kbd>)</strong></p>
<p>Each press grows the selection to the next syntactic unit. Starting on <code>r</code> inside a function call:</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">area</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">pi</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">*.</span><span class="ocaml-source"> </span><span class="ocaml-source">r</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">*.</span><span class="ocaml-source"> </span><span class="ocaml-source">r</span><span class="ocaml-source">
</span></code></pre>
<ul>
<li><code>M-h</code> once → selects <code>r</code>.</li>
<li><code>M-h</code> again → selects <code>pi *. r *. r</code>.</li>
<li><code>M-h</code> again → selects the whole <code>let</code> binding.</li>
</ul>
<p><code>M-h</code> displays numbers indicating where the next enclosing region starts, helping you visualize where the cursor will move if you perform a hierarchy-up navigation. Unlike Merlin's <code>type-enclosing</code> (which operates on typed AST expressions and requires code to typecheck), Combobulate's expansion is purely syntactic: it operates on any concrete syntax node (including patterns, type declarations, and comments) even when the code is incomplete or doesn't compile.</p>
</li>
<li>
<p><strong>Expanding an envelope (<kbd>C-c o e</kbd>)</strong></p>
<p>Envelopes are context-aware templates. Press <kbd>C-c o e</kbd> then <kbd>C-h</kbd> to see what's available.</p>
<p>For example, to add a module template:</p>
<ul>
<li>
<p>Place your cursor where you want to add the template.</p>
</li>
<li>
<p>Press <kbd>C-c o e</kbd> to list all available templates.</p>
</li>
<li>
<p>Press <kbd>M</kbd> to activate the modules template.</p>
</li>
<li>
<p>The template will be added with <code>name</code> as an editable hole:</p>
<pre><code><span class="ocaml-keyword-other">module</span><span class="ocaml-source"> </span><span class="ocaml-source">name</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other">struct</span><span class="ocaml-source">
</span><span class="ocaml-source">
</span><span class="ocaml-keyword-other">end</span><span class="ocaml-source">
</span></code></pre>
</li>
<li>
<p>Press <kbd>TAB</kbd> to jump between holes.</p>
</li>
</ul>
</li>
<li>
<p><strong>Adding multiple cursors (<kbd>C-c o t</kbd>)</strong></p>
<p>Cursors land on every sibling at the current level. This is perfect for bulk-editing collections. Place the cursor on any element of an array:</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">primes</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">[|</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">2</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">3</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">5</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">7</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">11</span><span class="ocaml-source"> </span><span class="ocaml-source">|]</span><span class="ocaml-source">
</span></code></pre>
<p>Press <kbd>C-c o t t</kbd> and a cursor is placed on each element. Anything you type happens to all five at once!</p>
</li>
<li>
<p><strong>Swapping siblings — drag forward / backward (<kbd>M-N</kbd> / <kbd>M-P</kbd>)</strong></p>
<p>Drag transposes the node at the cursor with its neighbor, preserving formatting. Useful for reordering elements or record fields:</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">primes</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">[|</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">2</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">3</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">5</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">7</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">11</span><span class="ocaml-source"> </span><span class="ocaml-source">|]</span><span class="ocaml-source">
</span></code></pre>
<p>With the cursor on <code>2</code>, press <kbd>M-N</kbd> (or <kbd>M-S-N</kbd>) to swap them:</p>
<pre><code><span class="ocaml-keyword">let</span><span class="ocaml-source"> </span><span class="ocaml-entity-name-function-binding">primes</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">[|</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">3</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">2</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">5</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">7</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source"> </span><span class="ocaml-constant-numeric-decimal-integer">11</span><span class="ocaml-source"> </span><span class="ocaml-source">|]</span><span class="ocaml-source">
</span></code></pre>
</li>
<li>
<p><strong>Cloning a node (<kbd>C-c o c</kbd>)</strong></p>
<p>Duplicates the node at the cursor. On a record field:</p>
<pre><code><span class="ocaml-keyword-other">type</span><span class="ocaml-source"> </span><span class="ocaml-source">user</span><span class="ocaml-source"> </span><span class="ocaml-keyword-operator">=</span><span class="ocaml-source"> </span><span class="ocaml-source">{</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-source">name</span><span class="ocaml-source"> </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-support-type">string</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">  </span><span class="ocaml-source">age</span><span class="ocaml-source">  </span><span class="ocaml-keyword-other-ocaml punctuation-other-colon punctuation">:</span><span class="ocaml-source"> </span><span class="ocaml-support-type">int</span><span class="ocaml-keyword-other-ocaml punctuation-separator-terminator punctuation-separator">;</span><span class="ocaml-source">
</span><span class="ocaml-source">}</span><span class="ocaml-source">
</span></code></pre>
<p>Place the cursor on <code>name : string</code> and press <kbd>C-c o c</kbd> to duplicate it seamlessly.</p>
</li>
</ul>
<h2>Inspection &amp; Search</h2>
<div role="region"><table>
<tbody><tr>
<th>Binding</th>
<th>Summary</th>
<th>What it does</th>
</tr>
<tr>
<td><kbd>C-c o B q</kbd></td>
<td>Query builder</td>
<td>Open the interactive tree-sitter query builder, with completion and highlighting, for ad-hoc searches and bulk edits.</td>
</tr>
</tbody></table></div><h3>Query Builder Example</h3>
<p>Open a live tree-sitter query builder with <kbd>C-c o B q</kbd>. If you have <code>value_definition</code>s in your file, you can underline all of them with a blue line using the query:</p>
<pre><code class="language-scheme">(value_definition) @hl.blue.underline
</code></pre>
<h2>Setup</h2>
<p>Since Combobulate is built on tree-sitter you will need Emacs 29 or later, as that's when built-in tree-sitter support landed. Install <a href="https://github.com/mickeynp/combobulate#getting-started-with-combobulate">Combobulate</a> from the master branch and add the OCaml grammars to your config file.</p>
<p>To get started with OCaml, add the OCaml grammars to your config file:</p>
<pre><code class="language-elisp">(setq treesit-language-source-alist
      '((ocaml . ("https://github.com/tree-sitter/tree-sitter-ocaml"
                  "v0.26.0" "grammars/ocaml/src"))
        (ocaml_interface ("https://github.com/tree-sitter/tree-sitter-ocaml"
                            "v0.26.0" "grammars/interface/src"))))
</code></pre>
<p>Run <code>M-x treesit-install-language-grammar</code> for each.</p>
<p>Combobulate can be used with either <code>neocaml-mode</code> or <code>tuareg-mode</code> or <code>tuareg-interface-mode</code> as your major mode. When it's working you'll see <code>©</code> in the mode line, and <code>C-c o o</code> opens the full command palette.</p>
<p><img src="https://tarides.com/blog/images/combobulate-full-command-1360w~i5MLp_b6HtLgrtDI7TFtUA.webp" sizes="(min-width: 1360px) 1360px, (min-width: 680px) 680px, 100vw" srcset="/blog/images/combobulate-full-command-170w~uPwygjvKyjWPtucQ_3pN6A.webp 170w, /blog/images/combobulate-full-command-340w~8asLf7zW_KkmhALr04rhtQ.webp 340w, /blog/images/combobulate-full-command-680w~cquK9p-8L2uyZRJvnXfGHA.webp 680w, /blog/images/combobulate-full-command-1360w~i5MLp_b6HtLgrtDI7TFtUA.webp 1360w" alt="The full command palette for Combobulate"></p>
<h2>Try it out</h2>
<p>Open up a project you are working on. Place your cursor on a case in a match expression and try to teleport to the next sibling.</p>
<p>You can check out the <a href="https://github.com/mickeynp/combobulate/pull/157">PR adding OCaml support</a> and the <a href="https://github.com/mickeynp/combobulate/pull/206">PR adding OxCaml support</a> in the Combobulate repo to explore the implementation process in more detail.</p>
<h2>Feedback Welcome</h2>
<p>OCaml's syntax is flexible enough that there isn't always one obvious answer to "what should the next sibling be?" or "what counts as descending one level?". We had to make judgment calls on a number of corner cases, like what sibling navigation does inside a <code>type ... and ...</code> block, how hierarchy behaves around functors, where sibling navigation should land in deeply nested expressions. We're happy with the choices we made, but we know they won't match everyone's expectations perfectly. If something feels off in your workflow, or you think a particular movement should behave differently, we'd like to hear about it. Open an issue on the <a href="https://github.com/mickeynp/combobulate/issues">Combobulate repo</a>, make a post on <a href="https://discuss.ocaml.org">Discuss</a>, or <a href="https://tarides.com/contact/">contact us</a> to let us know.</p>
<p>Stay in touch  with us on <a href="https://bsky.app/profile/tarides.com">Bluesky</a>, <a href="https://mastodon.social/@tarides">Mastodon</a>, and <a href="https://www.linkedin.com/company/tarides">LinkedIn</a> or sign up to our mailing list to stay updated on our latest projects. We look forward to hearing from you!</p>

