# 블로그 구축 체크리스트

Jekyll + Chirpy 기반 개발 블로그 (`https://data-sy.github.io`)

## A. 레포·뼈대
- [x] A1. Chirpy 스타터 템플릿으로 `data-sy.github.io` 레포 생성 + 로컬 클론
- [x] A2. `setup/chirpy` 작업 브랜치 생성
- [x] A3. CHECKLIST.md 작성

## B. 설정
- [x] B1. `_config.yml` — url, title, tagline, lang(ko-KR), timezone(Asia/Seoul)
- [x] B2. 소셜/프로필 — GitHub 링크, 트위터 제거
- [x] B3. 스타터 데모 정리 (스타터 자체가 최소 구성)

## C. 콘텐츠
- [x] C1. About 페이지 (자기소개·프로젝트 링크)
- [x] C2. 첫 글 작성 (`_posts/2026-06-24-블로그를-시작하며.md`)

## D. 배포·검증
- [x] D1. `setup/chirpy` push
- [ ] D2. `main` 병합
- [ ] D3. GitHub Pages 소스 = GitHub Actions 활성화
- [ ] D4. Actions 빌드 성공 + 사이트 라이브 확인

## 후속 (선택)
- [ ] 아바타 이미지 추가 (`_config.yml`의 `avatar`)
- [ ] 댓글 시스템 (giscus 추천) 연동
- [ ] 로컬 미리보기용 Ruby 3.x 설치 (현재 시스템 Ruby 2.6 — 클라우드 빌드는 무관)
- [ ] Google 검색 등록 (`webmaster_verifications`)
