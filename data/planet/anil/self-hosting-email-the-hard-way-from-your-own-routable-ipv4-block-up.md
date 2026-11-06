---
title: Self-hosting email the hard way from your own routable IPv4 block up
description: How we refreshed self-hosted Recoil email with our own RIPE-allocated
  IPv4 block, and deployed Postfix/rspamd/Dovecot to get full SPF/DKIM/DMARC deliverability.
url: https://anil.recoil.org/notes/recoil-self-hosting-2026
date: 2026-06-06T00:00:00-00:00
preview_image: https://anil.recoil.org/images/moving-recoil-1.640.webp
authors:
- Anil Madhavapeddy
source:
ignore:
---

<p>Fresh from <a href="https://anil.recoil.org/notes/rewilding-the-web-report">rewilding the web</a>, I updated the Recoil self-hosting
infrastructure that <a href="https://nick.recoil.org">Nick Ludlam</a> and I have run since 1997. Most exciting, this is now
'email the hard way' that includes getting our very own dedicated IPv4 allocation routed thanks
to my buddy <a href="https://github.com/samoht">Thomas Gazagnaire</a> helping out from France!</p>
<p>This post will be quite technical, aimed at those interested in building their own email stack. We'll talk about <a href="https://anil.recoil.org/news.xml#why-run-your-own-email">why bother</a> running your own email; <a href="https://anil.recoil.org/news.xml#email-receipt">receiving email</a> (covering <a href="https://anil.recoil.org/news.xml#ip-reputation-and-denylists">IP reputation</a>, <a href="https://anil.recoil.org/news.xml#getting-our-very-own-ipv4-address-block">getting our own IPv4 allocation</a>, <a href="https://anil.recoil.org/news.xml#stopping-the-zombie-spam-horde">stopping bots</a>, and <a href="https://anil.recoil.org/news.xml#delivering-email-with-lmtp-and-sieve">Sieve delivery</a>); <a href="https://anil.recoil.org/news.xml#sending-email-to-other-people">sending email</a> reliably with <a href="https://anil.recoil.org/news.xml#spf-describes-who-can-send-email-for-a-domain">SPF</a>, <a href="https://anil.recoil.org/news.xml#dkim-cryptographic-signing-of-outbound-mail">DKIM</a>, <a href="https://anil.recoil.org/news.xml#dmarc-ties-it-together-and-provides-reporting-stats">DMARC</a> and <a href="https://anil.recoil.org/news.xml#srs-to-keep-email-forwarding-working">SRS</a>; <a href="https://anil.recoil.org/news.xml#accessing-email-for-users">user access</a> via Dovecot IMAP and Roundcube webmail; and finally <a href="https://anil.recoil.org/news.xml#what-else-is-left-to-do">what's left to do</a> and a <a href="https://anil.recoil.org/news.xml#is-this-a-negative-result-for-self-hosting">reflection</a> on future research ideas.</p>
<h2><a href="https://anil.recoil.org/news.xml#why-run-your-own-email" class="anchor" aria-hidden="true"></a>Why run your own email?</h2>
<p>For someone just getting into systems and networking, it's a hugely educational
experience. Running my own servers has been how I've learnt how the Internet
works, and how I got into open-source back in the day, by <a href="https://anil.recoil.org/notes/openbsd-developer">installing OpenBSD</a> and <a href="https://anil.recoil.org/notes/commit-access-to-php">fixing bugs in PHP</a>!</p>
<p>More broadly, self-hosting is important for sovereign access to your own data
as the <a href="https://doi.org/10.1145/3503158">web steadily consolidates</a> among a few
players. A <a href="https://www.netmeister.org/blog/mx-diversity.html">2023 analysis</a>
showed that two companies can mostly read all of your email traffic:</p>
<blockquote>
<p>So all in all, the answer to the question of who can read your email pretty
much boils down to -- yep -- "Google and Microsoft". Even if your domain
doesn't use one of their mail servers, chances are that <a href="https://mako.cc/copyrighteous/google-has-most-of-my-email-because-it-has-all-of-yours">whoever you are sending mail to does</a>.
<cite>-- <a href="https://www.netmeister.org/blog/mx-diversity.html">MX diversity</a>, <a href="https://mstdn.social/@jschauma">Jan Schaumann</a>, 2023</cite></p>
</blockquote>
<p>Email is right at the centre of our digital lives; consider just how many online
accounts you could reset if your email got <a href="https://dl.acm.org/doi/10.1145/3716489.3728437">hijacked or phished</a>. In many ways, it's <em>the</em> online service that connects up all the other ones.</p>
<h3><a href="https://anil.recoil.org/news.xml#should-you-run-your-own-email" class="anchor" aria-hidden="true"></a>Should you run your own email?</h3>
<p>Hosting your email is a fair bit of work, but that's spread over a long period of time as you keep an eye on it. <a href="https://nick.recoil.org">Nick Ludlam</a> and I started our hosting adventure back in 1998 or so when we <a href="https://anil.recoil.org/notes/netapp-tr-3071-1">worked at NASA</a> for the summer. The first time we did a major server move back in 2002, here's Nick and <a href="https://nanog.org/events/nanog-48/content/2951/">Chris Luke</a>
loading up our second server into a dodgy white minivan to host in Easynet, right outside <a href="https://en.wikipedia.org/wiki/Cyberia,_London">London's first Internet cafe</a>.</p>
<p><img src="https://anil.recoil.org/images/moving-recoil-1.webp" alt="%c" title="From Cyberia outside Fitzrovia over to Brick Lane and Easynet hosting! (2002)"></p>
<p>If you do decide to have a go for yourself, you'll need to find stable Internet hosting (your own home network isn't the best choice,
for reasons we'll see later in this post), and also build up a reputation. Luckily, finding stable Internet is <em>much</em> easier these days than it was in the late 90s, and we use <a href="https://mythic-beasts.com">Mythic Beasts</a> who are excellent, reliable and local.</p>
<p>I'll explain in this post how we obtained our own dedicated IPv4 address block
to help build up a high deliverability index for our personal email that's
entirely independent!  While email looks like one service to most users, it
is actually three separate activities: <a href="https://anil.recoil.org/news.xml#email-receipt">email receipt</a>, <a href="https://anil.recoil.org/news.xml#sending-email-to-other-people">email
submission</a>, and <a href="https://anil.recoil.org/news.xml#accessing-email-for-users">email access</a>.  I'll dive into how each of
these work on Recoil now, in case it's useful for your own setup.</p>
<h2><a href="https://anil.recoil.org/news.xml#email-receipt" class="anchor" aria-hidden="true"></a>Email receipt</h2>
<p>Each domain (like <code>recoil.org</code>) on the Internet runs an SMTP server that allows
it to receive email from other servers. You can query this server for any
domain by looking up its 'MX' DNS record:</p>
<pre><code class="language-bash">$ host -t mx recoil.org
recoil.org mail is handled by 10 pork.recoil.org.
</code></pre>
<p>This indicates that the Internet host <code>pork.recoil.org</code> must accept connections from any server
on the internet that wishes to deliver email to <code>&lt;email&gt;@recoil.org</code>.
The core difficulty is that <a href="https://www.rfc-editor.org/rfc/rfc5321">SMTP</a> (as
designed in the more trusting 1980s) doesn't mandate a built-in proof of trust.
Anyone can claim to be anyone else, and the 'sender' in the email we receive
can be trivially forged.</p>
<p>The IETF's <a href="https://datatracker.ietf.org/doc/html/rfc2505">response</a> was to accrete a stack of checks over the years that an
email sender must pass, or be filtered by the recipient. If we mess these
identity checks up, then our email won't get delivered reliably across the
Internet and the service won't be very useful.</p>
<h3><a href="https://anil.recoil.org/news.xml#ip-reputation-and-denylists" class="anchor" aria-hidden="true"></a>IP reputation and denylists</h3>
<p>Spam was cheap to send from anywhere on the Internet, and so naturally grew as
the wider network gained adoption. Paul Vixie came up with a <a href="https://en.wikipedia.org/wiki/Domain_Name_System_blocklist">DNS-based
blocklist</a> back in
1997. Since then, many more have sprung up, operated by organisations like
<a href="https://www.spamhaus.org/">Spamhaus</a>, <a href="https://www.spamcop.net/">Spamcop</a> and
<a href="https://www.barracudacentral.org/rbl">Barracuda</a> that aggregate reports about
botnets, compromised hosts and spammers into lists that any email server can
consult.</p>
<p>The DNS blacklist/whitelist protocol (<a href="https://www.rfc-editor.org/rfc/rfc5782">RFC 5782</a>) is super simple and can be queried right from your command line:</p>
<pre><code class="language-bash">$ dig 2.0.0.127.zen.spamhaus.org +short  # testing address
127.0.0.2
127.0.0.10
127.0.0.4
</code></pre>
<p>That's a localhost testing address, but the presence of a DNS record in the RBL
means that that server is suspect.  This is where having control of your own
IPv4 address really pays off. Reputation for email via these RBLs accrues
against the address and not the domain. In theory, the IP address you use for
your own self-hosted email <a href="https://doi.org/10.1145/3419394.3423657">might have been re-used</a> from a cloud provider
by someone else, and therefore be tainted by other people's bad behaviour.</p>
<p><em>(Update: <a href="https://ryan.freumh.org">Ryan Gibb</a> observes that as these lists often work on IP blocks, your IP neighbour can also effect your reputation in a multitenant cloud. He reports that he uses <a href="https://www.hetzner.com/">Hetzner</a> for his use who <a href="http://bef.no/DitchingWindowsAndAWS/">manually allowlist</a> SMTP servers to minimise abuse).</em></p>
<h3><a href="https://anil.recoil.org/news.xml#getting-our-very-own-ipv4-address-block" class="anchor" aria-hidden="true"></a>Getting our very own IPv4 address block</h3>
<p>In contrast, a fresh IPv4 starts neutral and earns its reputation through
consistent, well-formed email sending.  The only way to fully control this is
to control the address space itself, which is why Team Recoil now have our very
own IPv4 address block: <code>185.33.27.0/24</code>!</p>
<p>Getting our very own address allocation involved joining a very long queue due to IPv4
exhaustion. The <a href="https://www.ripe.net/">RIPE NCC</a> is the regional registrar for
Europe and ran out of unallocated IPv4 space in <a href="https://www.ripe.net/publications/news/the-ripe-ncc-has-run-out-of-ipv4-addresses/">November
2019</a>,
and since then the only way to get an allocation directly from RIPE is via a
<a href="https://www.ripe.net/manage-ips-and-asns/ipv4/">waiting list</a> for small allocations.</p>
<blockquote>
<p>While we have run out of IPv4 addresses, RIPE NCC members can still request a
single /24 allocation (256 addresses). [...] Requests are added to a waiting
list and processed when we recover IPv4 addresses in the future. [...] This
is only available to LIRs that have never received an IPv4 allocation from
the RIPE NCC before (of any size).
<cite>-- <a href="https://www.ripe.net/manage-ips-and-asns/ipv4/">RIPE NCC, /24 Allocation via the Waiting List</a></cite></p>
</blockquote>
<p>I got into that queue from the UK, and my buddy <a href="https://github.com/samoht">Thomas Gazagnaire</a> did the same from France.
He got to the head of his queue well ahead of me, and we got allocated our
own <code>/24</code> block after about a six month wait.</p>
<h4><a href="https://anil.recoil.org/news.xml#signing-up-to-ripe-for-yourself" class="anchor" aria-hidden="true"></a>Signing up to RIPE for yourself</h4>
<p>If you want to do this yourself and are based in Europe, then pay the annual RIPE NCC membership fee to open a
<a href="https://www.ripe.net/membership/become-a-member/">Local Internet Registry</a>
(LIR) account.</p>
<p>After that, confirm you've never previously received an IPv4 allocation,
join the waiting list, and wait until enough addresses are recovered (e.g. from
defunct LIRs, returned space, or revoked allocations) for a slot to come
up. The wait is currently a year or two I think and seems to depend which country you're in.</p>
<p><a href="https://www.ripe.net/manage-ips-and-asns/ipv4/ipv4-waiting-list/"> <img src="https://anil.recoil.org/images/ripe-ss-2.webp" alt="%c" title="RIPE LIR waiting list statistics"> </a></p>
<h4><a href="https://anil.recoil.org/news.xml#setting-up-ipv4-routes-for-an-autonomous-system" class="anchor" aria-hidden="true"></a>Setting up IPv4 routes for an autonomous system</h4>
<p>The next step was to route this allocation to an actual machine hooked up
to the public Internet. While it is possible to <a href="https://en.wikipedia.org/wiki/Border_Gateway_Protocol">advertise our own routes</a>, in the
interests of expediency we decided to request <a href="http://mythic-beasts.com">Mythic Beasts</a> to take care of it for us and handle the peering.</p>
<p>The procedure to do this via RIPE is straightforward. The IPv4 block is given
an assignment by creating a '<a href="https://en.wikipedia.org/wiki/Resource_Public_Key_Infrastructure">RPKI ROA</a>' in
their database. This is a PKI chain-of-trust used to connect up IP routing blocks on the
Internet.</p>
<p><img src="https://anil.recoil.org/images/ripe-ss-1.webp" alt="%c"></p>
<p>An <a href="https://en.wikipedia.org/wiki/Autonomous_system_(Internet)">Autonomous System</a>
(AS) is a unit of independent routing policy on the Internet and announced to the rest of the world via
<a href="https://www.rfc-editor.org/rfc/rfc4271">BGP</a>.
Working out who actually owns a given ASN turns out to be surprisingly hard as the WHOIS
databases are inconsistently updated, but <a href="https://doi.org/10.1145/3487552.3487853">third-party databases exist</a>
to help.
In our case <a href="https://ipinfo.io/AS44684/185.33.27.0/24">our IP block</a> is connected to the <a href="https://ipinfo.io/AS44684">Mythic Beasts AS44684</a>.</p>
<p>Once that was sorted, RIPE once again uses DNS to announce the connection to Mythic:</p>
<pre><code class="language-bash">$ dig soa -x 185.33.27.0 @pri.authdns.ripe.net +noall +authority
27.33.185.in-addr.arpa.	86400	IN	NS	ns1.mythic-beasts.com.
27.33.185.in-addr.arpa.	86400	IN	NS	ns2.mythic-beasts.com.
</code></pre>
<p>Another nice aspect is being able to control our own reverse DNS to this IP block, which is another
important signal for email reputation:</p>
<pre><code class="language-bash">$ host pork.recoil.org
pork.recoil.org has address 185.33.27.128
pork.recoil.org has IPv6 address 2a00:1098:39c::3
$ host 185.33.27.128
128.27.33.185.in-addr.arpa domain name pointer pork.recoil.org.
</code></pre>
<h3><a href="https://anil.recoil.org/news.xml#stopping-the-zombie-spam-horde" class="anchor" aria-hidden="true"></a>Stopping the Zombie spam horde</h3>
<p>We've so far gone to an enormous amount of hassle to get a clean IPv4 block,
but this is necessary but not sufficient to protect ourselves! We also need to
configure our server to defend itself against the zombie botnet hordes.</p>
<p>The overwhelming majority of incoming TCP connections on port 25
are botnets attempting to deliver spam, probe for open relays, guess
credentials (or more recently) harvest data for AI training.
Spending CPU cycles parsing the body of every one of these requests is both
wasteful and dangerous, so a good setup will try to filter these out as early as
possible.</p>
<p>We first deploy Postfix's <a href="https://www.postfix.org/POSTSCREEN_README.html">postscreen</a>, which
listens on port 25 as the first port of call. Architecturally it's a protocol proxy that accepts the
TCP connection, runs a battery of cheap checks:</p>
<ul>
<li>DNSBL lookups in parallel from multiple providers</li>
<li>adds a deliberate pre-greet pause from <a href="https://www.rfc-editor.org/rfc/rfc5321">RFC 5321 §3.1</a> that catches
bots which start talking before the server's banner appears</li>
<li>a couple of pipelining and non-SMTP-command tests to check for compliance</li>
</ul>
<p>It only hands the connection off to the real <code>smtpd</code> process if the client looks legitimate after
these. Bad clients are dropped during the pre-greet pause with a temporary failure, which
will encourage false positives to retry in the future.
Interestingly this proxy is exactly what <a href="https://anil.recoil.org/notes/cacm-docker-cover">we do in Docker for Desktop</a>, where a userspace <a href="https://anil.recoil.org/notes/icfp25-ocaml5-js-docker">OCaml VPNKit proxy</a> mediates between containers and the host network without exposing the host stack directly. I'm going to reimplement postscreen in OxCaml soon...</p>
<p>For those configuring your own server, the relevant Postfix knobs are in <code>main.cf</code>:</p>
<pre><code class="language-ini">postscreen_dnsbl_sites   = zen.spamhaus.org*3 bl.spamcop.net*2 b.barracudacentral.org*2
postscreen_dnsbl_action  = enforce        # reject when DNSBL score crosses threshold
postscreen_greet_action  = enforce        # reject pre-greet slammers
</code></pre>
<p>The <code>*3</code> and <code>*2</code> weights let us combine blocklists rather than trust any single
source; in the above we trust Spamhaus alone and the two weaker lists need to both agree to
reject.
Just postscreen by itself seems to reject over 90% of incoming spam requests, but there's
another trick we can apply.</p>
<p>One minor wrinkle that <a href="https://nick.recoil.org">Nick Ludlam</a> noticed is that Apple's iCloud email service interacts
badly with postscreen. Apple's outbound MX pool that delivers email doesn't
retry from the same IP, which means postscreen's allowlist for that connection
never matches and mail can stay in limbo for hours. This isn't really a
misconfiguration on our end, but it affect users badly.</p>
<p>The fix (since Apple still owns the whole of <code>17.0.0.0/8</code>!) is to allowlist that whole range up front in
<code>postscreen_access.cidr</code> so it bypasses the protocol tests entirely. The
<a href="https://docs.mailcow.email/manual-guides/Postfix/u_e-postfix-postscreen_whitelist/">mailcow allowlist guide</a>
walked me through the syntax in case you hit the same problem:</p>
<pre><code class="language-bash"># /etc/postfix/postscreen_access.cidr
17.0.0.0/8    permit       # Apple iCloud MX pool is somewhere in here
</code></pre>
<h4><a href="https://anil.recoil.org/news.xml#greylisting" class="anchor" aria-hidden="true"></a>Greylisting</h4>
<p>Once a connection survives postscreen, our next defensive layer is
greylisting (<a href="https://www.rfc-editor.org/rfc/rfc6647">RFC 6647</a>),
implemented for us by <a href="https://rspamd.com/">rspamd</a>. The idea is that
the first time we ever see a particular source, our server returns a temporary failure rather
than accepting the message, and records the source for future reference.</p>
<p>Legitimate MTAs are required by <a href="https://www.rfc-editor.org/rfc/rfc5321">RFC 5321</a> to queue/retry after a
few minutes. When the retry comes in, we'll have seen it from the first attempt
and then let the message through.  A large fraction of botnets are
"single-shot" delivery engines that just move on to the next victim when they
get a failure, since maintaining a retry queue is expensive when you're
shotgunning millions of messages.  The cost to a <em>real</em> sender is a one-time
delay of a few minutes, which is amortised across all users from that source
(the greylisting only happens to the first sender).</p>
<p>All these mechanics are implemented in rspamd, which itself has a nice local connection: its author
<a href="https://github.com/vstakhov">Vsevolod Stakhov</a> did his PhD in the CL with Jon Crowcroft and me, and I spent
much time back in 2015 working with him with <a href="https://www.highsecure.ru/httpcrypt.pdf">HTTPCrypt</a>, a scheme for
opportunistic HTTP encryption that uses NaCl-style cryptography to skip the full TLS
handshake by passing the server public key out-of-band (typically via DNS).
The paper got soundly <a href="https://www.usenix.org/conference/usenixsecurity15">rejected from USENIX Security 2015</a>
for reasons I can't remember but weren't very important, but the
protocol lives on as the <a href="https://docs.rspamd.com/developers/encryption/">encryption layer between rspamd and its clients</a>!</p>
<p>Ok, so now by the time a message has survived postscreen and greylisting, well over 99%
of the original bot traffic has been turned away at the door for a tiny
fraction of the CPU cost of actually reading it. After this, we still need
to do a bit of work scanning the content itself.</p>
<h4><a href="https://anil.recoil.org/news.xml#milter-clamav-and-bayesian-filtering" class="anchor" aria-hidden="true"></a>Milter, ClamAV and Bayesian filtering</h4>
<p>Everything that makes it past postscreen and greylisting gets handed to rspamd
over the <a href="https://www.postfix.org/MILTER_README.html">milter</a> protocol.
Our postfix is configured to consult rspamd for every message,
which allows rspamd to either hard reject or defer a delivery while the
sending server is still 'on the line'</p>
<p>This allows dodgy mail to be refused at the source rather than accepted for delivery
and then bounced afterwards (which is unreliable, as the bouncing address may also
be fake and cause <a href="https://en.wikipedia.org/wiki/Backscatter_(email)">backscatter spam</a> to be generated by us!).</p>
<p>The Postfix configuration for this milter is as easy as running the rspamd
daemon configured to listen on localhost.</p>
<pre><code class="language-ini"># /etc/postfix/main.cf
smtpd_milters        = inet:localhost:11332
</code></pre>
<p><a href="https://www.clamav.net/"> <img src="https://anil.recoil.org/images/clamav-logo.webp" alt="%rc"> </a>
The first port of call is to filter out messages with known virus attachments.
<a href="https://www.clamav.net/">ClamAV</a> is what we use for this, and it maintains
its own <a href="https://docs.clamav.net/manual/Signatures.html">virus signature</a> database.
rspamd hands the message body to the local <code>clamd</code> daemon over a Unix
socket, and rejects outright on a virus hit so the message is never delivered.</p>
<pre><code class="language-ini"># /etc/rspamd/local.d/antivirus.conf
clamav { type = "clamav"; servers = "/run/clamav/clamd.ctl"; action = "reject"; }
</code></pre>
<p>The other main filter (among many) is rspamd's "<a href="https://docs.rspamd.com/configuration/statistic/">Bayesian
classifier</a>" that scores each
incoming message against a dynamic corpus of known <a href="https://en.wikipedia.org/wiki/Anti-spam_techniques">spam and
ham</a> messages stored in
<a href="https://redis.io/">Redis</a>. The classifier auto-learns from messages that score
extremely high or low, but it can also be personalised by the Recoil users by
keeping an eye on each user's Junk folder and adding those to the classifier.</p>
<p>These messages are learnt by being piped to the <code>rspamc</code> command, which can
learn both ham and spam on stdin.  Over a few weeks of doing this on every
false-positive (or false-negative), the classifier gets pretty good
at matching what our users want to see without having to maintain a static database.</p>
<pre><code class="language-bash">$ rspamc stat
Results for command: stat (0.028 seconds)
Messages scanned: 6206
Messages with action reject: 135, 2.18%
Messages with action soft reject: 0, 0.00%
Messages with action rewrite subject: 0, 0.00%
Messages with action add header: 137, 2.21%
Messages with action greylist: 658, 10.60%
Messages with action no action: 5276, 85.01%
Messages treated as spam: 272, 4.38%
Messages treated as ham: 5934, 95.62%
</code></pre>
<h3><a href="https://anil.recoil.org/news.xml#delivering-email-with-lmtp-and-sieve" class="anchor" aria-hidden="true"></a>Delivering email with LMTP and Sieve</h3>
<p>Once our intrepid email message has run the gauntlet of postscreen,
greylisting, rspamd, ClamAV and Bayesian classifiers, we <em>finally</em> get to
actually send it to the right user.</p>
<p>By default Postfix would just write the file straight into the user's home
directory, but that's not much use in the modern world where the volume of
email most people receive means that we'd like to file them into folders.</p>
<p>We therefore hand the message over to <a href="https://www.dovecot.org/">Dovecot</a> via
<a href="https://www.rfc-editor.org/rfc/rfc2033">LMTP</a>, which is basically SMTP (the protocol
we used to receive the email from the outside world), but without the queueing complexity.
This handoff happens over a normal Unix domain socket that's inside Postfix's
queue directory:</p>
<pre><code class="language-ini"># /etc/postfix/main.cf
virtual_alias_domains = recoil.org
virtual_alias_maps    = hash:/etc/postfix/recoil.org
mailbox_transport     = lmtp:unix:private/dovecot-lmtp
</code></pre>
<p><code>virtual_alias_maps</code> turns an arbitrary <code>anything@recoil.org</code> address into an
address that can be delivered locally (e.g. <code>anil@recoil.org</code> goes to <code>avsm@pork.recoil.org</code>).
The reason for handing off to Dovecot rather than letting Postfix write the maildir directly
is that Dovecot then owns the local user operations of indexing, quotas, full-text search and Sieve
filtering. We'll come back to this in the <a href="https://anil.recoil.org/news.xml#accessing-email-for-users">email retrieval</a> section.</p>
<h4><a href="https://anil.recoil.org/news.xml#durable-on-disk-storage-with-maildir" class="anchor" aria-hidden="true"></a>Durable on-disk storage with Maildir</h4>
<p>Our storage format on disk is the reliable old <a href="https://en.wikipedia.org/wiki/Maildir">Maildir</a>
format, which stores each email as a single file under each user's <code>~/Maildir</code>. It's a format
that we've been using on Recoil since 1998, when we first used <a href="http://qmail.org/man/man5/maildir.html">qmail</a> as our mail server.
The reason I like it so much is that <a href="https://github.com/avsm/maildir-eio">processing libraries</a> are
trivial to write, so email is never locked up in a proprietary format or database over the march
of decades.</p>
<p>The format itself is minimal. Each <code>~/Maildir</code> is just three
subdirectories <code>tmp/</code>, <code>new/</code> and <code>cur/</code>.</p>
<pre><code class="language-bash">/home/avsm/Maildir/.archive.2018
/home/avsm/Maildir/.archive.2018/tmp
/home/avsm/Maildir/.archive.2018/cur
/home/avsm/Maildir/.archive.2018/new
/home/avsm/Maildir/.github.mention
/home/avsm/Maildir/.github.mention/cur
/home/avsm/Maildir/.github.mention/new
/home/avsm/Maildir/.github.mention/tmp
...
</code></pre>
<p>The concurrency story is much simpler as <a href="https://en.wikipedia.org/wiki/Rename_(computing)">POSIX guarantees local atomicity</a>
of file rename operations. An incoming message
is first written to a unique file in <code>tmp/</code>, then once fully written is (atomically) renamed
into <code>new/</code>.</p>
<p>A client reading the Maildir then moves files from <code>new/</code> into <code>cur/</code> once
they've been seen. The message file itself is immutable, and so clients use the
filename to store message information by appending a flag suffix (e.g. <code>:2,S</code>
for Seen, <code>:2,SR</code> for Seen and Replied).  With this setup, no user level mailbox-wide locking is needed and so
Postfix can deliver a new message without synchronisation while normal email
reading is going on.</p>
<h4><a href="https://anil.recoil.org/news.xml#indexing-the-email-with-flatcurve" class="anchor" aria-hidden="true"></a>Indexing the email with Flatcurve</h4>
<p>I did evaluate some much more modern email servers like <a href="https://stalw.art/">Stalwart</a>
which have tons of cool features, but ultimately decided against switching
because they don't support Maildir. They instead require stashing email in a
custom database format (e.g. in RocksDB) which (I think) mixes up the durability
of email with having fast search.
Instead, I took advantage of Dovecot support for full text indexing via a
<em>separate</em> index, which is the
<a href="https://github.com/slusarz/dovecot-fts-flatcurve">Flatcurve</a> full-text index.</p>
<p>Flatcurve is actually a wrapper around the venerable <a href="https://xapian.org/">Xapian</a>,
which (like rspamd) is yet another locally developed Cambridge technology!
<a href="https://en.wikipedia.org/wiki/Martin_Porter">Martin Porter</a> (inventor of the famous
<a href="https://en.wikipedia.org/wiki/Stemming">stemming algorithm</a>), did the Computer Science
Diploma in the CL in 1967 and released the first version of Xapian in 2002. Note that
there's no connection to our <a href="https://anil.recoil.org/papers/2010-icfp-xen">OCaml Xen XAPI toolstack</a> which was
developed by us independently in 2004!</p>
<p>In our setup, Flatcurve keeps a separate Xapian index per mailbox under
<code>~/Maildir/fts-flatcurve</code> and updates it automatically as new mail arrives.
The Dovecot side of the config is just:</p>
<pre><code class="language-ini"># /etc/dovecot/conf.d/90-fts-flatcurve.conf
mail_plugins { fts = yes; fts_flatcurve = yes }
fts_autoindex = yes
</code></pre>
<p>Flatcurve also ships CLI tools as part of the Dovecot plugin to mess around
with the index or do CLI searches (handy for agentic search!):</p>
<pre><code class="language-bash"> $ doveadm fts flatcurve stats -u avsm INBOX
INBOX guid=436177290d20ee4e664a00007b9d9320 last_uid=1018743
messages=83485 shards=2 version=1
</code></pre>
<h4><a href="https://anil.recoil.org/news.xml#the-sieve-language-for-server-side-filtering" class="anchor" aria-hidden="true"></a>The Sieve language for server-side filtering</h4>
<p>Before the message lands in the maildir, we also need to decide which folder
to put it into, or do other preprocessing like labeling. Since this can get
arbitrarily complicated based on the user's needs, we use a
<a href="https://pigeonhole.dovecot.org/">Pigeonhole Sieve</a> plugin that runs user-defined
delivery-time filters.</p>
<p>Sieve (<a href="https://www.rfc-editor.org/rfc/rfc5228">RFC 5228</a>) is a declarative language designed specifically for mail
filtering.  There's a system-wide script that runs first and
files anything rspamd has flagged into <code>Junk</code>, and then each user's personal
script runs:</p>
<pre><code class="language-ini"># /etc/dovecot/conf.d/90-sieve.conf
sieve_script spam-to-junk { type = before; path = /etc/dovecot/sieve/spam-to-junk.sieve }
sieve_script personal     { path = ~/sieve; active_path = ~/.dovecot.sieve }
</code></pre>
<pre><code class="language-sieve"># /etc/dovecot/sieve/spam-to-junk.sieve
require ["fileinto", "mailbox"];
if header :contains "X-Spam" "Yes" {
  fileinto :create "Junk";
  stop;
}
</code></pre>
<p>Running these rules at delivery time on the server
means the same rules apply whether I'm eventually reading mail on my laptop, phone, or
via webmail. I can also write Sieve rules for very custom vacation rules, email
priorities, and coding agents like Claude can easily figure out the DSL intricacies.
I found the <a href="https://gist.github.com/Hotrod369/6b7a24e1ea060e48e0c02459cbb950a0">Sieve cheatsheet</a> very
useful here: you can do things like <a href="https://gist.github.com/Hotrod369/6b7a24e1ea060e48e0c02459cbb950a0#complex-sieve-script-examples">modify messages</a>, create dynamic folders and <a href="https://doc.dovecot.org/2.3/configuration_manual/sieve/extensions/editheader/">edit headers</a>.</p>
<p>There's also a "ManageSieve" (<a href="https://www.rfc-editor.org/rfc/rfc5804">RFC 5804</a>) daemon running,
which lets a mail client edit a user's Sieve script remotely without needing shell access.
I got this working with both <a href="https://addons.thunderbird.net/en-US/thunderbird/addon/sieve/">Thunderbird</a>
and <a href="https://plugins.roundcube.net/packages/kolab/managesieve">Roundcube</a> which bundles a plugin natively.</p>
<p>My email filter's massive, but I generate it from OCaml code that outputs something like:</p>
<pre><code class="language-sieve">if header :contains "List-Id" "caml-list.inria.fr"
{
        fileinto "dev.caml-list";
        stop;
}
if header :contains "List-Id" "types-list.LISTS.SEAS.UPENN.EDU"
{
        fileinto "lists.types";
        stop;
}
&lt;...&gt;
</code></pre>
<p>The Junk Bayesian training loop from earlier piggybacks on this too, as a Sieve
IMAP event
(<a href="https://www.rfc-editor.org/rfc/rfc6785">RFC 6785</a>) script fires on each
folder move, which then pipes the message to <code>rspamc learn_spam</code> or <code>learn_ham</code>.
This might all feel like a Rube Goldberg machine, but each component does have
its own specialised role.</p>
<h2><a href="https://anil.recoil.org/news.xml#sending-email-to-other-people" class="anchor" aria-hidden="true"></a>Sending email to other people</h2>
<p>We've so far put a stupid amount of effort into <em>receiving</em> email safely, but
this wouldn't be much use if we also can't reliably <em>send</em> email that won't
get rejected.
Failing any one of the gauntlet of checks by the hyperscalers will send our mail to
someone's spam folder or be quietly dropped. There are three separate protocols
that work together to avoid this unfortunate outcome: SPF, DKIM and DMARC.</p>
<h3><a href="https://anil.recoil.org/news.xml#spf-describes-who-can-send-email-for-a-domain" class="anchor" aria-hidden="true"></a>SPF describes who can send email for a domain</h3>
<p>The <a href="https://www.rfc-editor.org/rfc/rfc7208">SPF</a> (Sender Policy Framework) protocol
is a DNS TXT record at the apex of our email domain which declares which IP addresses are
allowed to <em>originate</em> mail claiming to be from <code>@recoil.org</code>. As before, we can
query the live record for Recoil from the CLI:</p>
<pre><code class="language-bash">$ dig +short TXT recoil.org
"v=spf1 a mx -all"
"Llamaz United"
$ dig mx recoil.org +short
10 pork.recoil.org.
</code></pre>
<p>The first one is the SPF record, and the second is a random record I created in
1999 to test TXT records. The SPF entry says that the only valid senders for
<code>recoil.org</code> are whatever hosts the <code>MX</code> records of <code>recoil.org</code> point to. In
our case this is just <code>pork.recoil.org</code>, and everything else is expected
to be illegitimate email not authorized by us.</p>
<p>It's possible to also declare a softer <code>~all</code> softfail that lets receivers accept dubious mail with a
warning. This setup is safe for Recoil because we never send legitimate mail from anywhere except our
own mail server.</p>
<p>One little footgun is that we have lots of IP addresses bound to the
pork.recoil.org host (because of the aforementioned /24 IPv4 block that we were
allocated), and so the Postfix daemon needs to be bound specifically to one
address to ensure that all of the outbound TCP connections it makes are indeed
from the MX entry and not another address from our pool.</p>
<pre><code class="language-ini"># /etc/postfix/main.cf
smtp_bind_address    = 185.33.27.128
smtp_bind_address6   = 2a00:1098:39c::3
</code></pre>
<p>Without these, Postfix would happily send out from whatever address the kernel
decides to use, which might be the IPv4 we use for the webmail or some other
service entirely, and the receiver's SPF check would fail...</p>
<h3><a href="https://anil.recoil.org/news.xml#dkim-cryptographic-signing-of-outbound-mail" class="anchor" aria-hidden="true"></a>DKIM: cryptographic signing of outbound mail</h3>
<p>SPF only authenticates the connecting IP of the email server to other people, but
says nothing about the contents of the message itself. We could still use a way
to authenticate that a full message has been certified by Recoil as originating
from us, even if that message has been through several other email relays (e.g. by
being forwarded).</p>
<p>The <a href="https://www.rfc-editor.org/rfc/rfc6376">DKIM</a> (DomainKeys Identified Mail)
protocol adds a per-message cryptographic signature, in a <code>DKIM-Signature:</code>
email header.  This covers a canonicalised form of the body and selected headers,
to permit some flexibility in rearranging the email but still be robust against
tampering. The public verification keys for DKIM live in the public DNS and so
are available for any receiver to easily check.</p>
<p>The job of signing every single message leaving Recoil is <a href="https://docs.rspamd.com/modules/dkim_signing/">rspamd's job</a>. Our
DKIM private key lives on disk and never leaves the server. rspamd
adds the signature to every outbound message during the same milter pass that
<a href="https://anil.recoil.org/news.xml#milter-clamav-and-bayesian-filtering">scores inbound mail</a>:</p>
<pre><code># /etc/rspamd/local.d/dkim_signing.conf
domain {
  recoil.org    { selector = "mail"; path = "/var/lib/rspamd/dkim/recoil.org.mail.key" }
}
</code></pre>
<p>Since you don't want to have the same private key in use forever, DKIM
supports rotation via "selectors" (<code>mail</code> in our case). This lets us rotate keys by
publishing a new public key under a new selector while keeping the old one
live, so signatures on already-sent mail still verify. The public side
lives in DNS at <code>&lt;selector&gt;._domainkey.&lt;domain&gt;</code>:</p>
<pre><code>$ dig +short TXT mail._domainkey.recoil.org
"v=DKIM1; k=rsa; p=MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA..."
</code></pre>
<p>One gotcha if you're setting this up yourself is that DKIM TXT records
frequently exceed the <a href="https://www.rfc-editor.org/rfc/rfc7208">255-byte string
limit</a> and have to be split into
multiple quoted strings inside one record. Most authoritative DNS providers
will do that for you, but you may need to mess around in their various web UIs to figure out how.</p>
<p>Another oddity about DKIM is that it's really intended for live verification. If you want
to go back and re-verify some emails that are (say) a few years old, then the DNS keys
will have expired and so you won't be able to do so. If anyone knows of any DKIM
<a href="https://certificate.transparency.dev/">transparency logs</a> like exist for TLS, I'd love
to try it to go back over my historical email and do some data mining.</p>
<h3><a href="https://anil.recoil.org/news.xml#dmarc-ties-it-together-and-provides-reporting-stats" class="anchor" aria-hidden="true"></a>DMARC ties it together and provides reporting stats</h3>
<p>Neither SPF nor DKIM actually check the <code>From:</code> header that
decides what our users actually see in their mail client:</p>
<ul>
<li>SPF only authenticates the envelope sender (the <code>MAIL FROM</code> SMTP command)</li>
<li>DKIM only authenticates the sending domain in its own <code>d=</code> tag.</li>
</ul>
<p>Without some glue, a spammer could pass the SPF check for their own <code>evil.example.com</code>,
then sign a message with a valid DKIM key for <code>evil.example.com</code>, and
<em>still</em> write <code>From: anil@recoil.org</code> in the message that the user eventually reads.
The protocol glue to prevent this is <a href="https://www.rfc-editor.org/rfc/rfc7489">DMARC</a>, which checks
that the domains authenticated by SPF and DKIM actually match the visible
<code>From:</code> in the email message, and also tells receivers what to do when the check fails.</p>
<p>As you might have guessed by now, this involved yet another DNS record:</p>
<pre><code>$ dig +short TXT _dmarc.recoil.org
"v=DMARC1; p=quarantine; rua=mailto:postmaster@pork.recoil.org"
</code></pre>
<p>The strongest policy is <code>p=reject</code>, but we're going for a softer 'quarantine'
until I'm comfortable with the setup for a few more months.
A <em>really</em> useful part for actually debugging deliverability (given how many
third parties are involved here) is <code>rua=</code>, which is the email address for
a regular aggregate report.</p>
<p>Once a day or so, every major receiver who gets email from Recoil (including Google,
Microsoft, Yahoo, Fastmail, and some smaller ones) sends an XML report to
this address summarising the messages they saw claiming to be from
<code>recoil.org</code>.
Some of these actually look like valid fails; for example this one from Yahoo
seems to indicate that we've had some email sent <em>not</em> from our servers:</p>
<pre><code class="language-xml">$ gzcat yahoo.co.uk!recoil.org!1780617600!1780703999.xml.gz
&lt;?xml version="1.0"?&gt;
&lt;feedback&gt;
  &lt;report_metadata&gt;
    &lt;org_name&gt;Yahoo&lt;/org_name&gt;
    &lt;email&gt;dmarchelp@yahooinc.com&lt;/email&gt;
    &lt;report_id&gt;1780732274.742817&lt;/report_id&gt;
    &lt;date_range&gt;
      &lt;begin&gt;1780617600&lt;/begin&gt;
      &lt;end&gt;1780703999&lt;/end&gt;
    &lt;/date_range&gt;
  &lt;/report_metadata&gt;
  &lt;policy_published&gt;
    &lt;domain&gt;recoil.org&lt;/domain&gt;
    &lt;adkim&gt;r&lt;/adkim&gt;
    &lt;aspf&gt;r&lt;/aspf&gt;
    &lt;p&gt;quarantine&lt;/p&gt;
    &lt;pct&gt;100&lt;/pct&gt;
  &lt;/policy_published&gt;
  &lt;record&gt;
    &lt;row&gt;
      &lt;source_ip&gt;192.134.164.83&lt;/source_ip&gt;
      &lt;count&gt;4&lt;/count&gt;
      &lt;policy_evaluated&gt;
        &lt;disposition&gt;quarantine&lt;/disposition&gt;
        &lt;dkim&gt;fail&lt;/dkim&gt;
        &lt;spf&gt;fail&lt;/spf&gt;
      &lt;/policy_evaluated&gt;
    &lt;/row&gt;
    &lt;identifiers&gt;
      &lt;header_from&gt;recoil.org&lt;/header_from&gt;
    &lt;/identifiers&gt;
    &lt;auth_results&gt;
      &lt;dkim&gt;
        &lt;domain&gt;inria.fr&lt;/domain&gt;
        &lt;selector&gt;dc&lt;/selector&gt;
        &lt;result&gt;pass&lt;/result&gt;
      &lt;/dkim&gt;
      &lt;spf&gt;
        &lt;domain&gt;inria.fr&lt;/domain&gt;
        &lt;result&gt;pass&lt;/result&gt;
      &lt;/spf&gt;
    &lt;/auth_results&gt;
  &lt;/record&gt;
