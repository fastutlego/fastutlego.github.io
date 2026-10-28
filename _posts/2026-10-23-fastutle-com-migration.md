---
layout: post
lang: ko
title: "출시 직전에 자체 도메인으로 이사하기 1: 도메인 등록부터 Cloudflare Pages까지"
date: 2026-10-23 22:00:00 +0900
description: "앱 세 개의 사이트를 GitHub Pages에서 Cloudflare Pages와 fastutle.com으로 옮긴 기록 1편. 도메인 등록과 DNS, 비공개 저장소 배포, 보안 헤더, 옛 주소 정리까지."
tags: [사이트, 운영]
read_minutes: 8
---

<p class="lang-link"><a href="/2026/10/23/fastutle-com-migration-en/" lang="en">Read in English →</a></p>

*세 편 중 1편이다. [2편: 앱·스토어·Google 로그인의 주소 바꾸기](/2026/10/27/fastutle-com-migration-apps-stores/) · [3편: support@fastutle.com 메일 만들기](/2026/10/27/fastutle-com-support-email/)*

<aside class="summary" markdown="1">
**요약**

- 도메인은 Cloudflare Registrar에서 샀고, DNS 레코드 10개는 BIND 파일로 한 번에 넣었다.
- 저장소를 비공개로 두고 싶어서 GitHub Pages 대신 Cloudflare Pages로 옮겼다. 끊긴 시간은 3분쯤이다.
- 보안 헤더는 `_headers` 파일 하나로 해결했고, 옛 주소는 한동안 리디렉션 전용으로 살려 뒀다.
</aside>

iOS 심사 제출을 앞두고, 앱 소개·개인정보처리방침·다운로드 페이지가 있던 GitHub Pages 기본 주소(`<계정>.github.io`)를 자체 도메인 `fastutle.com`으로 옮겼다. 처음에는 "도메인 하나 연결하기"였는데, 하다 보니 사이트 호스팅, 앱 코드, App Store Connect, Google Play, Google 로그인(OAuth), 메일 주소까지 하루 동안 줄줄이 바뀌었다. 다음에 같은 일을 할 때 보려고 주제별로 세 편에 나눠 적어 둔다.

## 한눈에 보기

| 영역 | 전 | 후 |
|---|---|---|
| 사이트 주소 | GitHub Pages 기본 주소 | `https://fastutle.com` |
| 호스팅 | GitHub Pages (공개 저장소) | Cloudflare Pages (비공개 저장소) |
| 도메인·DNS | 없음 | Cloudflare Registrar + Cloudflare DNS |
| HTTPS | GitHub Pages 인증서 | Cloudflare 인증서 + HSTS 등 보안 헤더 |
| 앱 안의 링크·스토어 URL | 옛 주소 | 새 주소 ([2편](/2026/10/27/fastutle-com-migration-apps-stores/)) |
| Google 로그인 브랜딩 | 옛 주소, 브랜드 검증 실패 상태 | 새 주소, 재검증 통과 ([2편](/2026/10/27/fastutle-com-migration-apps-stores/)) |
| 문의 메일 | Gmail 주소 | `support@fastutle.com` ([3편](/2026/10/27/fastutle-com-support-email/)) |
| 옛 GitHub Pages 주소 | 앱 사이트 | 개인 페이지(지금 이 블로그) |

## 1. 도메인 사기와 DNS

**도메인 고르기.** 후보(`fastutle.com`, `.app`, `.kr` 등)가 비어 있는지는 `dig +short NS <도메인>`과 `whois`로 먼저 봤다. 스토어와 개인정보처리방침에 적을 주소라서 가장 익숙한 `.com`으로 정했다.

**등록업체.** Cloudflare Registrar를 골랐다. 원가로 팔아서 갱신비가 오르지 않고(첫해 약 $10), DNS를 같은 화면에서 바로 다룰 수 있다. Squarespace Domains(옛 Google Domains)에서도 살 수 있지만 가격이 두 배쯤이었다. `.kr`은 Cloudflare에서 살 수 없다.

