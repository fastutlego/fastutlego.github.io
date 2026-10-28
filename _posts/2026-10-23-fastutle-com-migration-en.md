---
layout: post
lang: en
title: "Moving to our own domain right before launch, part 1: from registering a domain to Cloudflare Pages"
date: 2026-10-23 22:00:00 +0900
description: "How the site for three apps moved from GitHub Pages to Cloudflare Pages and fastutle.com. Part 1: buying the domain, DNS, deploying from a private repository, security headers, and retiring the old address."
tags: [Site, Ops]
read_minutes: 6
---

<p class="lang-link"><a href="/2026/10/23/fastutle-com-migration/" lang="ko">한국어로 읽기 →</a></p>

*Part 1 of 3. [Part 2: changing the address in the apps, the stores and Google Sign-In](/2026/10/27/fastutle-com-migration-apps-stores-en/) · [Part 3: setting up support@fastutle.com](/2026/10/27/fastutle-com-support-email-en/)*

<aside class="summary" markdown="1">
**In short**

- The domain came from Cloudflare Registrar, and the ten DNS records went in at once from a BIND file.
- To keep the repository private, the site moved from GitHub Pages to Cloudflare Pages, with about three minutes of downtime.
- Security headers took one `_headers` file, and the old address lived on for a while as a redirect-only site.
</aside>

With the iOS review submission coming up, I moved the app pages, privacy policies and download pages from the default GitHub Pages address (`<account>.github.io`) to our own domain, `fastutle.com`. It started as "connect one domain" and turned into a day where the hosting, the app code, App Store Connect, Google Play, Google Sign-In (OAuth) and the contact email all changed one after another. I'm writing it down in three parts, by topic, so the next move goes faster.

## At a glance

| Area | Before | After |
|---|---|---|
| Site address | default GitHub Pages address | `https://fastutle.com` |
| Hosting | GitHub Pages (public repository) | Cloudflare Pages (private repository) |
| Domain and DNS | none | Cloudflare Registrar + Cloudflare DNS |
| HTTPS | GitHub Pages certificate | Cloudflare certificate + HSTS and other security headers |
| Links inside the apps, store URLs | old address | new address ([part 2](/2026/10/27/fastutle-com-migration-apps-stores-en/)) |
| Google Sign-In branding | old address, brand verification failing | new address, verified again ([part 2](/2026/10/27/fastutle-com-migration-apps-stores-en/)) |
| Contact email | a Gmail address | `support@fastutle.com` ([part 3](/2026/10/27/fastutle-com-support-email-en/)) |
| Old GitHub Pages address | the app site | a personal page (this blog) |

## 1. Buying the domain and setting up DNS

**Choosing a name.** I checked whether the candidates (`fastutle.com`, `.app`, `.kr` and so on) were free with `dig +short NS <domain>` and `whois`. Since the address goes into the stores and the privacy policies, I went with the most familiar one, `.com`.

**Registrar.** I chose Cloudflare Registrar. It sells at cost, so renewals don't creep up (about $10 for the first year), and DNS lives on the same screen. Squarespace Domains (formerly Google Domains) also sells it, at roughly twice the price. `.kr` can't be bought through Cloudflare.

**All records at once.** Instead of typing the ten GitHub Pages records (four A, four AAAA, the `www` CNAME and the ownership TXT) one by one, I wrote a BIND-format file and used Import in Cloudflare DNS. Leave "Proxy imported DNS records" off, or GitHub can't issue the certificate.

```text
$ORIGIN fastutle.com.
fastutle.com. 3600 IN A 185.199.108.153
...
www.fastutle.com. 3600 IN CNAME <account>.github.io.
```

**On the GitHub side.** I verified domain ownership first under account settings → Pages → *Add a domain* (TXT), then added a `CNAME` file to the repository. The Let's Encrypt certificate appeared a minute or two after the push, and then I turned on *Enforce HTTPS*.

## 2. From GitHub Pages to Cloudflare Pages

The original plan was just to attach the domain to GitHub Pages. But the same day, the repository was briefly switched to private and the whole site went down: on the free plan, GitHub Pages only works from public repositories. I wanted the site source private, so I decided to move to Cloudflare Pages.

**Cost.** Cloudflare Pages doesn't charge for static requests or bandwidth. This site (175 files, 6.7 MB) is far below the free limits (20,000 files, 500 builds a month).

**Order of work.** To keep downtime short:

