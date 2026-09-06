---
title: This site
description:
url: https://jon.recoil.org/blog/2025/04/this-site.html
date: 2025-04-07T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---

<h1><a href="https://jon.recoil.org/atom.xml#this-site" class="anchor"></a>This site</h1>
<ul class="at-tags"><li class="x-ocaml.requires"><span class="at-tag">x-ocaml.requires</span> <p>mime_printer</p></li></ul>
<ul class="at-tags"><li class="published"><span class="at-tag">published</span> <p>2025-04-07</p></li></ul>
<ul class="at-tags"><li class="page-tags"><span class="at-tag">page-tags</span> <p><a href="https://jon.recoil.org/tags/odoc.html" title="odoc">odoc</a> <a href="https://jon.recoil.org/tags/meta.html" title="meta">meta</a></p></li></ul>
<p>I've spent a <em>lot</em> of time over the past few years working on Odoc, the OCaml documentation generator, so when it came time to (re)start my own website and blog, I found it hard to resist thinking about how I might use odoc as part of it. We've spent a lot of time recently trying to make odoc more able to generate structured documentation sites, so I've gone all in and am trialling using it as a tool to generate my entire site. This is a bit of an experiment, and I don't know how well it will work out, but let's see how it goes.</p>
<p>Additionally, I've recently been working on a project currently called <code>odoc_notebook</code>, which is a set of tools to allow odoc <code>mld</code> files to be used as a sort of Jupyter-style notebook. The idea is that you can write both text and code in the same file, and then run the code in the notebook interactively. Since I've only got a webserver, all the execution of code has to be done client side, so I'm making extensive use of the phenomenal <a href="https://github.com/ocsigen/js_of_ocaml">Js_of_ocaml</a> project to get an OCaml engine running in the browser.</p>
<p>My focus has initially been on getting 'toplevel-style' code execution working. As an example, let's write a little demo.</p>
<h2><a href="https://jon.recoil.org/atom.xml#demo" class="anchor"></a>Demo</h2>
<p>Let's start with a little demo:</p>
<div><pre class="language-ocaml"><code>let x = 1 + 2</code></pre></div>
<p>It's intended to look like an OCaml toplevel session, so each new expression starts with a <code>#</code> and is terminated with a double semicolon. The response from the toplevel is then below that indented with 2 spaces. Right now, there's not much in the way of error checking so you can make it all very confused by deleting the hash, removing the <code>;;</code> and so on. Avoiding this, however, you can edit the numbers here and hit 'run' (maybe twice!) to see the results being updated.</p>
<p>There is also a little integration to allow the code to produce output more interesting than just text. The following cell creates an SVG image and 'pushes' it to <code>Mime_printer</code>, which receives the mime value and renders it in the browser below the code block.</p>
<div><pre class="language-ocaml"><code>let svg = [
  {|&lt;svg height="210" width="500" xmlns="http://www.w3.org/2000/svg"&gt;|};
  {|&lt;polygon points="100,10 40,198 190,78 10,78 160,198" |};
  {|style="fill:lime;stroke:purple;stroke-width:5;"/&gt;&lt;/svg&gt;|}];;

Mime_printer.push "image/svg" (String.concat "\n" svg)</code></pre></div>
<h2><a href="https://jon.recoil.org/atom.xml#things-to-come" class="anchor"></a>Things to come</h2>
<h3><a href="https://jon.recoil.org/atom.xml#merlin-support" class="anchor"></a>Merlin support</h3>
<p>There are a bunch of things I want to add to this, for example, Merlin support. In fact, <a href="https://github.com/voodoos/merlin-js">merlin-js</a> already exists and works, thanks to the fantastic work of <a href="https://github.com/voodoos">Ulysse</a>, but the problem is that it's not really designed for toplevel work, and it doesn't work when the code is broken up into chunks like I do here. So either I need to concatenate all the cells together before I give it to Merlin, or I need to make each cell it's own little module and 'open' every previous cell's module.</p>
<p>Within a single cell, it does already work. You can see that Merlin is correctly underlining the error in the following cell. You should also be able to hover over the variables and see their types.</p>
<div><pre class="language-ocaml"><code>type t = { foo : int; bar : string };;

let x = { foo = 1; bar = "hello" };;

let this_line_has_an_error = { foo = 1; bar = None };;</code></pre></div>
<p>But across cells, I've broken Merlin, though the code is executes correctly. You can see the problem in the following cell, which re-pushes the SVG image using the variable <code>svg</code> defined in the cell above. Merlin highlights the use of the varible <code>svg</code> is, because it's not aware of the varible, but the code gets executed correctly and the image is rendered below the cell.</p>
<div><pre class="language-ocaml"><code>Mime_printer.push "image/svg" (String.concat "\n" svg)</code></pre></div>
<p>Edit 2025-05-20: I have now got merlin working across cells, though I'm not convinced the current solution is the right long-term solution. S</p>
<h3><a href="https://jon.recoil.org/atom.xml#dynamic-libraries" class="anchor"></a>Dynamic libraries</h3>
<p>Currently the use of libraries it quite limited - they are defined more-or-less statically. I've had dynamic libraries working in the past, but I need to re-implement them. The plan is to have the <code>cma</code> files converted to <code>js</code> files and then load them on-demand when the notebook specifies them. The tricky thing here is that we need to be able to use them both in the browser and in bytecode executables so that the 'test-promote' workflow still works. This will probably require specifying the libraries by name, and having to re-implement the work that <a href="https://projects.camlcity.org/projects/findlib.html">findlib</a> does to find the libraries and load them and their dependencies in the right order, though this time entirely over HTTP.</p>
<h3><a href="https://jon.recoil.org/atom.xml#other-things" class="anchor"></a>Other things</h3>
<p>There are loads of other things I'm interested in doing, including:</p>
<ul><li>Investigating how to do 'exercises' to allow readers to try things out in a guided way</li><li>'Test cells' to see if implementations are correct</li><li>Persistence of the notebook state - both using local and remote storage</li><li>Integration of docs</li><li>Exploration of the execution model - how to run the code in the right order and ensure reproducibility</li><li>Use of remote execution engines rather than just in the browser</li><li>Other languages?</li></ul>
<p>Right now though, my focus is on the functionality required for this blog, with a secondary goal of looking at how we might use this sort of technology on the docs site on ocaml.org. Wouldn't it be cool to be able to drop into a live OCaml toplevel for any library in opam?</p>
<h2><a href="https://jon.recoil.org/atom.xml#example-notebooks" class="anchor"></a>Example notebooks</h2>
<p>As a more extended example of odoc notebooks, I have converted to this format the course that I help teach at the University of Cambridge; <a href="https://www.cl.cam.ac.uk/teaching/2425/FoundsCS/">Foundations of Computer Science</a>. <a href="https://jon.recoil.org/notebooks/foundations/index.html" title="index">Try them out for yourself!</a>.</p>