**레코드 한 번에 넣기.** GitHub Pages용 레코드 10개(A 4개, AAAA 4개, `www` CNAME, 소유 인증 TXT)를 하나씩 입력하는 대신, BIND 형식 파일을 만들어 Cloudflare DNS의 Import로 한 번에 넣었다. 이때 "Proxy imported DNS records"는 꺼 둬야 GitHub이 인증서를 발급할 수 있다.

```text
$ORIGIN fastutle.com.
fastutle.com. 3600 IN A 185.199.108.153
...
www.fastutle.com. 3600 IN CNAME <계정>.github.io.
```

**GitHub 쪽.** 계정 설정의 Pages → *Add a domain*으로 도메인 소유를 먼저 인증하고(TXT), 저장소에 `CNAME` 파일을 넣었다. 인증서는 push하고 1~2분 만에 Let's Encrypt로 나왔고, 그 뒤 *Enforce HTTPS*를 켰다.

## 2. GitHub Pages에서 Cloudflare Pages로

처음 계획은 GitHub Pages에 도메인만 붙이는 거였다. 그런데 같은 날 저장소가 잠깐 비공개로 바뀌면서 사이트가 통째로 내려간 일이 있었다. 무료 플랜의 GitHub Pages는 공개 저장소에서만 동작한다. 사이트 원본을 비공개로 두고 싶어서 Cloudflare Pages로 옮기기로 했다.

**비용.** Cloudflare Pages는 정적 파일 요청과 전송량에 요금이 없다. 이 사이트(파일 175개, 6.7MB)는 무료 한도(파일 20,000개, 월 빌드 500회)에 한참 못 미친다.

**순서.** 끊기는 시간을 줄이려고 이렇게 했다.

1. 사용자 사이트 저장소(`<계정>.github.io`)의 이름을 일반 저장소 이름으로 바꿨다. GitHub은 저장소 이름이 바뀌어도 커스텀 도메인 설정을 따라가서, 그동안에도 `fastutle.com`은 열렸다.
2. Cloudflare Pages 프로젝트를 만들고, GitHub 앱에는 이 저장소 하나만 권한을 줬다.
3. `*.pages.dev` 임시 주소에서 GitHub Pages와 응답을 비교했다. 주요 주소, 리디렉션, 404, `app-ads.txt`, 개인정보처리방침 HTML까지 같은지 확인했다.
4. Pages 프로젝트에 커스텀 도메인을 추가했다. Cloudflare가 A/AAAA 레코드를 `<프로젝트>.pages.dev` CNAME(프록시 켬)으로 바꿔 준다. 도메인 확인이 끝날 때까지 약 3분간 522 오류가 났다.
5. `www`도 Pages에 붙이고, Redirect Rule로 `https://www.*` → `https://${1}`(쿼리 유지) 301을 걸었다.
6. 저장소를 비공개로 바꾸고, 비공개 상태에서도 Cloudflare 빌드가 되는지 다시 배포해 확인했다.

**첫 빌드 실패: 문자 인코딩.** Jekyll 빌드가 `Invalid US-ASCII character "\xE2"`로 SCSS 변환에서 멈췄다. Cloudflare 빌드 서버의 기본 로케일이 UTF-8이 아니라서다. 빌드 명령을 이렇게 바꿔 해결했다.

```bash
LANG=C.UTF-8 LC_ALL=C.UTF-8 bundle exec jekyll build
```

같이 `RUBY_VERSION`(환경 변수와 `.ruby-version`)을 고정하고, `.gitignore`에 있던 `Gemfile.lock`을 커밋해서 로컬과 같은 버전으로 빌드되게 했다.

