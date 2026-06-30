# 블로그 로드맵

Jekyll + Chirpy 기반 개인 기술 블로그 (`https://data-sy.github.io`) — 학습 기록·자료 공유

> **운영 방식:** 평소엔 `main`에 직접 commit + push → GitHub Actions가 빌드·배포(라이브 반영). 큰 개편만 브랜치+PR. 초안은 `_drafts/`(빌드 제외), 완성 시 `_posts/`로 이동. 로컬 미리보기: `export PATH="/opt/homebrew/opt/ruby@3.4/bin:$PATH"` 후 `bundle exec jekyll serve`.

## A. 레포·뼈대 ✅
- [x] A1. Chirpy 스타터 템플릿으로 `data-sy.github.io` 레포 생성 + 로컬 클론
- [x] A2. `setup/chirpy` 작업 브랜치 생성
- [x] A3. 로드맵 문서 작성 (CHECKLIST.md → ROADMAP.md로 개명)

## B. 설정 ✅
- [x] B1. `_config.yml` — url, title, tagline, lang(ko-KR), timezone(Asia/Seoul)
- [x] B2. 소셜/프로필 — GitHub 링크, 트위터 제거
- [x] B3. 스타터 데모 정리

## C. 콘텐츠(초기) ✅
- [x] C1. About 페이지 (자기소개·주요 프로젝트)
- [x] C2. 첫 글 작성 (`블로그를 시작하며`)

