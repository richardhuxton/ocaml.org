---
title: Building OxCaml packages for Debian, Fedora, Homebrew and Arch
description: "Native OxCaml system packages for Debian/Ubuntu, Fedora, Arch and Homebrew
  \u2014 plus reviving a 2013 GPG key that modern tooling rejects for SHA-1, and using
  agentic coding to collapse the opam build into one tarball."
url: https://anil.recoil.org/notes/oxcaml-packages
date: 2026-05-17T00:00:00-00:00
preview_image: https://anil.recoil.org/images/faces/avsm.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>I needed to install <a href="https://anil.recoil.org/projects/oxcaml">OxCaml</a> quickly on a fresh machine without
any <a href="https://opam.ocaml.org">opam</a> machinery. This was an excellent excuse
to refresh my memory on how distributing system packages for distributions
like Debian, Arch, Fedora and Homebrew work.</p>
<p>I've built <strong><a href="https://tangled.org/anil.recoil.org/oxcaml-pkgs">oxcaml-pkgs</a></strong>
to churn out native packages for the distros I need, with details below
for future reference.</p>
<p>These all install an <code>oxcaml-compiler</code> package into <code>/opt</code>, since anywhere else
would clash with the native OCaml packaging. The idea is that a consumer can
add this to their PATH to specifically get OxCaml. However, I don't intend
most people to do this manually as I'm wrapping this in my <a href="https://github.com/avsm/oi">oi</a>
at the moment.</p>
<h2><a href="https://anil.recoil.org/news.xml#the-quick-installer-script" class="anchor" aria-hidden="true"></a>The quick installer script</h2>
<p>If you just want OxCaml and don't care how, there's a one-shot installer:</p>
<pre><code class="language-sh">curl -fsSL https://oi.thicket.dev/repo/install.sh | sh
</code></pre>
<p>The rest of this note is the manual breakdown of what that script automates,
mostly so I remember how each packaging system works the next time I revisit
it!</p>
<h2><a href="https://anil.recoil.org/news.xml#debian--ubuntu" class="anchor" aria-hidden="true"></a>Debian / Ubuntu</h2>
<pre><code class="language-sh">curl -fsSL https://oi.thicket.dev/repo/apt/oxcaml.asc | sudo gpg --dearmor -o /usr/share/keyrings/oxcaml.gpg
# pick your release: noble, resolute, or trixie
echo 'deb [signed-by=/usr/share/keyrings/oxcaml.gpg] https://oi.thicket.dev/repo/apt resolute main' | sudo tee /etc/apt/sources.list.d/oxcaml.list
sudo apt update &amp;&amp; sudo apt install oxcaml-compiler
</code></pre>
<p>Debian and Ubuntu both maintain <a href="https://tangled.org/anil.recoil.org/oxcaml-pkgs/tree/main/packaging/deb/debian">packaging metadata in a debian/</a>
directory with various bits of metadata, e.g.:</p>
<pre><code>Source: oxcaml-compiler
Section: devel
Priority: optional
Maintainer: @MAINTAINER@
Build-Depends: debhelper-compat (= 13),
 gcc, g++, make, m4, autoconf, perl, rsync, tar, gzip, bzip2, pkg-config, zstd
Standards-Version: 4.7.0
Homepage: https://oxcaml.org
Rules-Requires-Root: no

Package: oxcaml-compiler
Architecture: any
Depends: ${misc:Depends}, libc6
</code></pre>
<p>This is then compiled from a source package to an architecture-specific binary one that has a <code>.deb</code> extension.
That's done via <a href="https://tangled.org/anil.recoil.org/oxcaml-pkgs/blob/main/packaging/deb/pbuild.sh">scripts</a> that invoke <a href="https://pbuilder-team.pages.debian.net/pbuilder/">pbuilder</a> in a Docker container for the exact Ubuntu or Debian distro.</p>
<p>I would have used <a href="https://launchpad.net">Launchpad</a> as I used to do for <a href="https://anil.recoil.org/notes/docker-and-opam">opam PPAs</a> back in the day, but it's currently down due to a <a href="https://discourse.ubuntu.com/t/launchpad-still-down-even-though-status-reports-as-up-all-day/81586">DDoS</a> so I'm building these myself for now.</p>
<h2><a href="https://anil.recoil.org/news.xml#fedora-44" class="anchor" aria-hidden="true"></a>Fedora 44</h2>
<pre><code class="language-sh">sudo tee /etc/yum.repos.d/oxcaml.repo &gt;/dev/null &lt;&lt;'EOF'
[oxcaml]
name=OxCaml
baseurl=https://oi.thicket.dev/repo/rpm/fedora-44
enabled=1
gpgcheck=0
repo_gpgcheck=1
gpgkey=https://oi.thicket.dev/repo/rpm/fedora-44/oxcaml.asc
EOF
sudo dnf install oxcaml-compiler
</code></pre>
<p>Fedora's got the <a href="https://docs.fedoraproject.org/en-US/quick-docs/dnf/">DNF</a> package manager, which uses <a href="https://tangled.org/anil.recoil.org/oxcaml-pkgs/blob/main/packaging/rpm/oxcaml-compiler.spec.in">spec files</a> to wrap the build. These source RPMs are then compiled to binary ones via a <a href="https://tangled.org/anil.recoil.org/oxcaml-pkgs/blob/main/packaging/rpm/mockbuild.sh">mock build in a Docker container</a> for that distro.</p>
<p>Once that's done, the repository metadata is assembled using <a href="https://tangled.org/anil.recoil.org/oxcaml-pkgs/blob/main/packaging/repo/mkrepo.sh#L129">createrepo</a>, and a bunch of files that can be served over HTTP.</p>
<h2><a href="https://anil.recoil.org/news.xml#arch-linux" class="anchor" aria-hidden="true"></a>Arch Linux</h2>
<pre><code class="language-sh">curl -fsSL https://oi.thicket.dev/repo/arch/oxcaml.asc | sudo pacman-key --add -
sudo pacman-key --lsign-key &lt;keyid&gt;
sudo tee -a /etc/pacman.conf &gt;/dev/null &lt;&lt;'EOF'

