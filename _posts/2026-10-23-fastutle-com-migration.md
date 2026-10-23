---
layout: post
title: "출시 직전에 자체 도메인으로 이사하기: GitHub Pages에서 fastutle.com까지"
date: 2026-10-23 22:00:00 +0900
description: "앱 세 개의 사이트를 GitHub Pages에서 Cloudflare Pages와 fastutle.com으로 옮기고, 앱·스토어·Google 로그인·메일 주소까지 하루에 정리한 기록이에요."
---

iOS 심사 제출을 앞두고, 앱 소개·개인정보처리방침·다운로드 페이지가 있던 `fastutlego.github.io`를 자체 도메인 `fastutle.com`으로 옮겼어요. 처음에는 "도메인 하나 연결하기"였는데, 하다 보니 사이트 호스팅, 앱 코드, App Store Connect, Google Play, Google 로그인(OAuth), 메일 주소까지 하루 동안 줄줄이 바뀌었어요. 나중에 같은 일을 다시 할 때 보려고, 한 일을 주제별로 나누고 막혔던 점을 함께 적어 둬요.

## 한눈에 보기

| 영역 | 전 | 후 |
|---|---|---|
| 사이트 주소 | `https://fastutlego.github.io` | `https://fastutle.com` |
| 호스팅 | GitHub Pages (공개 저장소) | Cloudflare Pages (비공개 저장소) |
| 도메인·DNS | 없음 | Cloudflare Registrar + Cloudflare DNS |
| HTTPS | GitHub Pages 인증서 | Cloudflare 인증서, HSTS 등 보안 헤더 추가 |
| 앱 안의 링크 | `fastutlego.github.io/...` | `fastutle.com/...` |
| 스토어 URL | 개인정보처리방침·지원·마케팅·웹사이트 모두 옛 주소 | 모두 새 주소 |
| Google 로그인 브랜딩 | 옛 주소, 브랜드 검증 실패 상태 | 새 주소, 재검증 통과 후 게시 |
| 문의 메일 | `fastutle.support@gmail.com` | `support@fastutle.com` (Gmail로 전달·답장) |
| `fastutlego.github.io` | 앱 사이트 | 개인 페이지(지금 이 블로그) |

## 1. 도메인 사기와 DNS

**도메인 고르기.** 후보(`fastutle.com`, `.app`, `.kr` 등)가 비어 있는지는 `dig +short NS 도메인`과 `whois`로 먼저 봤어요. 스토어와 개인정보처리방침에 쓸 주소라 가장 익숙한 `.com`으로 정했어요.

**등록업체.** Cloudflare Registrar를 골랐어요. 원가로 팔아서 갱신비가 오르지 않고(첫해 약 $10), DNS를 같은 화면에서 바로 다룰 수 있어요. Squarespace Domains(옛 Google Domains)도 살 수 있지만 가격이 두 배쯤이었어요. `.kr`은 Cloudflare에서 살 수 없어요.

**레코드 한 번에 넣기.** GitHub Pages용 레코드 10개(A 4개, AAAA 4개, `www` CNAME, 소유 인증 TXT)를 하나씩 입력하는 대신, BIND 형식 파일을 만들어 Cloudflare DNS의 **Import**로 한 번에 넣었어요. "Proxy imported DNS records"는 꺼 둬야 GitHub이 인증서를 발급할 수 있어요.

```text
$ORIGIN fastutle.com.
fastutle.com. 3600 IN A 185.199.108.153
...
www.fastutle.com. 3600 IN CNAME fastutlego.github.io.
```

**GitHub 쪽.** 계정 설정의 Pages → *Add a domain*으로 도메인 소유를 먼저 인증하고(TXT), 저장소에 `CNAME` 파일을 넣었어요. 인증서는 push하고 1~2분 만에 Let's Encrypt로 나왔고, 그 뒤 *Enforce HTTPS*를 켰어요.

## 2. GitHub Pages에서 Cloudflare Pages로

처음 계획은 GitHub Pages에 도메인만 붙이는 거였는데, 같은 날 저장소가 잠깐 비공개로 바뀌면서 사이트가 통째로 내려간 일이 있었어요. 무료 플랜의 GitHub Pages는 공개 저장소에서만 동작해요. 사이트 원본을 비공개로 두고 싶어서 Cloudflare Pages로 옮기기로 했어요.