## D. 배포·검증 ✅
- [x] D1. `setup/chirpy` push
- [x] D2. `main` 병합 (PR #1, #2)
- [x] D3. GitHub Pages 소스 = GitHub Actions
- [x] D4. 빌드 성공 + 사이트 라이브 확인

## E. 학습 자료 공유 트랙 (study-sources → 블로그)
- [ ] E1. HTML 게재 방식 결정 — ① 원본 HTML 그대로(정적 페이지) / ② Chirpy 글로 변환(마크다운) / ③ 혼합
- [ ] E2. **AI 고지 문구** 정하고 게재 — 어디에(글 하단 / About / 공통 푸터)·어떤 문구로
- [ ] E3. 공유 자료의 작성자 표기 통일 (대상 파일 한정 — 일부만)
- [ ] E4. 공유할 자료 큐레이션 (84개 전부 아님 — 정말 유용한 것부터, 점진적으로)
- [ ] E5. 파일럿 1편 — "동시성 시작 전 알아야 할 사전지식" 게재해보고 방식 확정

## F. Velog 기술글 이관 (→ `AI Programming` 카테고리)
- [x] F1. 기술글 9편 이관 — AI Assistant #3~#9 + Spring AOP + 웹페이지 접속 과정. 원본 마크다운 충실 이식, 이미지 24개 로컬화 (PR #4)
- [x] F2. 카테고리 재편 — `[AI Programming, QLT]` 2단계 + ASCII 슬러그(`/categories/ai-programming/`)
- [x] F3. 타이머 앱 개발기 이미지 표 → flex 좌우 배치, 소개 블록 불릿 정리
- [ ] F4. **이관 글 미완성 표시 채우기** (원문 충실 이식 원칙상 보존된 작성자 "예정" 자리. 줄번호는 편집 시 밀릴 수 있어 표시 문구 병기)
  - [ ] `_posts/2025-08-17-ai-assistant-5.md` `(간격 주는 용도의 짤. 넣을 예정)` — 간격용 짤 이미지 추가
  - [ ] `…ai-assistant-5.md` `(이미지 위치 조절 예정)` — 알림/오디오 세션 이미지 위치 조정
  - [ ] `…ai-assistant-5.md` `삽질 기록은 여기... (추가 예정)` — "알람 기능 트러블 슈팅" 글 작성 후 링크 연결 (원본 velog 글은 비공개/삭제 상태)
  - [ ] `…ai-assistant-5.md` `(이전 아키텍처 다이어그램 → 새로운 아키텍처 다이어그램 첨부 예정)` — 클린 아키텍처 전/후 다이어그램 첨부
  - [ ] `_posts/2025-09-21-ai-assistant-9.md` `*Suspended*문제*(링크 연결 예정)*` — #5의 Suspended 단락으로 링크 연결
- [ ] F5. (사용자) 이관·검증 완료 후 Velog 원본 글 삭제
- [x] F6. **Velog "Database" 시리즈 이관** — 공개 39편을 같은 날 주제별 병합 → 9편(`[CS, Database]`, 서술형 제목, 2025-02-03~02-17). 이미지 85장 로컬화, 얼굴 GIF 2개 제외, velog 의존성 0건. (커밋 1832ec2, feat/migrate-velog)
- [ ] F7. **(백로그) DB 시리즈 9편 전반 폴리싱** — 현재는 예전 글을 거의 그대로 긁어온 상태. 이관 완료된 본문을 바탕으로 전반적으로 손봐야 함. 다듬을 축(예시): 도입부 훅·문제의식, 군더더기/구어체 정리, 글 간 흐름·상호 링크, 제목·요약 일관성, 코드/예시 현행화, 이미지 캡션·alt. 우선순위·범위는 착수 시 정함. 대상: `2025-02-03 ~ 02-17` DB 9편(`database-basics-transaction-integrity` 외 8편).

## G. 브랜딩·홈 정돈
- [x] G1. **파비콘 교체** — ⚖️ 저울 이모지 세트(`_includes/favicons.html` 오버라이드, SVG 주력). (커밋 367679e)
- [x] G2. **사이트/탭 제목 결정** — `_config.yml` `title: 이소연의 기술 블로그`, `tagline: 선택의 근거를 수치로 남기는 백엔드 개발자`. (커밋 367679e)
- [x] G3. **"블로그를 시작하며" 글 처리** — `2025-01-02`로 재배치(파일명·date) + `pin: true`. 시리즈 맨 앞으로 가 "시작" 서사가 타임라인과 일치하고, 홈 상단에도 고정. 글 끝에 "여기서 시작하세요" CTA 블록(대표 글 + GitHub) 추가.
- [x] G4. **랜딩 어필 — featured/pin 글 + CTA** — `_drafts/쿠폰-동시성-1-락-사다리.md`를 `_posts/2026-06-25-coupon-concurrency-lock-ladder.md`로 게재 + `pin: true`. 핀 정렬(날짜 역순)상 홈 최상단 = 쿠폰 락 사다리(대표작) → 블로그를 시작하며(안내) 순. CTA는 사이드바 GitHub + 소개글 CTA 블록으로(외부 프로필은 현재 GitHub만). **hero 커스텀은 의도적으로 안 함**(테마 업데이트 유지보수 부담 회피). ✍️ 작성자 메모 주석 2곳 제거. 빌드 무경고 + htmlproofer 통과 확인.
  - 후속(선택): devetym 출시글 나오면 3편째 pin 검토. 외부 프로필 링크 추가 시 CTA·사이드바에 반영.

## H. 콘텐츠 후속
- [ ] H1. **"성능 사다리" 글** — 쿠폰 동시성 글(`coupon-concurrency-lock-ladder`)이 본문 마지막에서 공개적으로 예고한 후속편. v3(DB 행 락)를 출발선에 놓고 이번엔 *성능*(점유 시간·대기 189건·p95 2630ms)을 다룸. "커넥션 풀을 늘리면 빨라지나"라는 직관을 데이터로 검증("톨게이트가 느리면 차선 늘려도 차만 쌓인다"). 출처: high-traffic-performance-tuning.

## 후속 (선택)
- [x] 아바타 이미지 추가 (`_config.yml`의 `avatar`) — `assets/img/avatar.png`
- [ ] 댓글 시스템 (giscus 추천) 연동
- [x] 로컬 미리보기용 Ruby 3.x 설치 (Homebrew `ruby@3.4`)
- [ ] Google 검색 등록 (`webmaster_verifications`)
