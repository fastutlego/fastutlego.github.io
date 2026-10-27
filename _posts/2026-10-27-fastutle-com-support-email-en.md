---
layout: post
lang: en
title: "Moving to our own domain right before launch, part 3: setting up support@fastutle.com"
date: 2026-10-27 18:40:00 +0900
description: "A custom-domain contact address without a mailbox: Cloudflare Email Routing for receiving, Gmail for sending, and the real cause of the SMTP 535 error that held me up."
tags: [Email, Ops]
read_minutes: 3
---

<p class="lang-link"><a href="/2026/10/27/fastutle-com-support-email/" lang="ko">한국어로 읽기 →</a></p>

*Part 3 of 3. [Part 1: from registering a domain to Cloudflare Pages](/2026/10/23/fastutle-com-migration-en/) · [Part 2: changing the address in the apps, the stores and Google Sign-In](/2026/10/27/fastutle-com-migration-apps-stores-en/)*

<aside class="summary" markdown="1">
**In short**

- Incoming mail is forwarded to Gmail by Cloudflare Email Routing; outgoing mail uses Gmail's "Send mail as". Total cost: nothing.
- Gmail's SMTP `535` error wasn't a wrong password. **Two-step verification was off**, so an app password couldn't be created at all.
- SPF includes both Cloudflare and Google; DMARC is `p=none` for now.
</aside>

Once the site lived at `fastutle.com`, I wanted the contact email on the same domain ([part 1](/2026/10/23/fastutle-com-migration-en/), [part 2](/2026/10/27/fastutle-com-migration-apps-stores-en/)). Rather than paying for a mailbox, I went with a free combination: Cloudflare for receiving, Gmail for sending.

## 1. Receiving: Cloudflare Email Routing

Turning on Email Routing in the Cloudflare dashboard adds three MX records, a DKIM record for forwarding and an SPF record automatically. The only rule is `support@fastutle.com` → my existing Gmail; mail to any other address (catch-all) is dropped.

## 2. Sending: Gmail's "Send mail as"

In Gmail settings, under "Send mail as", I added `support@fastutle.com`. The SMTP server is `smtp.gmail.com:587` (TLS), the username is the Gmail address, and the password has to be an **app password**. This is where I got stuck for a long time.

- Every attempt ended in `535-5.7.8 Username and Password not accepted`.
- The app-passwords page only said "The setting you are looking for is not available for your account".
- The cause: **two-step verification was off** on this Google account, so app passwords couldn't be created at all. After turning it on and creating an app password, it went through.

The confirmation email went to `support@` and arrived in Gmail, which also proved the receiving path.

## 3. SPF and DMARC

```text
fastutle.com.        TXT "v=spf1 include:_spf.mx.cloudflare.net include:_spf.google.com ~all"
_dmarc.fastutle.com. TXT "v=DMARC1; p=none"
```

Mail sent through Gmail doesn't carry a `fastutle.com` DKIM signature, so DMARC stays at `p=none` (monitoring). For a domain that sends no mail at all, the right setup would be `v=spf1 -all`, `p=reject` and a null MX.

## 4. Replacing the contact address

I changed the address to `support@fastutle.com` on the site (footer, support pages, privacy policies and terms with their change logs), in the app code, in the Play contact details, in the TestFlight feedback email and in the OAuth developer contact. The review contact on the version waiting for review was locked, so that one waits for the next submission.

**Email obfuscation.** The site's `mailto:` links had turned into `/cdn-cgi/l/email-protection…`. That's Cloudflare's proxy feature *Email Address Obfuscation*. I turned it off so the address is readable without scripts.

## If I did this again

- **On a Gmail SMTP 535 error**, check the account's two-step verification before doubting the password.
- **Cloudflare's proxy defaults** (email obfuscation) can change your HTML.
- **Put both the receiving and the sending side in SPF.** The SPF record Email Routing adds needs `_spf.google.com` as well, or mail sent through Gmail fails the check.

## Still to do

- Tighten DMARC step by step from `p=none`, once the reports come in
- Change the App Store review contact email after the review is done

*Start from the beginning: [Part 1: from registering a domain to Cloudflare Pages](/2026/10/23/fastutle-com-migration-en/)*