**비용.** Cloudflare Pages는 정적 파일 요청과 전송량에 요금이 없어요. 이 사이트(파일 175개, 6.7MB)는 무료 한도(파일 20,000개, 월 빌드 500회)에 한참 못 미쳐요.

**순서.** 다운타임을 줄이려고 이렇게 했어요.

1. 저장소 이름을 `fastutlego.github.io` → `fastutle-site`로 바꿨어요. GitHub은 사용자 사이트 저장소 이름이 바뀌어도 커스텀 도메인 설정을 따라가서, 그동안에도 `fastutle.com`은 열렸어요.
2. Cloudflare Pages 프로젝트를 만들고 GitHub 앱은 이 저장소 하나에만 권한을 줬어요.
3. `*.pages.dev` 임시 주소에서 GitHub Pages와 응답을 비교했어요. 주요 주소, 리디렉션, 404, `app-ads.txt`, 개인정보처리방침 HTML까지 같은지 확인했어요.
4. Pages 프로젝트에 커스텀 도메인을 추가했어요. Cloudflare가 A/AAAA 레코드를 `fastutle-site.pages.dev` CNAME(프록시 켬)으로 바꿔 줘요. 도메인 확인이 끝날 때까지 **약 3분간 522 오류**가 났어요.
5. `www`도 Pages에 붙이고, Redirect Rule로 `https://www.*` → `https://${1}`(쿼리 유지) 301을 걸었어요.
6. 저장소를 비공개로 바꾸고, 비공개 상태에서도 Cloudflare 빌드가 되는지 다시 배포해 확인했어요.

**첫 빌드 실패: 문자 인코딩.** Jekyll 빌드가 `Invalid US-ASCII character "\xE2"`로 SCSS 변환에서 멈췄어요. Cloudflare 빌드 서버의 기본 로케일이 UTF-8이 아니라서예요. 빌드 명령을 이렇게 바꿔 해결했어요.

```bash
LANG=C.UTF-8 LC_ALL=C.UTF-8 bundle exec jekyll build
```

같이 `RUBY_VERSION`(환경 변수와 `.ruby-version`)을 고정하고, `.gitignore`에 있던 `Gemfile.lock`을 커밋해서 로컬과 같은 버전으로 빌드되게 했어요.

**대소문자 주의.** 맥 기본 디스크는 대소문자를 구분하지 않아서 `/PopPunch/`와 `/poppunch/`가 한 폴더로 합쳐져요. 이 사이트는 원래 대소문자를 구분하는 디스크 이미지에서 빌드하고 있었는데, 옛 주소용 리디렉션 파일을 만들 때 이 함정에 한 번 빠졌어요.

## 3. 보안 점검과 보안 헤더

옮기는 김에 별도 에이전트로 읽기 전용 보안 점검을 돌렸어요.

- 작업 트리와 전체 커밋 이력에 API 키·토큰·비밀번호는 없었어요.
- 다운로드 페이지(`/poppunch/get/`)는 목적지 호스트가 고정돼 있고 쿼리를 인코딩해서 오픈 리디렉션이 아니었어요.
- **보안 헤더가 하나도 없었어요.** GitHub Pages는 헤더를 못 바꾸지만 Cloudflare Pages는 `_headers` 파일로 정할 수 있어요.

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

Jekyll은 `_`로 시작하는 파일을 빌드에서 빼기 때문에 `_config.yml`에 `include: [_headers]`가 필요해요. 미리보기 주소(`*.pages.dev`)가 검색에 잡히지 않게 `noindex`도 붙였어요.

CSP를 켜자 Cloudflare가 자동으로 끼워 넣는 통계 스크립트(`static.cloudflareinsights.com`)가 막혔어요. 개인정보처리방침에 없는 분석 스크립트라 막히는 쪽이 맞다고 보고 그대로 뒀어요.

## 4. 옛 주소 `fastutlego.github.io` 정리

처음에는 옛 주소를 살려 두지 않아도 된다고 생각했는데, 이미 배포된 TestFlight 빌드가 옛 주소의 개인정보처리방침·공유 링크를 쓰고 있었어요. 그래서 한동안 `fastutlego.github.io` 저장소를 **리디렉션 전용**으로 썼어요.

