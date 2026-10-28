---
layout: post
lang: en
title: "Moving to our own domain right before launch, part 2: changing the address in the apps, the stores and Google Sign-In"
date: 2026-10-27 18:30:00 +0900
description: "After the site address changed, the old one was still in the app code, App Store Connect, Google Play, the Google Sign-In (OAuth) branding and Search Console. Clearing all of it in a day."
tags: [Stores, Ops]
read_minutes: 4
---

<p class="lang-link"><a href="/2026/10/27/fastutle-com-migration-apps-stores/" lang="ko">한국어로 읽기 →</a></p>

*Part 2 of 3. [Part 1: from registering a domain to Cloudflare Pages](/2026/10/23/fastutle-com-migration-en/) · [Part 3: setting up support@fastutle.com](/2026/10/27/fastutle-com-support-email-en/)*

<aside class="summary" markdown="1">
**In short**

- I replaced the old address in three apps' code, including hidden files, binaries and other branches.
- App Store Connect was done through the API as "print only → check → apply → re-check"; Google Play's privacy-policy URL can only be changed in the console.
- Google Sign-In brand verification passed immediately once the domain was added as a Search Console domain property.
</aside>

[Part 1](/2026/10/23/fastutle-com-migration-en/) moved the site to `fastutle.com`. But a site's address doesn't live only on the site. It was also in the app code, the store listings, the Google Sign-In consent screen and the search console.

## 1. App code

Across the three apps (Pop Punch for iOS and Android, Hanbam (formerly Modu Viewer) in Flutter, and 2ndTV) I searched for the old address and replaced it: privacy-policy, licence, share and recommendation links, and even the host Hanbam resolves over DNS to check for an internet connection. To make sure nothing was missed:

- Search hidden files and binaries too (`rg -uuu -a`).
- For other branches, compute ahead of time whether merging main removes the old address: `git merge-tree --write-tree main <branch>`.
- Old design and planning documents stayed as they were; they're records.

## 2. App Store Connect

Instead of editing each language in the web UI, I used Spaceship (the App Store Connect API client bundled with fastlane) to do it in one pass. First a **dry run that only prints** the fields to change, then, after checking the output, the real run.

- App information: the privacy-policy URL per language
- Version information: support URL and marketing URL (editable versions only)
- Afterwards, a sweep of all 344 text fields plus the TestFlight information confirmed zero occurrences of the old address.

## 3. Google Play

The contact website and email can be changed through the Android Publisher API with an edit (modify `edits.details`, then commit). But **the privacy-policy URL isn't in the API**, so that one had to be changed in the Play Console. For an app that's already live the button reads "Save and publish", and it publishes right away.

## 4. Google Sign-In (OAuth) branding and Search Console

Hanbam runs an OAuth consent screen for its Google Drive integration. The branding's home page, privacy-policy URL and authorised domains were all the old address, and brand verification had in fact already failed with "the home page URL is not registered to you".

1. Add `fastutle.com` to Search Console as a **domain property**. Rather than the automatic Cloudflare connection (which hands DNS permissions to another service), I chose "any DNS provider", took the TXT value and added it myself.
2. Change the branding links and authorised domains in all three projects to `fastutle.com`.
3. Google's guidance says "retry 24 hours after verifying ownership", but requesting re-verification right away with "I've fixed the issue" **passed immediately**. Within the next seven days you then have to click "Publish branding", or the consent screen won't show the name and logo.
4. Submit the new sitemap (`https://fastutle.com/sitemap.xml`).

The OAuth **user support email** only accepts a Google account address or a Google Group, so the custom-domain address couldn't go there.

## If I did this again

- **List every place that uses the old address first.** Not only the app code: builds already shipped, the stores (per language), OAuth branding and Search Console.
- **Use the API, not the UI.** For App Store Connect and Play, "print only → check → apply → re-check" leaves no language behind.
- **Play's privacy-policy URL can only be changed in the console.**
- **Retry brand verification straight away.** Despite the 24-hour notice, it can pass right after ownership is verified.

*Next: [Part 3: setting up support@fastutle.com](/2026/10/27/fastutle-com-support-email-en/)*
