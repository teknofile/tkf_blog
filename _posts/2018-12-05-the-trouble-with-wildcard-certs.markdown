---
layout: post
title: "The Trouble with Wildcard Certs; Distributing Them"
date: 2018-12-05 00:00:00 -0700
categories: development home python vault github letsencrypt certbot ssl
---

<p>When <a href="https://www.letsencrypt.org/">Let's Encrypt</a> started issuing wildcard certs (i.e. *.your.domain.com) I was so very happy. In 'production' senses, I really dislike them. But for 'home' use?</p>
<p>Eh, they're ok.</p>
<p>They really simplify the management of being able to use them all over the place. No longer do I need a different cert for my SEIM, wiki, ldap hosts, www hosts, software repos, etc. Just deal with the cert.</p>
<p>When the cert (via <a href="https://certbot.eff.org/">certbot</a>) is renewed, however, it's only updated on the host that contacts the Lets Encrypt servers. What I've found is that because of the short expiration of the certs, every three months all of my hosts start complaining because of expired certs.</p>
<p>That's an issue. I also don't want to bundle the private key in my generic configuration management system. Checking in any private keys into any source code repo just gives me the shivers. Ew.</p>
<p>What I did then was realized, I already had a <a href="https://vaultproject.io/">vault</a> store that I I could store them in.</p>
<p>Since's its been awhile since I really wrote any code, I figured I would go ahead and write some python that would store the certs/chains/privkeys from my certbot host into my Vault instance.</p>
<p>Then, each host that needs the cert, I can then just query the vault (weekly, for example) and update the certificate from the vault.</p>
<p>All the code is in <a href="https://github.com/teknofile/CertMgr">github</a>. Would love to have folks let me know what they think.</p>
