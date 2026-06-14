# Asset Mommy

RisuAI 플러그인 — NovelAI 에셋 생성·관리 + 캐릭터 외견 추출기. **sns_nai**와 연동됩니다. iOS(아이폰) 최적화.

## 설치 (iOS Safari / RisuAI 웹)

1. RisuAI → 설정 → **플러그인** → **플러그인 가져오기**
2. URL: `https://raw.githubusercontent.com/aredsea/asset-mommy/main/asset-mommy.js`
3. import 완료 후 RisuAI 새로고침

자동업데이트는 RisuAI 플러그인 매니저에서 동작합니다.

## 버전 규칙

`MAJOR.MINOR.PATCH` — 매 릴리스마다 PATCH를 올리고, PATCH가 10이 되면 MINOR로 올림(PATCH는 0으로).
예) `1.2.5 → 1.2.6 → … → 1.2.9 → 1.3.0`. sns_nai와 동일한 규칙.
(RisuAI `compareVersions`는 dot-separated integer 비교라 이 규칙이 안전합니다. suffix 미사용.)

## 핵심 기능

### 외견 추출기 (lb-xnai.lb.extra)

캐릭터 로어북·에셋·설명을 LLM으로 분석해 danbooru 태그 형식의 외견 정보를 `lb-xnai.lb.extra` 항목으로 저장합니다.

- 출력 형식: `## Character Image Tags` → `### 캐릭터명` → 의상/상황별 라벨(`Casual:` / `Maid:` / `Sleep:` / `NSFW:` 등) + danbooru 태그
- 에셋 파일명의 의상 토큰을 추출해 의상별로 태그를 분리
- 단일 캐릭터는 헤딩을 `char.name` 전체(한글/영문 이중표기)로 강제 → sns_nai 등 소비자가 정확히 매칭
- 추출 모달은 플러그인 설정 → 분석 도구 섹션에서 진입

### NovelAI 에셋 생성·관리

채팅 이미지 토큰 → NovelAI 에셋 생성·교체. 캐릭터별 프롬프트·의상 레퍼런스·줌 갤러리(재생성).

## 모바일·연동 개선

iOS RisuAI 환경 최적화:

- **모바일 디자인 전면**: 검정/녹색 톤, 풀스크린 모달, 터치 44px, 단일 컬럼, 한국어 keep-all wrap. element-level inline-style 강제로 cascade 무관 적용.
- **lb-xnai.lb.extra 저장 안정화**: `setCharacter`/`setCharacterToIndex` 다중 경로 + spec_v3 호환 필드. setDatabase 전체 덮어쓰기(플러그인 전체 삭제 유발) 제거.
- **중복 누적 방지**: 추출 재실행 시 기존 항목 제거 후 1개만.
- **캐릭터 enrichment**: `getCharacterFromIndex`/`getCharacter`로 `additionalAssets`/`globalLore` 풀 데이터 fetch.
- **설명→로어북**: 싱글 캐릭터 봇은 description을 합성 로어북 항목으로 — 분석/매칭 대상 확보.
- **줌 갤러리 모바일**: 세로 스택 + 닫기 노출 + recolor.
- **JSON 파서 견고화**, **캐시 무효화**, **에셋 tuple/객체 양식 호환**.

## sns_nai 연동

```
Asset Mommy: 외견 추출 → lb-xnai.lb.extra (### 다온비 (Da Onbi))
        ↓
sns_nai: 이미지 태그 사전 스캔 → 매칭 → profile.imageTags 갱신 → NAI 이미지 생성
```

## 버전 이력

- `1.3.0`: ★SNS 레퍼런스 — 캐릭터 자체 이미지 픽커★ 플러그인 설정 → 분석 도구에 "SNS 레퍼런스 지정…" 추가. 줌 갤러리(아이폰에서 불안정)를 거치지 않고, 캐릭터의 아바타·additionalAssets를 그리드로 보여줘 탭 한 번으로 레퍼런스 지정/해제. `Risu.readImage`로 바이트를 직접 읽어 압축 후 `lb-xnai.ref` 로어북에 저장(scoped write). sns_nai가 이를 NAI director reference로 사용.
- `1.2.9`: ★sns_nai 연동 — SNS 캐릭터 레퍼런스 지정★ 줌 갤러리 상단에 "SNS 레퍼런스" 버튼 추가. 현재 보고 있는 이미지를 캐릭터 레퍼런스로 지정하면 캐릭터 로어북(`lb-xnai.ref`)에 압축 저장되고, sns_nai가 이를 NAI director reference로 사용해 그림체·캐릭터를 유지한 채 상황만 바뀐 이미지를 생성합니다(NAI V4.5 모델 필요, 캐릭터당 1장). 저장은 scoped write만 사용.
- `1.2.8`: 줌 갤러리 버건디 제거 + 상단바 오버플로/사이즈텍스트 클립 수정. iOS RisuAI 아이프레임이 데스크탑 폭으로 보고돼 `@media(max-width:600px)` 모바일 규칙이 안 먹던 문제 → 뷰포트 무관 `!important` 오버라이드로 검정/녹색 테마 강제, 닫기 버튼 줄바꿈 처리, 사이즈 셀렉트 폭 자동. display-name에 버전 표기(목록에서 로드된 버전 확인용).
- `1.2.7`: ★NAI 이미지 생성 핵심 수정★ NAI 호출 전송 계층을 항상 `risuFetch`(=globalFetch)로 강제. 기존엔 iOS UA 정규식이 맞을 때만 risuFetch를 쓰고 아니면 `nativeFetch`로 떨어졌는데, 아이폰 RisuAI 네이티브 WebView의 UA가 이 검사와 안 맞아 이미지 생성이 통째로 실패했음. 본체 "기타 봇>이미지 생성"이 NAI를 정상 호출하는 경로가 바로 globalFetch라 모든 플랫폼에서 동작. (Vertex OAuth 토큰 교환은 risuFetch 비호환이라 기존 경로 유지)
- `1.2.5`: sns_nai 연동(헤딩 char.name 강제) + 줌 갤러리 모바일 + 의상별 추출.
- `1.0.0`: 초기 릴리즈.