1. Rename the user-site repository (`<account>.github.io`) to an ordinary repository name. GitHub keeps the custom-domain setting across a rename, so `fastutle.com` stayed up meanwhile.
2. Create the Cloudflare Pages project and give its GitHub app access to this one repository only.
3. Compare responses between the temporary `*.pages.dev` address and GitHub Pages: the main pages, redirects, the 404 page, `app-ads.txt`, and the privacy-policy HTML.
4. Add the custom domain to the Pages project. Cloudflare rewrites the A/AAAA records to a proxied CNAME for `<project>.pages.dev`. Until the domain check finished there were about three minutes of 522 errors.
5. Attach `www` to Pages as well, and add a Redirect Rule: `https://www.*` → `https://${1}` (query preserved), 301.
6. Make the repository private and redeploy to confirm that Cloudflare still builds it.

**The first build failed on encoding.** Jekyll stopped during the SCSS conversion with `Invalid US-ASCII character "\xE2"`, because the build server's default locale isn't UTF-8. The fix was in the build command:

```bash
LANG=C.UTF-8 LC_ALL=C.UTF-8 bundle exec jekyll build
```

At the same time I pinned `RUBY_VERSION` (as an environment variable and in `.ruby-version`) and committed the `Gemfile.lock` that had been in `.gitignore`, so the build uses the same versions as my machine.

**Mind the case.** The default Mac disk is case-insensitive, so `/PopPunch/` and `/poppunch/` collapse into one folder. The site was already being built on a case-sensitive disk image, but I fell into this trap once while creating the redirect files for the old URLs.

## 3. A security check and security headers

While at it, I ran a read-only security review in a separate agent.

- No API keys, tokens or passwords in the working tree or anywhere in the commit history.
- The download page (`/poppunch/get/`) has a fixed destination host and encodes the query, so it isn't an open redirect.
- **There were no security headers at all.** GitHub Pages can't set them; Cloudflare Pages can, through a `_headers` file.

```text
/*
  Strict-Transport-Security: max-age=31536000; includeSubDomains
  Content-Security-Policy: default-src 'none'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self'; ...
  X-Content-Type-Options: nosniff
  X-Frame-Options: DENY
  Referrer-Policy: strict-origin-when-cross-origin

https://:project.pages.dev/*
  X-Robots-Tag: noindex
```

Jekyll leaves files starting with `_` out of the build, so `_config.yml` needs `include: [_headers]`. I also added `noindex` for the preview addresses (`*.pages.dev`) so they stay out of search results.

Turning on CSP blocked the analytics script Cloudflare injects automatically (`static.cloudflareinsights.com`). That script isn't in the privacy policy, so blocking it is the right outcome, and I left it that way.

## 4. Retiring the old GitHub Pages address

At first I thought the old address didn't need to stay alive. But TestFlight builds already out there used the old privacy-policy and share links. So for a while the old repository became a redirect-only site.

- Each old page got an `index.html` that forwards with `meta refresh` + `canonical` + `location.replace("https://fastutle.com" + path + query + hash)`, lowercasing the path on the way.
- Every other path was forwarded the same way by `404.html`.
- `app-ads.txt` kept its one line as is, because ad crawlers don't follow redirects.
- The Google Search Console verification file stayed too, because the OAuth branding was still verified against the old domain ([part 2](/2026/10/27/fastutle-com-migration-apps-stores-en/)).

Once the OAuth re-verification was done, I wiped that repository, history included, and restarted it as a personal page. Before launch, only testers were affected, and a new test build with the new address followed shortly. I also removed the old URL-prefix property from Search Console.

## If I did this again

- **Order is everything.** Connect the domain → verify the new hosting → switch DNS → apps and stores → retire the old address. In that order, downtime is close to zero.
- **List every place that uses the old address first.** Not only the app code: builds already shipped, the stores (per language), OAuth branding, Search Console, and ad crawlers (`app-ads.txt`).
- **Cloudflare Pages + Jekyll** needs `LANG=C.UTF-8`, a pinned `RUBY_VERSION`, a committed `Gemfile.lock`, and `_headers` in `include`.
- **Cloudflare's proxy defaults** (email obfuscation, the injected analytics script) can change your HTML.

## Still to do

- Turn on DNSSEC

*Next: [Part 2: changing the address in the apps, the stores and Google Sign-In](/2026/10/27/fastutle-com-migration-apps-stores-en/)*