&lt;/feedback&gt;
</code></pre>
<p>A little bit of sleuthing on that IP shows that:</p>
<pre><code>$ host 192.134.164.83
83.164.134.192.in-addr.arpa domain name pointer mail2-relais-roc.national.inria.fr
</code></pre>
<p>...it's the INRIA email server, which probably means that I sent an
email to 'caml-devel@inria.fr' from 'anil@recoil.org', which proceeded to forward
that to a recipient hosted on a '@yahoo.com' email address which failed
verification since it hadn't come straight from recoil. Note that both
SPF failed (to be expected since the INRIA server would have sent the email)
but <em>also</em> DKIM failed (since the INRIA server probably rewrote some mail headers).</p>
<p>This is all quite complex sounding (and it is), but it is invaluable to help
debug the distributed system over time.
I run a quick OCaml script over the various emails that are coming in, and steadily
telling our users when I need to reconfigure one of their clients.
DMARC reporting itself has had some security implications. A <a href="https://www.usenix.org/conference/usenixsecurity23/presentation/ashiq">2023 study</a> demonstrated that a single attacker email can be
turned into a flood of DMARC reports. The fix was to lock down the acceptable
<code>rua</code> addresses to the domain itself, so it doesn't apply to our self hosting setup.</p>
<h3><a href="https://anil.recoil.org/news.xml#srs-to-keep-email-forwarding-working" class="anchor" aria-hidden="true"></a>SRS to keep email forwarding working</h3>
<p>There's one painful corner case that I identified above that I haven't quite sorted
yet: mailing lists. This is why we're still in 'quarantine' mode for our Recoil setup.
If someone emails <code>anil@recoil.org</code> and my server forwards it on to (say) my
Cambridge address, the original sender's domain is now being sent to the
destination from our IP, which will fail their SPF check.</p>
<p>The fix is the <a href="https://www.open-spf.org/SRS/">Sender Rewriting Scheme</a> (SRS),
implemented by <a href="https://github.com/roehling/postsrsd">postsrsd</a>. Using this, we rewrite
the envelope sender on the way out from <code>original@example.com</code> to
something like <code>SRS0=…=example.com=original@recoil.org</code>, so that SPF checks at the
destination evaluates against our domain. We also reverse the rewrite on the way back for any bounces.</p>
<p>SRS doesn't seem to have an IETF RFC that I can find, but it does let some
forwarding paths survive in this DMARC-enforced world. I'm still figuring out exactly how it all
works in our especially complex Cambridge email setup (which involves many hoops and
forwarding layers), but this is all it takes in the Postfix setup for now:</p>
<pre><code class="language-ini"># /etc/postfix/main.cf
sender_canonical_maps    = socketmap:unix:srs:forward
recipient_canonical_maps = socketmap:unix:srs:reverse
</code></pre>
<p>Phew, so with SPF, DKIM, DMARC and SRS wired up, our deliverability index against
Gmail and Outlook seems reliable. Not one of our (loudly complaining) families has
complained about spam since we switched to this setup. Hurrah!</p>
<h2><a href="https://anil.recoil.org/news.xml#accessing-email-for-users" class="anchor" aria-hidden="true"></a>Accessing email for users</h2>
<p>Now that we can both send and receive email, all that's left is for users to be
able to access it easily!  On Recoil it comes down to two paths: a regular IMAP
client (e.g. Mail.app on macOS) talking to our Dovecot server, or via a
web browser pointing at our self-hosted webmail (which itself acts as an IMAP
client).</p>
<h3><a href="https://anil.recoil.org/news.xml#dovecot-and-imap" class="anchor" aria-hidden="true"></a>Dovecot and IMAP</h3>
<p><a href="https://www.dovecot.org/">Dovecot</a> handles all the mailbox access on <code>pork</code>,
encrypting listeners with TLS (<a href="https://www.rfc-editor.org/rfc/rfc8314">RFC 8314</a>).
All our ports require TLS so no plaintext mail or passwords ever cross the
public network. We use <a href="https://letsencrypt.org/">LetsEncrypt</a> for this, with
multiple host aliases (like <code>imap.recoil.org</code> or <code>smtp.recoil.org</code>) served via
<a href="https://en.wikipedia.org/wiki/Server_Name_Indication">SNI</a> so that our users
who last configured their phones in 2008 don't have to touch anything:</p>
<pre><code class="language-ini"># /etc/dovecot/conf.d/11-ssl-imap.conf
local_name imap.recoil.org {
  ssl_server_cert_file = /etc/letsencrypt/live/imap.recoil.org/fullchain.pem
  ssl_server_key_file  = /etc/letsencrypt/live/imap.recoil.org/privkey.pem
}
</code></pre>
<p>Dovecot also pulls double duty as Postfix's SASL backend (<a href="https://www.rfc-editor.org/rfc/rfc4422">RFC
4422</a>) for outbound submission. This
allows users to have the same password for IMAP (to access their email) and
SMTP (to send email).</p>
<h3><a href="https://anil.recoil.org/news.xml#roundcube-webmail" class="anchor" aria-hidden="true"></a>Roundcube webmail</h3>
<p>I used to work on <a href="https://anil.recoil.org/notes/horde-developer">Horde IMP back in the 2000s</a>, and so
I did try to get my beloved <a href="https://github.com/horde/imp/">IMP webmail</a> running again. However, it looks
like it's between release cycles right now and things are in flux,
so I switched over to running
<a href="https://roundcube.net/">Roundcube</a> behind a
<a href="https://caddyserver.com/">Caddy</a> TLS reverse proxy, all packaged together
as a Docker Compose service.</p>
<p>Roundcube is configured to connect to
<code>pork</code> over the same TLS/IMAP as any other client would:</p>
<pre><code class="language-yaml">services:
  roundcube:
    image: roundcube/roundcubemail
    environment:
      ROUNDCUBEMAIL_DEFAULT_HOST: ssl://pork.recoil.org
      ROUNDCUBEMAIL_SMTP_SERVER:  tls://pork.recoil.org
      ROUNDCUBEMAIL_PLUGINS: "managesieve,markasjunk,archive"
  caddy:
    image: caddy:latest