[oxcaml]
SigLevel = Required DatabaseOptional
Server = https://oi.thicket.dev/repo/arch/$arch
EOF
sudo pacman -Sy oxcaml-compiler
</code></pre>
<p>Arch uses <a href="https://tangled.org/anil.recoil.org/oxcaml-pkgs/blob/main/packaging/arch/PKGBUILD.in">PKGBUILD</a> files for its metadata format, which <code>makepkg</code> compiles into a simple <a href="https://tangled.org/anil.recoil.org/oxcaml-pkgs/blob/main/packaging/arch/makepkg/build-pkg.sh"><code>.pkg.tar.zst</code> layer</a> in an <code>archlinux</code> container; <code>repo-add</code> then stitches them into a pacman database.</p>
<h2><a href="https://anil.recoil.org/news.xml#homebrew" class="anchor" aria-hidden="true"></a>Homebrew</h2>
<p>My <a href="https://anil.recoil.org/notes/custom-homebrew-taps">custom Homebrew instructions</a> from yesteryear still work fine, so I just pushed a <a href="https://github.com/avsm/homebrew-ocaml/pull/22/changes">Homebrew OxCaml formula</a> there and let the <a href="https://docs.brew.sh/BrewTestBot">brew bot</a> do its magic.</p>
<p>The release flow here is two stage:</p>
<ul>
<li><code>tests.yml</code> runs <code>brew test-bot</code> on every PR, building the formula and uploading the resulting bottles as CI artifacts.</li>
<li>Then when I manually add the <code>pr-pull</code> label to that PR, <code>publish.yml</code> fires on the <code>labeled</code> event, runs <code>brew pr-pull</code> to download those built bottles, commits them with the formula to <code>main</code>, pushes, and deletes the branch.</li>
</ul>
<p>The only quirk here is to not link it into <code>/opt/homebrew</code> as it would collide with OCaml, so it's marked "keg only" (installed but not symlinked into the prefix).  I think we can integrate the brew bottling directly into <a href="https://github.com/ocurrent/obuilder">obuilder</a> just as soon as we add <a href="https://github.com/ocurrent/obuilder/pull/207">secrets support</a> to the macOS backend.</p>
<h2><a href="https://anil.recoil.org/news.xml#resurrecting-a-2013-gpg-key" class="anchor" aria-hidden="true"></a>Resurrecting a 2013 GPG key</h2>
<p>I did also have to do some GPG shenanigans as my ancient Debian signing key
from 2013 is now rejected because its crypto signature used SHA-1.
Modern <code>gpg</code>, <code>apt</code> and <code>pacman</code> reject SHA-1 certifications outright now
as being too weak, so <code>reprepro</code> refused to trust me anymore.</p>
<p>Upgrading the key gracefully (rather than minting a brand new identity and
losing decades worth of signatures) turned out to be fiddly, and I have
only the haziest memory of modern keyserver etiquette. The last time I
looked at this seriously was chatting to <a href="https://github.com/yminsky">Yaron Minsky</a> about fifteen years
ago about his <a href="https://github.com/yminsky/sks-keyserver">SKS OCaml keyserver</a>!</p>
<p>Anyway, I have a fresh <code>ed25519</code> signing subkey now, so I'll properly
rebuild the web of trust later on. Maybe <a href="https://tangled.org">tangled.org</a>'s
new <a href="https://anil.recoil.org/notes/2026w18">vouching system</a> would be a good place to anchor a PGP web
of trust again.</p>
<h2><a href="https://anil.recoil.org/news.xml#reproducing-the-oxcaml-opam-directives" class="anchor" aria-hidden="true"></a>Reproducing the OxCaml opam directives</h2>
<p>The most fiddly part of all this was reproducing the OxCaml build
<em>exactly</em> as its <a href="https://github.com/oxcaml/opam-repository">opam directives</a> would, but without opam in the loop
and just a single unified tarball where you can do a <code>make &amp;&amp; make install</code>. Distros typically abhor other package managers...</p>
<p>I used a fair amount of <a href="https://anil.recoil.org/notes/aoah-2025">agentic coding</a> here: I pinned the
<code>oxcaml-compiler.5.2.0minus31</code> opam package and had Claude resolve the
patch list and build/install steps into a single shell script, then
verified that the resulting unified patches were byte-identical to what
opam would have assembled.</p>
<p>This is also all fiddly enough that it makes me want to investigate <a href="https://oppi.li/weeklies/2026-19/">package
repositories on ATProto</a> more now...</p><h1>References</h1><ul><li>Madhavapeddy (2025). How to publish custom Homebrew taps for OCaml. <a href="https://doi.org/10.59350/sf0ze-pbf15" target="_blank"><i>10.59350/sf0ze-pbf15</i></a></li></ul>