- 옛 페이지마다 `index.html`을 만들어 `meta refresh` + `canonical` + `location.replace("https://fastutle.com" + 경로 + 쿼리 + 해시)`로 넘겼어요. 경로는 소문자로 바꿔서 넘겼어요.
- 그 밖의 경로는 `404.html`이 같은 방식으로 넘겼어요.
- `app-ads.txt`는 광고 크롤러가 리디렉션을 따라가지 않아서 같은 한 줄을 그대로 뒀어요.
- Google Search Console 인증 파일도 남겨 뒀어요. 아래 OAuth 브랜딩이 아직 옛 도메인으로 인증돼 있었기 때문이에요.

OAuth 재검증이 끝난 뒤 이 저장소를 이력까지 지우고 개인 페이지로 다시 시작했어요. 출시 전이라 영향을 받는 건 테스터뿐이었고, 곧이어 새 주소가 들어간 테스트 빌드를 배포했어요. Search Console의 옛 URL 접두어 속성도 지웠어요.

## 5. 앱 코드와 스토어 주소 바꾸기

**앱 코드.** 세 앱(팝펀치 iOS·Android, 모두뷰어 Flutter, 2ndTV)에서 `fastutlego.github.io`를 찾아 바꿨어요. 개인정보처리방침·라이선스·공유·추천 링크, 그리고 모두뷰어가 인터넷 연결을 확인할 때 DNS로 조회하는 호스트까지 바꿨어요. 확인할 때는 이렇게 했어요.

- 숨김 파일과 바이너리까지 검색했어요(`rg -uuu -a`).
- 다른 브랜치는 `git merge-tree --write-tree main <branch>`로 "main을 합치면 옛 주소가 사라지는지"를 미리 계산했어요.
- 과거 설계·계획 문서는 기록이라 그대로 뒀어요.

**App Store Connect.** 화면에서 언어마다 고치는 대신 fastlane에 들어 있는 Spaceship(ASC API)으로 한 번에 처리했어요. 먼저 바꿀 필드를 출력만 하는 점검을 돌리고, 확인한 뒤 적용했어요.

- 앱 정보: 언어별 개인정보처리방침 URL
- 버전 정보: 지원 URL, 마케팅 URL (편집 가능한 버전만)
- 텍스트 필드 344개와 TestFlight 정보까지 다시 훑어 옛 주소가 0건인지 확인했어요.

**Google Play.** 연락처 웹사이트와 이메일은 Android Publisher API의 edit로 바꿀 수 있어요(`edits.details` 수정 후 commit). 하지만 **개인정보처리방침 URL은 API에 없어서** Play Console 화면에서 바꿔야 했어요. 이미 출시된 앱은 버튼이 "저장 및 출시"로 나오고, 누르면 바로 게시돼요.

## 6. Google 로그인(OAuth) 브랜딩과 Search Console

모두뷰어는 Google Drive 연동 때문에 OAuth 동의 화면을 운영 중이에요. 브랜딩의 홈페이지·개인정보처리방침·승인된 도메인이 모두 옛 주소였고, 확인해 보니 브랜드 검증이 "홈페이지 URL이 나에게 등록되어 있지 않습니다"로 이미 실패해 있었어요.

1. Search Console에 `fastutle.com`을 **도메인 속성**으로 추가했어요. Cloudflare 자동 연결(다른 서비스에 DNS 권한 넘기기) 대신 "모든 DNS 제공업체"를 골라 TXT 값을 받아 직접 넣었어요.
2. 세 프로젝트의 브랜딩 링크와 승인된 도메인을 `fastutle.com`으로 바꿨어요.
3. Google 안내는 "소유 인증 후 24시간 뒤 재시도"였지만, 바로 "문제를 해결함"으로 재검증을 요청하니 **즉시 통과**했어요. 그다음 7일 안에 "브랜딩 게시"를 눌러야 동의 화면에 이름과 로고가 보여요.
4. 새 사이트맵(`https://fastutle.com/sitemap.xml`)을 제출했어요.

OAuth의 **사용자 지원 이메일**은 Google 계정 주소나 Google 그룹만 고를 수 있어서 자체 도메인 메일을 넣을 수 없었어요.