</code></pre>
<p>The Roundcube plugins I'm using are:</p>
<ul>
<li><a href="https://plugins.roundcube.net/packages/kolab/managesieve"><code>managesieve</code></a>
that speaks the <a href="https://www.rfc-editor.org/rfc/rfc5804">ManageSieve</a> protocol
to allow editing a Sieve filter in the browser.</li>
<li><a href="https://plugins.roundcube.net/packages/johndoh/markasjunk"><code>markasjunk</code></a> translates the "Junk button" in the webmail into a move to the Junk folder that causes the ham/spam classification to function invisibly to the user.</li>
</ul>
<p><img src="https://anil.recoil.org/images/roundcube-ss-filter.webp" alt="%c" title="The Roundcube ManageSieve UI doesn't expose the raw Sieve DSL, so it's easier to use"></p>
<h2><a href="https://anil.recoil.org/news.xml#what-else-is-left-to-do" class="anchor" aria-hidden="true"></a>What else is left to do?</h2>
<p>This setup has been pretty solid for day-to-day use in the past few weeks, but
there is (always) more work to do.</p>
<p>As a recap, here's the list of DNS records <code>recoil.org</code> publishes
to make everything work:</p>
<div role="region"><table>
<tbody><tr>
<th>Record</th>
<th>Purpose</th>
<th>Reference</th>
</tr>
<tr>
<td><code>MX (pork.recoil.org)</code></td>
<td>Where mail for the domain is delivered</td>
<td><a href="https://www.rfc-editor.org/rfc/rfc5321">RFC 5321 §5</a></td>
</tr>
<tr>
<td><code>A</code> / <code>AAAA</code></td>
<td>pork's IP addresses</td>
<td><a href="https://www.rfc-editor.org/rfc/rfc1035">RFC 1035</a></td>
</tr>
<tr>
<td><code>PTR</code> (rDNS)</td>
<td>IP to <code>pork.recoil.org</code> reverse mapping</td>
<td><a href="https://www.rfc-editor.org/rfc/rfc1912">RFC 1912 §2.1</a></td>
</tr>
<tr>
<td><code>TXT</code> SPF (<code>v=spf1 mx -all</code>)</td>
<td>Which hosts may send for the domain</td>
<td><a href="https://www.rfc-editor.org/rfc/rfc7208">RFC 7208</a></td>
</tr>
<tr>
<td><code>TXT</code> DKIM (<code>mail._domainkey</code>)</td>
<td>Public key for signature verification</td>
<td><a href="https://www.rfc-editor.org/rfc/rfc6376">RFC 6376</a></td>
</tr>
<tr>
<td><code>TXT</code> DMARC (<code>_dmarc</code>)</td>
<td>Policy and reporting for SPF/DKIM alignment</td>
<td><a href="https://www.rfc-editor.org/rfc/rfc7489">RFC 7489</a></td>
</tr>
</tbody></table></div><h3><a href="https://anil.recoil.org/news.xml#modern-transport-security-mta-sts-dane-and-dnssec" class="anchor" aria-hidden="true"></a>Modern transport security: MTA-STS, DANE and DNSSEC</h3>
<p><a href="https://www.rfc-editor.org/rfc/rfc8461">MTA-STS</a> tells other
mail servers they should only talk to us over TLS with a valid
certificate.  This mitigates the <a href="https://nostarttls.secvuln.info/">STARTTLS-downgrade attack</a>
whereby an attacker strips the TLS upgrade from the SMTP session. It also
helps that email between servers is guaranteed to be TLS encrypted so that
casual network snooping can no longer read emails.</p>
<p><a href="https://www.rfc-editor.org/rfc/rfc7672">DANE/TLSA</a> adds support for
DNS-pinned TLS certificate hashes, rather than using HTTPS for this. The
delay in deploying this is that DANE requires the DNS zone to be DNSSEC-signed, which <code>recoil.org</code>
isn't yet. Moving a domain to DNSSEC requires understanding a lot more about
key rotation than I have time for right now, but it's getting higher up on my
TODO list!</p>
<p><a href="https://www.open-spf.org/SRS/">SRS</a> is semi-deployed right now,
but I haven't tested it against every forwarding path that
exists in our setup. In particular, the INRIA failure is a bit worrying as
it triggers a DMARC failure (and hence might affect our domain reputation), but
involves an email server out of my immediate control.</p>
<h3><a href="https://anil.recoil.org/news.xml#a-jmap-proxy-in-ocaml" class="anchor" aria-hidden="true"></a>A JMAP proxy in OCaml?</h3>
<p>I'd also like to expose email access via
<a href="https://www.rfc-editor.org/rfc/rfc8620">JMAP</a> (the JSON Mail Access
Protocol, <a href="https://www.rfc-editor.org/rfc/rfc8620">RFC 8620</a>
and <a href="https://www.rfc-editor.org/rfc/rfc8621">RFC 8621</a>)
JMAP is a much nicer fit for modern network clients than IMAP is, as
it uses more widely deployed protocols and formats like HTTPS and JSON.</p>
<p>However, Dovecot doesn't speak JMAP natively, and the only standalone JMAP servers I've
evaluated (like Stalwart) all want to own the mailbox storage themselves, which would mean giving up Maildir.
I'm not quite willing to give up the simplicity of that email storage just yet...</p>
<p>The plan I'm considering is to put my <a href="https://anil.recoil.org/notes/aoah-2025-17">OCaml JMAP implementation</a> in front of Dovecot as a translating proxy.
JMAP requests would come in over HTTPS, get mapped to IMAP calls, and the responses can be sent back as JSON.
This also gives me an excuse to stress-test my OCaml JMAP code against real traffic. Stay tuned!</p>
<h2><a href="https://anil.recoil.org/news.xml#is-this-a-negative-result-for-self-hosting" class="anchor" aria-hidden="true"></a>Is this a negative result for self-hosting?</h2>
<p>It is rather unfortunate that "running an email server" in 2026 means
getting at least six separate DNS records correct before reliably sending or
receiving email. And securing an IPv4 block allocation from RIPE took <a href="https://github.com/samoht">Thomas Gazagnaire</a> almost a year.
And keeping all this up-to-date is a fair bit of work with respect to security,
but both <a href="https://nick.recoil.org">Nick Ludlam</a> and I use this (along with our friends and family who have accounts)
so it's for a small group of people.</p>
<p>The upside though, is what an excellent learning process it is to go through to
get up to speed on how the modern Internet really works. Email these days can
reset almost any aspect of our digital lives, and so it feels important to
maintain some semblance of agency over how it works. And it is quite
heartwarming that it's still possible to do on the Internet as a small outfit
without requiring any central authority to approve it!</p>
<p>The other thing I'm increasingly conscious of is that "secure" is a moving
target. Self-hosted services like ours have always faced opportunistic bot
scans, but the <a href="https://anil.recoil.org/notes/internet-immune-system">autonomous chaining of vulnerabilities by frontier AI models</a> has completely shifted the threat model.</p>
<p>The gap between a CVE being published and a working exploit being thrown at every
SMTP/IMAP listener on the public Internet is now probably measured in hours and not
weeks. Most of the hardening choices in this post; pinning Postfix to
specific addresses, isolating the webmail in containers on a separate IP,
greylisting and DNSBLs before handling email, are all pretty conventional
decisions to get some security in depth.
It does make me want to push way harder towards the dynamic antibotty-style active defences
I <a href="https://anil.recoil.org/papers/2025-internet-ecology">sketched out last year</a>...</p>
<p><img src="https://anil.recoil.org/images/moving-recoil-9.webp" alt="%c" title="Running your email server will occasionally result in your zooming through central London in a white van desperately trying to stop it flying out of a window (me, Nick Ludlam, James Cronin, 2002)"></p>
<p>I hope that this guide might come in useful to anyone else who wants to have a go!  I'm particularly excited by projects like <a href="https://ryan.freumh.org/talks/2026-fosdem-eilean.html">Eilean</a> which make this process more of a single-click process to deploy. Over the course of the next few months, I plan to write about how we're self hosting <em>other</em> services like photos, chat, location and more; see also our <a href="https://anil.recoil.org/notes/mirage-self-hosting">self-hosted MirageOS website</a> or my <a href="https://anil.recoil.org/notes/owntracks-and-lifecycle">OwnTracks location stack</a> for some background. It's good fun!</p><h1>References</h1><ul><li>Madhavapeddy et al (2025). Steps towards an Ecology for the Internet. Association for Computing Machinery. <a href="https://doi.org/10.1145/3744169.3744180" target="_blank"><i>10.1145/3744169.3744180</i></a></li>
<li>Scott et al (2010). Using functional programming within an industrial product group: perspectives and perceptions. ACM. <a href="https://doi.org/10.1145/1863543.1863557" target="_blank"><i>10.1145/1863543.1863557</i></a></li>
<li>Madhavapeddy (2025). Tracking locations with OwnTracks, Life Cycle and Home Assistant. <a href="https://doi.org/10.59350/13ras-yd957" target="_blank"><i>10.59350/13ras-yd957</i></a></li>
<li>Madhavapeddy (2026). Rewilding the Web: my workshop report from Edinburgh. <a href="https://doi.org/10.59350/g40yy-ks003" target="_blank"><i>10.59350/g40yy-ks003</i></a></li>
<li>Madhavapeddy (2026). The Internet needs an antibotty immune system, stat. <a href="https://doi.org/10.59350/snnnf-asc02" target="_blank"><i>10.59350/snnnf-asc02</i></a></li>
<li>Madhavapeddy (2025). Jane Street and Docker on moving to OCaml 5 at ICFP/SPLASH 2025. <a href="https://doi.org/10.59350/3jkaq-d3398" target="_blank"><i>10.59350/3jkaq-d3398</i></a></li>
<li>Doan et al (2022). An Empirical View on Consolidation of the Web. <a href="https://doi.org/10.1145/3503158" target="_blank"><i>10.1145/3503158</i></a></li>
<li>Ramanathan et al (2020). Quantifying the Impact of Blocklisting in the Age of Address Reuse. ACM. <a href="https://doi.org/10.1145/3419394.3423657" target="_blank"><i>10.1145/3419394.3423657</i></a></li>
<li>Ziv et al (2021). ASdb: a system for classifying owners of autonomous systems. ACM. <a href="https://doi.org/10.1145/3487552.3487853" target="_blank"><i>10.1145/3487552.3487853</i></a></li>
<li>Olea et al (2025). Evaluating Phishing Email Efficacy. ACM. <a href="https://doi.org/10.1145/3716489.3728437" target="_blank"><i>10.1145/3716489.3728437</i></a></li></ul>
