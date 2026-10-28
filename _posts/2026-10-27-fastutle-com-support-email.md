---
layout: post
lang: ko
title: "출시 직전에 자체 도메인으로 이사하기 3: support@fastutle.com 메일 만들기"
date: 2026-10-27 18:40:00 +0900
description: "메일함 없이 자체 도메인 문의 주소를 만든 기록. 받기는 Cloudflare Email Routing, 보내기는 Gmail, 그리고 오래 막혔던 SMTP 535 오류의 진짜 원인."
tags: [메일, 운영]
read_minutes: 4
---

<p class="lang-link"><a href="/2026/10/27/fastutle-com-support-email-en/" lang="en">Read in English →</a></p>

*세 편 중 3편이다. [1편: 도메인 등록부터 Cloudflare Pages까지](/2026/10/23/fastutle-com-migration/) · [2편: 앱·스토어·Google 로그인의 주소 바꾸기](/2026/10/27/fastutle-com-migration-apps-stores/)*

<aside class="summary" markdown="1">
**요약**

- 받기는 Cloudflare Email Routing으로 Gmail에 전달하고, 보내기는 Gmail의 "다른 주소에서 메일 보내기"를 썼다. 비용은 0원이다.
- Gmail SMTP의 `535` 오류는 비밀번호 문제가 아니라 **2단계 인증이 꺼져 있어서** 앱 비밀번호를 만들 수 없던 게 원인이었다.
- SPF는 Cloudflare와 Google을 모두 포함했고, DMARC는 일단 `p=none`이다.
</aside>

사이트 주소가 `fastutle.com`이 되자 문의 메일도 같은 도메인으로 쓰고 싶었다([1편](/2026/10/23/fastutle-com-migration/), [2편](/2026/10/27/fastutle-com-migration-apps-stores/)). 메일함을 따로 두지 않고, 받기는 Cloudflare, 보내기는 Gmail로 하는 무료 조합을 골랐다.

## 1. 받기: Cloudflare Email Routing

Cloudflare 대시보드에서 Email Routing을 켜면 MX 3개, 전달용 DKIM, SPF 레코드가 자동으로 들어간다. 규칙은 `support@fastutle.com` → 기존 Gmail 하나만 두고, 그 밖의 주소로 오는 메일(catch-all)은 버린다.

## 2. 보내기: Gmail "다른 주소에서 메일 보내기"

Gmail 설정의 "다른 주소에서 메일 보내기"에 `support@fastutle.com`을 추가했다. SMTP 서버는 `smtp.gmail.com:587`(TLS), 사용자 이름은 Gmail 주소, 비밀번호는 **앱 비밀번호**다. 여기서 오래 막혔다.

- 계속 `535-5.7.8 Username and Password not accepted`가 났다.
- 앱 비밀번호 화면은 "찾고 있는 설정을 계정에서 사용할 수 없습니다"라고만 했다.
- 원인은 이 Google 계정에 **2단계 인증이 꺼져 있어서** 앱 비밀번호 자체를 만들 수 없는 상태였다는 것이다. 2단계 인증을 켜고 앱 비밀번호를 만든 뒤에야 통과했다.

확인 메일이 `support@`로 가서 Gmail에 도착한 것으로, 받기 경로도 함께 검증됐다.

## 3. SPF와 DMARC

```text
fastutle.com.        TXT "v=spf1 include:_spf.mx.cloudflare.net include:_spf.google.com ~all"
_dmarc.fastutle.com. TXT "v=DMARC1; p=none"
```

Gmail로 보내는 메일에는 `fastutle.com`의 DKIM 서명이 붙지 않아서 DMARC는 `p=none`(모니터링)으로 뒀다. 메일을 아예 안 쓰는 도메인이었다면 `v=spf1 -all`, `p=reject`, null MX로 막는 게 맞다.

## 4. 문의 주소 교체

사이트(바닥글, 지원, 개인정보처리방침·약관과 그 변경 이력), 앱 코드, Play 연락처, TestFlight 피드백 이메일, OAuth 개발자 연락처를 `support@fastutle.com`으로 바꿨다. 심사 대기 중인 버전의 심사 연락처는 잠겨 있어서 다음 제출 때 바꾸기로 했다.

**이메일 난독화.** 사이트 HTML의 `mailto:`가 `/cdn-cgi/l/email-protection…`으로 바뀌어 있었다. Cloudflare 프록시의 *Email Address Obfuscation* 기능 때문이다. 스크립트 없이도 주소가 보이도록 이 기능은 껐다.

## 다음에 같은 일을 한다면

- **Gmail SMTP 535 오류**가 나면 비밀번호를 의심하기 전에 그 계정의 2단계 인증부터 확인한다.
- **Cloudflare 프록시 기본 기능**(이메일 난독화)이 사이트 HTML을 바꿀 수 있다.
- **SPF에는 받는 쪽과 보내는 쪽을 모두** 넣는다. Email Routing이 넣어 준 SPF에 `_spf.google.com`을 더해야 Gmail로 보낸 메일이 통과한다.

## 남은 일

- DMARC를 `p=none`에서 단계적으로 강화하기(보고서를 보고 나서)
- 심사가 끝나면 App Store 심사 연락처 이메일 바꾸기

*처음부터 읽기: [1편: 도메인 등록부터 Cloudflare Pages까지](/2026/10/23/fastutle-com-migration/)*