**대소문자 주의.** 맥 기본 디스크는 대소문자를 구분하지 않아서 `/PopPunch/`와 `/poppunch/`가 한 폴더로 합쳐진다. 이 사이트는 원래 대소문자를 구분하는 디스크 이미지에서 빌드하고 있었는데, 옛 주소용 리디렉션 파일을 만들 때 이 함정에 한 번 빠졌다.

## 3. 보안 점검과 보안 헤더

옮기는 김에 별도 에이전트로 읽기 전용 보안 점검을 돌렸다.

- 작업 트리와 전체 커밋 이력에 API 키·토큰·비밀번호는 없었다.
- 다운로드 페이지(`/poppunch/get/`)는 목적지 호스트가 고정돼 있고 쿼리를 인코딩해서 오픈 리디렉션이 아니었다.
- **보안 헤더가 하나도 없었다.** GitHub Pages는 헤더를 못 바꾸지만 Cloudflare Pages는 `_headers` 파일로 정할 수 있다.

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

Jekyll은 `_`로 시작하는 파일을 빌드에서 빼기 때문에 `_config.yml`에 `include: [_headers]`가 필요하다. 미리보기 주소(`*.pages.dev`)가 검색에 잡히지 않게 `noindex`도 붙였다.

CSP를 켜자 Cloudflare가 자동으로 끼워 넣는 통계 스크립트(`static.cloudflareinsights.com`)가 막혔다. 개인정보처리방침에 없는 분석 스크립트라 막히는 쪽이 맞다고 보고 그대로 뒀다.

## 4. 옛 GitHub Pages 주소 정리

처음에는 옛 주소를 살려 두지 않아도 된다고 생각했다. 그런데 이미 배포된 TestFlight 빌드가 옛 주소의 개인정보처리방침·공유 링크를 쓰고 있었다. 그래서 한동안 옛 주소의 저장소를 리디렉션 전용으로 썼다.

- 옛 페이지마다 `index.html`을 만들어 `meta refresh` + `canonical` + `location.replace("https://fastutle.com" + 경로 + 쿼리 + 해시)`로 넘겼다. 경로는 소문자로 바꿔서 넘겼다.
- 그 밖의 경로는 `404.html`이 같은 방식으로 넘겼다.
- `app-ads.txt`는 광고 크롤러가 리디렉션을 따라가지 않아서 같은 한 줄을 그대로 뒀다.
- Google Search Console 인증 파일도 남겨 뒀다. OAuth 브랜딩이 아직 옛 도메인으로 인증돼 있었기 때문이다([2편](/2026/10/27/fastutle-com-migration-apps-stores/)).

OAuth 재검증이 끝난 뒤 이 저장소를 이력까지 지우고 개인 페이지로 다시 시작했다. 출시 전이라 영향을 받는 건 테스터뿐이었고, 곧이어 새 주소가 들어간 테스트 빌드를 배포했다. Search Console의 옛 URL 접두어 속성도 지웠다.

## 다음에 같은 일을 한다면

- **순서가 전부다.** 도메인 연결 → 새 호스팅 검증 → DNS 전환 → 앱·스토어 → 옛 주소 정리 순서로 하면 끊기는 시간이 거의 없다.
- **옛 주소를 쓰는 곳 목록을 먼저 만든다.** 앱 코드뿐 아니라 이미 배포된 빌드, 스토어(언어별), OAuth 브랜딩, Search Console, 광고 크롤러(`app-ads.txt`)까지 챙겨야 한다.
- **Cloudflare Pages + Jekyll**은 `LANG=C.UTF-8`, `RUBY_VERSION`, `Gemfile.lock` 커밋, `_headers`를 `include`에 넣기를 챙긴다.
- **Cloudflare 프록시 기본 기능**(이메일 난독화, 자동 통계 스크립트)이 사이트 HTML을 바꿀 수 있다.

## 남은 일

- DNSSEC 켜기

*다음 글: [2편: 앱·스토어·Google 로그인의 주소 바꾸기](/2026/10/27/fastutle-com-migration-apps-stores/)*