## 7. `support@fastutle.com` 메일 만들기

메일함을 따로 두지 않고, 받기는 Cloudflare 이메일 라우팅, 보내기는 Gmail로 하는 무료 조합을 골랐어요.

**받기 (Cloudflare Email Routing).** 켜면 MX 3개, 전달용 DKIM, SPF가 자동으로 들어가요. 규칙은 `support@fastutle.com` → 기존 Gmail 하나만 두고, 그 밖의 주소(catch-all)는 버려요.

**보내기 (Gmail "다른 주소에서 메일 보내기").** SMTP 서버는 `smtp.gmail.com:587`(TLS), 사용자 이름은 Gmail 주소, 비밀번호는 **앱 비밀번호**예요. 여기서 오래 막혔어요.

- 계속 `535-5.7.8 Username and Password not accepted`가 났어요.
- 앱 비밀번호 화면이 "찾고 있는 설정을 계정에서 사용할 수 없습니다"였어요. 이 Google 계정에 **2단계 인증이 꺼져 있어서** 앱 비밀번호 자체를 만들 수 없는 상태였어요.
- 2단계 인증을 켜고 앱 비밀번호를 만든 뒤에야 통과했어요. 확인 메일이 `support@`로 가서 Gmail에 도착한 것으로 받기 경로도 함께 확인됐어요.

**SPF·DMARC.**

```text
fastutle.com.        TXT "v=spf1 include:_spf.mx.cloudflare.net include:_spf.google.com ~all"
_dmarc.fastutle.com. TXT "v=DMARC1; p=none"
```

Gmail로 보내는 메일에는 `fastutle.com` DKIM 서명이 붙지 않아서 DMARC는 `p=none`(모니터링)으로 뒀어요. 메일을 아예 안 쓰는 도메인이었다면 `v=spf1 -all`, `p=reject`, null MX로 막는 게 맞아요.

**문의 주소 교체.** 사이트(바닥글, 지원, 개인정보처리방침·약관과 그 변경 이력), 앱 코드, Play 연락처, TestFlight 피드백 이메일, OAuth 개발자 연락처를 `support@fastutle.com`으로 바꿨어요. 심사 대기 중인 버전의 심사 연락처는 잠겨 있어서 다음 제출 때 바꾸기로 했어요.

**이메일 난독화.** 사이트 HTML의 `mailto:`가 `/cdn-cgi/l/email-protection…`으로 바뀌어 있었어요. Cloudflare 프록시의 *Email Address Obfuscation* 기능 때문이에요. 스크립트 없이도 주소가 보이도록 이 기능은 껐어요.

## 8. 다음에 같은 일을 한다면

- **순서가 전부예요.** 도메인 연결 → 새 호스팅 검증 → DNS 전환 → 앱·스토어 → 옛 주소 정리 순서로 하면 끊기는 시간이 거의 없어요.
- **옛 주소를 쓰는 곳 목록을 먼저 만들어요.** 앱 코드뿐 아니라 이미 배포된 빌드, 스토어(언어별), OAuth 브랜딩, Search Console, 광고 크롤러(`app-ads.txt`)까지 챙겨야 해요.
- **화면 대신 API를 써요.** App Store Connect와 Play는 API로 "먼저 출력만 → 확인 → 적용 → 다시 점검" 순서로 하면 빠지는 언어가 없어요.
- **Play의 개인정보처리방침 URL은 화면에서만** 바꿀 수 있어요.
- **Cloudflare Pages + Jekyll**: `LANG=C.UTF-8`, `RUBY_VERSION`, `Gemfile.lock` 커밋, `_headers`를 `include`에 넣기를 챙겨요.
- **Gmail SMTP 535 오류**가 나면 비밀번호를 의심하기 전에 그 계정의 2단계 인증부터 확인해요.
- **Cloudflare 프록시 기본 기능**(이메일 난독화, 자동 통계 스크립트)이 사이트 HTML을 바꿀 수 있어요.

## 남은 일

- DNSSEC 켜기
- DMARC를 `p=none`에서 단계적으로 강화하기(보고서를 보고 나서)
- 심사가 끝나면 App Store 심사 연락처 이메일 바꾸기
