# 블로그 구축 체크리스트

Jekyll + Chirpy 기반 개발 블로그 (`https://data-sy.github.io`) — 학습 자료 공유 + 구직 겸용

## A. 레포·뼈대 ✅
- [x] A1. Chirpy 스타터 템플릿으로 `data-sy.github.io` 레포 생성 + 로컬 클론
- [x] A2. `setup/chirpy` 작업 브랜치 생성
- [x] A3. CHECKLIST.md 작성

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

## 후속 (선택)
- [ ] 아바타 이미지 추가 (`_config.yml`의 `avatar`)
- [ ] 댓글 시스템 (giscus 추천) 연동
- [ ] 로컬 미리보기용 Ruby 3.x 설치
- [ ] Google 검색 등록 (`webmaster_verifications`)
