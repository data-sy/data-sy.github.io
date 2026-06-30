# 블로그 로드맵

Jekyll + Chirpy 기반 개발 블로그 (`https://data-sy.github.io`) — 학습 자료 공유 + 구직 겸용

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
- [ ] E3. "Robin" 가명 → 이소연 통일 (공유할 파일 한정, 11개 중 대상만)
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

## G. 브랜딩·홈 정돈
- [ ] G1. **파비콘 교체** — 현재 Chirpy 기본(곤충 모양). realfavicongenerator 세트 → `assets/img/favicons/` 또는 이모지 SVG. 어떤 아이콘/이모지로 할지 결정 필요.
- [ ] G2. **사이트/탭 제목 결정** — `_config.yml` `title`(사이드바 대제목 = 브라우저 탭, 분리 불가). 후보: `이소연` 유지 / `이소연의 개발기` / `Robin's Blog`. ROADMAP E3(Robin→이소연 통일)와 상충 가능. **다른 개발 블로그 제목 관례 레퍼런스 검토 중.**
- [ ] G3. **"블로그를 시작하며"(2026-06-24) 글 처리 결정** — 타임라인 끝에 "시작" 제목으로 있어 시작점(2025)과 모순. ① 홈 상단 pin(pin:true) ② 2025-01로 재배치 ③ 유지. **홈 화면 정돈·CTA 논의(G4) 때 함께 결정.**
- [ ] G4. **랜딩 어필 — featured/pin 글 + CTA** — 첫 화면에서 "어떤 개발자인지/뭘 먼저 읽을지" 전달. 대표 글 2~3편 pin 큐레이션, CTA 버튼(포트폴리오/GitHub/대표 글) 문구·대상, hero 영역 둘지 여부. Chirpy 기본엔 hero/CTA 없어 커스텀 필요할 수 있음 — 범위 산정 후 진행.

## 후속 (선택)
- [x] 아바타 이미지 추가 (`_config.yml`의 `avatar`) — `assets/img/avatar.png`
- [ ] 댓글 시스템 (giscus 추천) 연동
- [x] 로컬 미리보기용 Ruby 3.x 설치 (Homebrew `ruby@3.4`)
- [ ] Google 검색 등록 (`webmaster_verifications`)
