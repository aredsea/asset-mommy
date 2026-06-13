# Asset Mommy

RisuAI 플러그인 — NovelAI 에셋 생성·관리 + 외견 추출기.
[Asset maid 0.9.1](https://arca.live/b/characterai) (NovelAIAutoAsset)에서 파생, iOS 안정화 패치 포함.

## 설치 (iOS Safari / RisuAI 웹)

1. RisuAI → 설정 → **플러그인** → **플러그인 가져오기**
2. URL 또는 파일 선택:
   - **자동업데이트 URL**: `https://raw.githubusercontent.com/aredsea/asset-mommy/main/asset-mommy.js`
   - **직접 다운로드**: [asset-mommy.js](https://github.com/aredsea/asset-mommy/raw/main/asset-mommy.js)
3. import 완료 후 RisuAI 새로고침

자동업데이트는 RisuAI 플러그인 매니저에서 "Check for updates" 시 동작합니다 (silent 아님 — 사용자 수동 트리거).

## 핵심 기능

### 외견 추출기 (lb-xnai.lb.extra)

캐릭터 로어북·에셋을 LLM으로 분석해 danbooru 태그 형식의 외견 정보를 `lb-xnai.lb.extra` 항목으로 저장합니다.

- 출력 형식: `## Character Image Tags` → `### 캐릭터명` → `Real World:` / `Avatar:`
- 다른 플러그인(WygLoreLeaf, 삽화 모듈 등)과의 표준 lorebook 이름 호환
- 추출 모달은 플러그인 설정 → 분석 도구 섹션에서 진입

### NovelAI 에셋 생성·관리 (원본 기능)

원본 Asset maid의 모든 기능 유지. 자세한 사용법은 [원본 가이드](https://arca.live/b/characterai)를 참고하세요.

## Asset maid 0.9.1에서의 주요 변경 (Asset Mommy 1.0.0)

iOS RisuAI 환경에서 발견한 문제들의 패치:

- **lb-xnai.lb.extra 저장 안정화**: `setCharacter` → `setCharacterToIndex` → `setDatabase(Lite)` 다중 경로로 autosave `$effect` 트리거 보장
- **항목 포맷 spec_v3 호환**: `name`, `keys`, `enabled`, `extensions` 등 charx spec_v3 필드 모두 채움 (없으면 RisuAI UI가 "extra" 폴더로 오인하던 버그)
- **중복 누적 방지**: 추출 재실행 시 기존 `lb-xnai.lb.extra` 항목을 모두 제거 후 1개만 추가 (stale snapshot으로 인한 누적 버그 해결)
- **캐릭터 enrichment**: `getDatabase`가 stub만 반환하는 케이스 → `getCharacterFromIndex`로 `additionalAssets`/`globalLore` 풀 데이터 fetch
- **JSON 파서 견고화**: balanced brace counting + string escape 인식. 큰 응답에서 "Unexpected non-whitespace character after JSON" throw 방지
- **캐시 무효화**: DB 캐릭터 ID 시그니처 비교로 RisuAI 측 삭제/추가 자동 감지 → 캐시 자동 재로드
- **저장 버튼 dirty 우회**: 수동 "저장" 클릭 시 dirty 여부 무관하게 saveConfig 실행
- **에셋 정규화 양식 호환**: `additionalAssets`가 tuple `[name, key, ext]` 또는 객체 `{name, uri, ext}` 양쪽 처리

## 알려진 이슈

- 에셋 인식: iOS 빌드의 일부 환경에서 `Risu.getCharacterFromIndex`가 풀 데이터를 반환하지 않을 가능성 있음. lbx14의 진단 로그 (`[NAA-DB]`, `[NAA-ENRICH]`)로 실제 상태 확인 가능.
- 모바일 UI: "기존 프롬프트 덮기" 버튼이 좁은 화면에서 짤림. 모듈 선택 리스트 가독성 낮음. CSS media query 미완.
- 메타데이터 분석: NovelAI 메타데이터 없는 이미지는 "없음"으로 표시됨 (정상 동작 — 향후 LLM 추론으로 보완 예정).

## 라이선스 / 원본 attribution

- 원본 저자: NovelAIAutoAsset / Asset maid 0.9.1 (arca.live 라이트보드 채널)
- 이 fork는 개인 사용 목적의 패치 모음. 원본 저자의 의도와 다를 수 있음.
- 원본 저자 측 정식 업데이트가 나오면 그쪽을 권장합니다.

## 버전

- `1.0.0` (2026-06-13): 초기 릴리즈. Asset maid 0.9.1 + lbx10~16 통합.

버전 비교는 RisuAI 내부 `compareVersions` (dot-separated integers, 단순 numeric 비교)을 따릅니다. pre-release suffix(`1.0.0-beta` 등)는 비교 깨지므로 사용 안 함.
