---
layout: post
lang: ko
title: "출시 직전에 자체 도메인으로 이사하기 2: 앱·스토어·Google 로그인의 주소 바꾸기"
date: 2026-10-27 18:30:00 +0900
description: "사이트 주소가 바뀐 뒤 앱 코드, App Store Connect, Google Play, Google 로그인(OAuth) 브랜딩, Search Console에 남은 옛 주소를 하루에 정리한 기록."
tags: [스토어, 운영]
read_minutes: 4
---

<p class="lang-link"><a href="/2026/10/27/fastutle-com-migration-apps-stores-en/" lang="en">Read in English →</a></p>

*세 편 중 2편이에요. [1편: 도메인 등록부터 Cloudflare Pages까지](/2026/10/23/fastutle-com-migration/) · [3편: support@fastutle.com 메일 만들기](/2026/10/27/fastutle-com-support-email/)*

<aside class="summary" markdown="1">
**요약**

- 앱 세 개의 코드에서 옛 주소를 찾아 바꿨어요. 숨김 파일, 바이너리, 다른 브랜치까지 봤어요.
- App Store Connect는 API로 "출력만 → 확인 → 적용 → 재점검" 순서로 했고, Google Play의 개인정보처리방침 URL은 화면에서만 바꿀 수 있었어요.
- Google 로그인 브랜드 검증은 Search Console에 도메인 속성을 추가하자 바로 통과했어요.
</aside>

[1편](/2026/10/23/fastutle-com-migration/)에서 사이트를 `fastutle.com`으로 옮겼어요. 그런데 사이트 주소는 사이트 안에만 있는 게 아니에요. 앱 코드, 스토어의 링크, Google 로그인 동의 화면, 검색 콘솔에도 옛 주소가 남아 있었어요. 이 글은 그것들을 찾아 바꾼 기록이에요.

## 1. 앱 코드

세 앱(팝펀치 iOS·Android, 한밤(옛 이름 모두뷰어) Flutter, 2ndTV)에서 옛 주소를 찾아 바꿨어요. 개인정보처리방침·라이선스·공유·추천 링크, 그리고 한밤이 인터넷 연결을 확인할 때 DNS로 조회하는 호스트까지요. 빠진 곳이 없는지 이렇게 확인했어요.

- 숨김 파일과 바이너리까지 검색했어요(`rg -uuu -a`).
- 다른 브랜치는 `git merge-tree --write-tree main <branch>`로 "main을 합치면 옛 주소가 사라지는지"를 미리 계산했어요.
- 과거 설계·계획 문서는 기록이라 그대로 뒀어요.

## 2. App Store Connect

화면에서 언어마다 고치는 대신 fastlane에 들어 있는 Spaceship(App Store Connect API)으로 한 번에 처리했어요. 먼저 바꿀 필드를 **출력만 하는 점검**을 돌리고, 결과를 확인한 뒤 적용했어요.

- 앱 정보: 언어별 개인정보처리방침 URL
- 버전 정보: 지원 URL, 마케팅 URL (편집 가능한 버전만)
- 적용 뒤 텍스트 필드 344개와 TestFlight 정보까지 다시 훑어 옛 주소가 0건인지 확인했어요.

## 3. Google Play

연락처 웹사이트와 이메일은 Android Publisher API의 edit로 바꿀 수 있어요(`edits.details` 수정 후 commit). 하지만 **개인정보처리방침 URL은 API에 없어서** Play Console 화면에서 바꿔야 했어요. 이미 출시된 앱은 버튼이 "저장 및 출시"로 나오고, 누르면 바로 게시돼요.

## 4. Google 로그인(OAuth) 브랜딩과 Search Console

한밤은 Google Drive 연동 때문에 OAuth 동의 화면을 운영 중이에요. 브랜딩의 홈페이지·개인정보처리방침·승인된 도메인이 모두 옛 주소였고, 확인해 보니 브랜드 검증이 "홈페이지 URL이 나에게 등록되어 있지 않습니다"로 이미 실패해 있었어요.

1. Search Console에 `fastutle.com`을 **도메인 속성**으로 추가했어요. Cloudflare 자동 연결(다른 서비스에 DNS 권한 넘기기) 대신 "모든 DNS 제공업체"를 골라 TXT 값을 받아 직접 넣었어요.
2. 세 프로젝트의 브랜딩 링크와 승인된 도메인을 `fastutle.com`으로 바꿨어요.
3. Google 안내는 "소유 인증 후 24시간 뒤 재시도"였지만, 바로 "문제를 해결함"으로 재검증을 요청하니 **즉시 통과**했어요. 그다음 7일 안에 "브랜딩 게시"를 눌러야 동의 화면에 이름과 로고가 보여요.
4. 새 사이트맵(`https://fastutle.com/sitemap.xml`)을 제출했어요.

OAuth의 **사용자 지원 이메일**은 Google 계정 주소나 Google 그룹만 고를 수 있어서 자체 도메인 메일을 넣을 수 없었어요.

## 다음에 같은 일을 한다면

- **옛 주소를 쓰는 곳 목록을 먼저 만들어요.** 앱 코드뿐 아니라 이미 배포된 빌드, 스토어(언어별), OAuth 브랜딩, Search Console까지요.
- **화면 대신 API를 써요.** App Store Connect와 Play는 API로 "먼저 출력만 → 확인 → 적용 → 다시 점검" 순서로 하면 빠지는 언어가 없어요.
- **Play의 개인정보처리방침 URL은 화면에서만** 바꿀 수 있어요.
- **브랜드 검증 재시도는 바로 해 봐요.** 24시간 안내와 달리 소유 인증 직후에도 통과할 수 있어요.

*다음 글: [3편: support@fastutle.com 메일 만들기](/2026/10/27/fastutle-com-support-email/)*
