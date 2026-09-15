---
title: Using ocaml-lsp-server via an MCP server
description:
url: https://jon.recoil.org/blog/2025/08/ocaml-lsp-mcp.html
date: 2025-08-27T00:00:00-00:00
preview_image:
authors:
- Jon Ludlam
source:
ignore:
---


      <p>Here's a quick post on how to get the OCaml Language Server (ocaml-lsp-server) working with an MCP server.</p>
<p>We're going to use <a href="https://github.com/isaacphi">issacphi</a>'s adapter for LSP servers, which is written in go. So install go, and then:</p>
<pre><code class="language-bash">go install github.com/isaacphi/mcp-language-server@latest
</code></pre>
<p>Once that's done, make sure you've got `ocaml-lsp-server` installed in your switch:</p>
<pre><code class="language-bash">opam install ocaml-lsp-server
</code></pre>
<p>Then add the MCP config for claude where you want to run it:</p>
<pre><code class="language-bash">claude mcp add ocamllsp -s local -t stdio -- /Users/jon/go/bin/mcp-language-server -workspace . -lsp ocamllsp
</code></pre>
<p>It'd be nice to get this working `globally` - that is, with `-s user` - but I haven't been able to get that to work yet.</p>

    
