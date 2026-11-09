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
- 테이블이 비어 있으면 처음 한 번 기본 데이터(11월 블프 일정: 29CM·W컨셉·무신사, 11/8 와인빛 게시물과 성과, MICHIKO LONDON 반응, 릴스 아이디어 3개)를 넣는다.
- 에이블리·지그재그 블프 일정은 수진 요청으로 뺐다. `cleanup()`이 이미 들어간 두 일정을 한 번만 지우고 settings에 `cleanup1:true`를 남긴다.
- 6월에 올린 게시물 3개(그릇·샌들·키치 소품)는 수진 요청으로 뺐다. `cleanup2()`가 이미 들어간 세 포스트를 제목 기준으로 한 번만 지우고 settings에 `cleanup2:true`를 남긴다.
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

1. **캘린더**: 월간(일~토), 오늘·주말 표시. 상단 스위치로 "포스트" / "행사 일정"을 각각 켜고 끌 수 있다(localStorage `moi-planner-calshow`에 기억). 일정은 이어진 막대(색은 컬러피커+HEX+즐겨찾기 3색), 포스트는 4:5 썸네일(사진 없으면 칩). 빈 칸 더블클릭 → 그 날짜로 포스트. 상단에 이번 주 업로드 n/목표, 예정 수, 오늘 추천 업로드 시간.
2. **콘텐츠 보드**: 아이디어 → 제품 찾는 중 → 제작 중 → 업로드 완료, 카드 드래그로 단계 이동. 포스트 모달: 주제/형식/카테고리/상태/날짜/시간(추천 시간 안내), 표지 문구 후보 3개(하나 선택), 도입 문구, 사진, 제품 목록(목록 붙여넣기·스튜디오용 복사·캡션 복사), 해시태그, 메모, 성과.
3. **아이디어**: 빠른 추가, 카테고리 필터, "콘텐츠로 만들기" → 보드로 이동(아이디어는 삭제).
4. **브랜드**: 포스트 제품에서 브랜드 자동 집계(업로드 완료 기준 소개 횟수·마지막 소개일) + 브랜드 반응 기록. 찜한 제품 목록(선택 → 스튜디오용 복사 / 포스트 만들기).
5. **성과**: 업로드 완료 포스트 기준 타일, 저장률 막대, 카테고리·형식별 평균, 정렬 가능한 표.
6. **메모**: 포스트잇 보드, 자동 저장, 고정.

공통: 상단 전체 검색, 캐러셀 스튜디오 바로가기(https://moi-collects.github.io/carousel-studio/), 설정(서명, 주간 목표, 백업, 로그아웃).

## 자동화 (예약 작업)

예약 작업은 수진 컴퓨터의 Claude 데스크톱 앱 내장 브라우저로 플래너(https://moi-collects.github.io/moi-planner/, 로그인 유지됨)를 열고, 페이지의 `window.moiAPI`로 데이터를 읽고 쓴다. 화면 클릭으로 입력하지 않는다.

- `moiAPI.status()` → {mode, loggedIn, count, today}. loggedIn이 false면 로그인 풀림 → 수진에게 알리고 멈춘다.
- `moiAPI.list(kind)`, `get(id)`, `add(kind,data)`, `update(id,patch)`(data에 얕게 합침), `remove(id)`, `settings()`
- **moiAPI는 지우거나 이름을 바꾸지 않는다.** 예약 작업이 이걸로 동작한다.

자동화가 쓰는 추가 필드 (화면에는 안 보임)
- post: `metricsAt`(성과 읽은 시각 ISO), `metricsStage`(1=1일차, 2=3일차), `auto`(인스타에서 찾아 자동 생성한 포스트면 true)
- idea: `source:'morning'`, `themeDate`(아침 테마 날짜) — 같은 날짜 테마는 한 번만 넣는다
- event: `auto:true`(행사 일정 갱신 작업이 넣은 것)
- memo: 주간 리포트는 `report:'YYYY-MM-DD'`(그 주 일요일)
- brand: 반응은 `reactions` 플래그 + memo 끝에 "M/D 반응 내용" 한 줄 추가

예약 작업 목록
- moi.collects 평일 테마 / 일요일 테마: 아침 테마 메시지 + Google Drive에 `moi-theme-inbox YYYY-MM-DD` 텍스트 파일 저장. "후보 찾아줘" 결과는 찜한 제품에도 저장.
- moi.collects 인스타 체크 (매일 밤): 올린 게시물 ↔ 플래너 포스트 연결, 성과 1일차·3일차, 브랜드 반응, Drive 테마 → 아이디어
- moi.collects 주간 리포트 (일요일 오후): 리포트 메모 + 다음 주 업로드 계획(status idea 포스트)
- moi.collects 행사 일정 갱신 (월요일 오전): 플랫폼 세일 일정 추가·수정

## 형식 규칙

- 스튜디오 붙여넣기: `브랜드 | 계정 | 제품명 | 가격 | 컬러` (가격은 숫자만, 정가 기준)
- 캡션: 표지 문구(줄바꿈→공백, 끝 마침표 제거) / 도입 문구 / `✽ 브랜드 (@계정)` + `제품명 / 컬러 | 17,500₩` / `__` + 서명 / 해시태그
- 추천 업로드 시간: 일 밤 8~10시, 월 저녁 6~8시, 화 오후 3~5시·저녁 7시, 수 낮 12시·저녁 6시, 목 아침 7~9시, 금·토 밤 9~10시(반응 약함)

## 수정 후

- 스크립트 추출 후 `node --check`로 문법 확인.
- 확인 없이 디자인을 바꾸지 않는다.
