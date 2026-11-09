# moi.collects 플래너 작업 메모

수진의 인스타 아카이빙 계정 moi.collects 운영용 개인 플래너다.
Branding Planner(moineauseoul/branding-planner)와 구조를 비슷하게 맞추고, 색은 흰/회색/검정 미니멀로 만들었다.

## 꼭 지킬 것

- **모든 답변은 한국어로만 한다.** 영어로 답하지 않는다.
- **디자인은 수진이 정한다.** 임의로 레이아웃·색·기능을 바꾸지 말고, 요청받은 것만 바꾼다. 새 기능은 제안만 하고 확인을 받는다.
- 탭 안의 작은 요소(칩, 강조색 등)는 수진이 나중에 다시 정하기로 했다. 지금 값은 임시다.
- 컴퓨터 버전 먼저 완성 → 그다음 모바일 UI를 따로 다시 만든다.

## 파일

- `index.html` 한 파일 (HTML/CSS/JS). 프리텐다드 서브셋이 `@font-face`에 base64로 들어 있다. **base64 줄은 건드리지 않는다.**
- `supabase.sql` 테이블·권한·실시간 설정
- 외부 스크립트: supabase-js 2.45.4 (jsDelivr UMD)

## 저장 방식

- 스크립트 맨 위 `SUPABASE_URL`, `SUPABASE_ANON_KEY`가 비어 있으면 **브라우저 저장 모드**(localStorage `moi-planner-v1`).
- 채워져 있으면 **Supabase 모드**: 이메일/비밀번호 로그인 → `moi_items` 테이블 읽기/쓰기, Realtime으로 기기 간 실시간 동기화.
- 테이블 하나(`moi_items`)에 `kind` + `data(jsonb)`로 모든 항목을 넣는다. RLS로 본인(owner)만 읽고 쓸 수 있다.
- 테이블이 비어 있으면 처음 한 번 기본 데이터(11월 블프 일정, 기존 게시물 4개와 성과, MICHIKO LONDON 반응, 릴스 아이디어 3개)를 넣는다.
- 설정 → 내보내기/가져오기(JSON)로 백업, 브라우저 저장 모드 → Supabase 이전도 가져오기로 한다.

### kind별 data
- `event` 일정: title, start, end, color(HEX), memo
- `post` 포스트: title, format(carousel|reels), category, status(idea|sourcing|making|done), date, time, covers[3], coverPick, intro, hashtags, memo, images[](540px JPEG data URL, 최대 10장), products[{brand,handle,name,color,price,link}], metrics{reach,views,saves,likes,comments,shares,profile,follows}, link
- `idea` 아이디어: title, category, memo, link
- `brand` 브랜드 기록: name, handle, site, reactions{like,comment,follow,dm,collab}, memo
- `saved` 찜한 제품: brand, handle, name, color, price, link, category, memo
- `memo` 메모: text, pinned
- `settings` (고정 id 00000000-0000-4000-8000-000000000001): signature, weeklyGoal, favColors[3]

## 탭

1. **캘린더**: 월간(일~토), 오늘·주말 표시. 일정은 이어진 막대(색은 컬러피커+HEX+즐겨찾기 3색), 포스트는 4:5 썸네일(사진 없으면 칩). 빈 칸 더블클릭 → 그 날짜로 포스트. 상단에 이번 주 업로드 n/목표, 예정 수, 오늘 추천 업로드 시간.
2. **콘텐츠 보드**: 아이디어 → 제품 찾는 중 → 제작 중 → 업로드 완료, 카드 드래그로 단계 이동. 포스트 모달: 주제/형식/카테고리/상태/날짜/시간(추천 시간 안내), 표지 문구 후보 3개(하나 선택), 도입 문구, 사진, 제품 목록(목록 붙여넣기·스튜디오용 복사·캡션 복사), 해시태그, 메모, 성과.
3. **아이디어**: 빠른 추가, 카테고리 필터, "콘텐츠로 만들기" → 보드로 이동(아이디어는 삭제).
4. **브랜드**: 포스트 제품에서 브랜드 자동 집계(업로드 완료 기준 소개 횟수·마지막 소개일) + 브랜드 반응 기록. 찜한 제품 목록(선택 → 스튜디오용 복사 / 포스트 만들기).
5. **성과**: 업로드 완료 포스트 기준 타일, 저장률 막대, 카테고리·형식별 평균, 정렬 가능한 표.
6. **메모**: 포스트잇 보드, 자동 저장, 고정.

공통: 상단 전체 검색, 캐러셀 스튜디오 바로가기(https://moi-collects.github.io/carousel-studio/), 설정(서명, 주간 목표, 백업, 로그아웃).

## 형식 규칙

- 스튜디오 붙여넣기: `브랜드 | 계정 | 제품명 | 가격 | 컬러` (가격은 숫자만, 정가 기준)
- 캡션: 표지 문구(줄바꿈→공백, 끝 마침표 제거) / 도입 문구 / `✽ 브랜드 (@계정)` + `제품명 / 컬러 | 17,500₩` / `__` + 서명 / 해시태그
- 추천 업로드 시간: 일 밤 8~10시, 월 저녁 6~8시, 화 오후 3~5시·저녁 7시, 수 낮 12시·저녁 6시, 목 아침 7~9시, 금·토 밤 9~10시(반응 약함)

## 수정 후

- 스크립트 추출 후 `node --check`로 문법 확인.
- 확인 없이 디자인을 바꾸지 않는다.
