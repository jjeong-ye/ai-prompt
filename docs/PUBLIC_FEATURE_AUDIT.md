# 공개 기능 전수 조사 보고서

> 조사일: 2026-07-18  
> 기준 파일: `index.html`  
> 비공개 제외 기준: `stage14-public-hidden` 클래스 적용 요소

---

## 1. 무엇을 만들고 싶으세요? (메인 입력)

### 시작 화면
- `home-section` (초기 active 섹션)
- hero-card 안에 textarea와 생성 버튼 배치

### 사용자가 누르는 버튼
- "프롬프트 만들기" 버튼 (`onclick="generatePrompt()"`)
- "거꾸로 생성하기" 바로가기 (`onclick="openSection('reverse')"`)
- 첨부파일 체크박스 (`#main-attach-check`)

### 입력하는 칸
- `#main-input` (textarea, placeholder 순환)
- `#main-attach-purpose` (select, 첨부파일 활용 방식)
- `#main-attach-detail` (input, 특히 확인할 내용)

### 연결된 함수
- `generatePrompt()` — 핵심 프롬프트 생성 엔진
- `analyzeTeacherRequest(input)` — 유형/과목/학년/대상 분석
- `buildOptimizedPrompt(analysis)` — 교육 프롬프트 조립
- `buildUniversalMainPrompt(options)` — 일반 프롬프트 조립
- `detectEducationContext(text)` — 교육/일반 분기
- `detectPromptInjection(text)` — 주입 방지
- `toggleAttachPanel()` — 첨부 패널 토글
- `getMainAttachmentOptions()` — 첨부 옵션 수집

### 생성되는 결과
- `result-section`으로 전환
- `#result-intent` — 의도 해석 표시
- `#result-prompt` (textarea) — 완성 프롬프트 (편집 가능)
- `#result-tip` — 팁/안내
- `#placeholder-status` — 자리표시자 상태

### 빈 입력일 때 동작
- `showToast('만들고 싶은 것을 입력해주세요')` 표시 후 중단
- 결과 화면으로 이동하지 않음

### 뒤로 가기 동작
- 결과 화면의 "이전 화면으로" 버튼 → `goBackFromResult()` → `goHome()`

### 복사 기능
- "복사하기" 버튼 → `requestPromptCopy()`
- 자리표시자 남아있으면 확인 모달 표시 → `confirmPromptCopy()` 또는 `focusFirstUnresolvedPlaceholder()`
- `navigator.clipboard.writeText()` 사용, 실패 시 `execCommand('copy')` 폴백

### 개인정보 저장 여부
- `saveToRecent()` 호출 → `createSafeRecentEntry()` → 기능명/시간만 저장
- 원문 텍스트는 localStorage에 저장하지 않음 ✅

### 오류 가능성
- Enter 키로 생성 시 IME 조합 중 오작동 방지 (`e.isComposing || e.keyCode===229`) ✅
- placeholder 순환 타이머가 focus 시 정상 중단되는지 확인 필요
- 생기부 관련 키워드 감지가 과도하면 일반 요청이 생기부로 잘못 분기될 수 있음

### 실제 테스트 필요 항목
- [ ] 빈 입력 → 토스트 표시 확인
- [ ] Enter 키 생성, Shift+Enter 줄바꿈 구분
- [ ] 교육 요청과 일반 요청 분기 정상 여부
- [ ] 첨부파일 체크 → 패널 표시 → 프롬프트에 첨부 지침 포함
- [ ] 프롬프트 주입 패턴 차단 확인
- [ ] 결과 화면에서 자리표시자 상태 표시


---

## 2. 거꾸로 생성하기

### 시작 화면
- `reverse-section` (독립 섹션)
- 홈 hero-card 하단의 "거꾸로 생성하기" 버튼으로 진입

### 사용자가 누르는 버튼
- "거꾸로 생성하기" 진입 버튼 (`onclick="openSection('reverse')"`)
- "파일 선택" (label → file input)
- "새 주제로 결과물 생성" (`onclick="reverseGenerateStandalone()"`)
- "처음으로" 뒤로가기 버튼

### 입력하는 칸
- `#reverse-standalone-input` (textarea, 기존 자료 붙여넣기)
- `#reverse-new-topic` (input, 새로 만들 주제)
- 파일 드래그앤드롭 영역 (file-drop-zone)

### 연결된 함수
- `reverseGenerateStandalone()` — 메인 실행
- `doReverseGenerate(input, newTopic)` — 프롬프트 조립
- `handleDragOver/handleDragLeave/handleFileDrop` — 드래그앤드롭
- `handleFileSelect(event)` — 파일 선택
- `readFileContent(file)` — txt/md/csv/html 파일 읽기

### 생성되는 결과
- `result-section`으로 전환
- 프롬프트: 기존 자료 분석 + 새 주제 적용 지침
- 의도 해석: "기존 자료와 같은 구조로 [주제]의 새 최종 결과물을 생성합니다"

### 빈 입력일 때 동작
- textarea 비어있으면: `showToast('기존 자료를 붙여넣거나 파일을 업로드해주세요')`
- 새 주제 비어있으면: `showToast('새로 만들 주제 하나만 입력해주세요.')` + focus 이동

### 뒤로 가기 동작
- "처음으로" 버튼 → `goHome()` → home-section 표시

### 복사 기능
- 결과 화면의 공통 복사 버튼 사용 (기능 8번과 동일)

### 개인정보 저장 여부
- `doReverseGenerate()`에서 `setResultMode('REVERSE')` 설정
- `saveToRecent('거꾸로 생성 프롬프트', '')` — 기능명만 저장 ✅
- 기존 자료 원문은 저장하지 않음 ✅

### 오류 가능성
- 파일 업로드 시 지원 확장자 외 파일 → 토스트 안내 정상 처리
- 대용량 파일 읽기 시 브라우저 멈춤 가능 (FileReader 동기 아닌 비동기라 가능성 낮음)
- HTML 파일의 textContent 추출이 복잡한 DOM에서 불완전할 수 있음

### 실제 테스트 필요 항목
- [ ] 텍스트만 입력 + 주제 입력 → 정상 결과
- [ ] 파일 업로드 → textarea에 내용 로드 확인
- [ ] 드래그앤드롭 동작 확인
- [ ] 주제 없이 생성 시도 → 토스트 + focus
- [ ] 결과 화면에서 뒤로 가기 → reverse-section 복귀
- [ ] Ctrl+Enter 단축키 동작


---

## 3. 생기부 작성

### 시작 화면
- `record-section` → "생기부 작성" 탭 (`#record-write-panel`)
- 헤더 내비게이션 "생기부 제작소" 클릭으로 진입

### 사용자가 누르는 버튼
- 학년 탭 (1학년/2학년/3학년) — `selectRecordGrade()`
- "완성형 생기부 프롬프트 만들기" (`onclick="generateUnifiedRecordPrompt()"`)
- 활동자료 파일 선택/드래그앤드롭
- 학교 추가 기준 details 토글
- 출력 방식 선택 (대화창/표/Excel)

### 입력하는 칸
- `#record-item-select-unified` (select, 생기부 항목 선택)
- `#free-semester-subtype` (select, 자유학기 세부 영역)
- `#record-input` (textarea, 관찰·활동 자료)
- `#record-target-mode` (select, 처리 대상)
- `#record-sentence-count-unified` (number, 학생별 결과 개수)
- `#record-style-example` (textarea, 문체 참고 예시문)
- `#record-extra-unified` (input, 추가 조건)
- `#record-output-format` (select, 출력 방식)
- 학교 추가 기준 체크박스 3개 + textarea

### 연결된 함수
- `generateUnifiedRecordPrompt()` — 핵심 생성 함수
- `onUnifiedRecordItemChange()` — 항목 변경 처리
- `selectRecordGrade(grade)` — 학년 전환 (자율·자치/자율 분기)
- `onFreeSemesterSubtypeChange()` — 자유학기 세부 영역
- `renderRecordMeta(type)` — 메타 정보 렌더링
- `updateRecordInputGuide()` — 항목별 입력 안내
- `buildRecordSafetyRules()` — 안전 규칙 생성
- `buildRecordWritePrinciples()` — 작성 원칙 생성
- `buildRecordItemModule()` — 항목별 모듈 생성
- `getActiveSchoolQuickSettings()` — 학교 기준 수집
- `handleRecordPromptActivityFileDrop/Select` — 파일 처리

### 생성되는 결과
- `result-section`으로 전환, `RECORD_DRAFT` 모드
- 프롬프트: 정책 헤더 + 항목 규칙 + 관찰 자료 + 안전 규칙 + 출력 지침
- 팁: 교사 검토 필수 안내

### 빈 입력일 때 동작
- 텍스트와 파일 모두 없으면: `showToast('학생별 관찰 근거가 담긴 활동자료를 직접 입력하거나 파일을 선택해주세요.')`

### 뒤로 가기 동작
- "처음으로" → `goHome()`
- 결과 화면 → `goBackFromResult()` → record-section 복귀

### 복사 기능
- 결과 화면 공통 복사 기능 사용

### 개인정보 저장 여부
- `saveToRecent('생기부 [항목] 프롬프트', '')` — 기능명만 저장 ✅
- 관찰 자료 원문, 문체 예시문, 파일 내용은 저장하지 않음 ✅
- 학교 추가 기준(체크박스 상태)은 localStorage에 저장 (개인정보 아님) ✅

### 오류 가능성
- 자유학기활동 선택 시 세부 영역 패널 표시/숨김 타이밍
- 3학년 선택 시 "자율활동"으로 전환되는데, traffic 검토와 연동 시 모달 표시
- Excel 출력 모드에서 양식 파일 선택 UI의 실제 활용은 AI에 위임 (사이트 자체는 파일 전송 안 함)

### 실제 테스트 필요 항목
- [ ] 각 학년별 항목명 변화 (1·2학년: 자율·자치활동, 3학년: 자율활동)
- [ ] 자유학기활동 선택 → 세부 영역 드롭다운 표시
- [ ] 관찰 자료만 입력 → 정상 프롬프트 생성
- [ ] 파일만 선택 → 프롬프트에 파일 첨부 안내 포함
- [ ] 빈 입력 → 토스트 표시 확인
- [ ] 학교 기준 저장/로드 동작
- [ ] Ctrl+Enter 단축키
- [ ] 출력 방식별 프롬프트 차이 확인


---

## 4. 검토·다듬기 — 이거 써도 되나요?

### 시작 화면
- `record-section` → "검토·다듬기" 탭 (`#record-review-panel`)
- 카드 목록 중 "이거 써도 되나요?" 카드 클릭으로 진입
- (나머지 3개 카드: 문장 CPR, 최종 점검, 편향 검사는 `stage14-public-hidden`)

### 사용자가 누르는 버튼
- "이거 써도 되나요?" 카드 (`onclick="openRecordReviewTool('traffic')"`)
- "AI 정밀검토 프롬프트 만들기" (`onclick="runTrafficIntegratedCheck()"`)
- 항목 변경 시 모달의 "항목 변경"/"취소" 버튼
- 파일 선택/드래그앤드롭
- "프롬프트 복사" (`onclick="copyTrafficUnifiedPrompt()"`)
- "새 검사 시작" (`onclick="confirmResetTrafficCheck()"`)
- 분할 프롬프트 이전/다음 (`trafficChunkPrev/Next`)
- "검토·다듬기로 돌아가기" 뒤로 버튼

### 입력하는 칸
- `#traffic-item-select` (select, 생기부 항목)
- `#traffic-term-input` (textarea, 검토할 문장)
- `#traffic-file-input` (file input, 일괄 검사)
- `#traffic-sheet-select` (select, 시트)
- `#traffic-column-select` (select, 열)
- `#traffic-column-input` (input, 열 직접 입력)
- `#traffic-student-column-select` (select, 학생 구분 열)
- 학교 추가 기준 체크박스/textarea

### 연결된 함수
- `openRecordReviewTool('traffic')` — 패널 표시
- `runTrafficIntegratedCheck()` — 통합 실행
- `buildTrafficAIPrompt(opts)` — AI 정밀검토 프롬프트 생성
- `evaluateTrafficSentence(text, item)` — 문장 로컬 판정 (입력 품질)
- `isUnreadableInput(text)` — 판독 불가 입력 검사
- `detectTrafficTerms(text, item)` — 금지 표현 로컬 탐지
- `generateTrafficUnifiedPrompt(item, rows)` — 묶음 프롬프트 생성
- `splitRowsByDualLimit(rows, maxRows, maxChars)` — 분할 로직
- `readTrafficFile(file)` — 파일 읽기 (XLSX/CSV/TSV/TXT/PDF)
- `readTrafficPdfFile(file)` — PDF 텍스트 추출
- `onTrafficItemChange()` — 항목 변경 모달
- `confirmResetTrafficCheck()` — 초기화 확인
- `copyTrafficUnifiedPrompt()` — 프롬프트 복사

### 생성되는 결과
- 프롬프트: 7열 마크다운 표 출력 지시 + 수정본 복사 블록 + 최종 감사
- 처리 안내: 직접 입력 건수, 파일 행 수, 분할 수
- AI 사이트 바로가기 버튼 (ChatGPT/Gemini/Claude/Copilot)

### 빈 입력일 때 동작
- 텍스트도 파일도 없으면: 결과 영역에 안내 메시지 표시
- 파일 있지만 열 미선택: "검토할 열을 선택해주세요" 안내
- 입력이 판독 불가(자모 나열 등): "문장으로 인식하기 어려운 입력입니다" 안내

### 뒤로 가기 동작
- "검토·다듬기로 돌아가기" → `closeRecordReviewTool()` → 카드 목록 복원
- 결과 화면에서 뒤로 → `goBackFromResult()` → record-section + traffic-check-panel

### 복사 기능
- `copyTrafficUnifiedPrompt()` — `navigator.clipboard.writeText()` 사용
- 분할된 경우 현재 보이는 묶음의 프롬프트만 복사

### 개인정보 저장 여부
- 검토 문장, 파일 내용은 메모리에만 존재 ✅
- localStorage에 저장하지 않음 ✅
- 학교 기준 설정만 항목별로 localStorage 저장 (개인정보 아님) ✅

### 오류 가능성
- XLSX 라이브러리 로드 실패 시 폴백 (CSV/TSV/TXT만 가능) — 안내 표시
- PDF.js 로드 실패 시 PDF 입력 불가 — 안내 표시
- 항목 변경 시 작업 상태가 있으면 모달 표시 — 취소 시 이전 상태 복원 필요
- 대량 행(>10행) 분할 시 프롬프트 이전/다음 내비게이션 정상 동작 확인

### stage14-public-hidden 비공개 요소의 간섭 여부
- 문장 CPR, 최종 점검, 편향 검사 카드는 `stage14-public-hidden`으로 숨김
- 이들의 JS 함수(`generateCPRFromPanel`, `runFinalCheck`, `generateBiasCheckFromPanel`)는 존재하지만 UI 진입점이 숨겨져 호출 불가
- "이거 써도 되나요?" 기능에 간섭하지 않음 ✅

### 실제 테스트 필요 항목
- [ ] 직접 입력 1문장 → 프롬프트 생성 + 복사
- [ ] 직접 입력 여러 줄 → 정상 분리 처리
- [ ] Excel 파일 업로드 → 시트/열 선택 → 프롬프트 생성
- [ ] CSV/TXT 파일 업로드 → 열 선택
- [ ] PDF 파일 → 텍스트 추출 + 열 선택
- [ ] 빈 입력 → 안내 메시지
- [ ] 판독 불가 입력(ㅁㄴㅇㄹ) → 차단 메시지
- [ ] 항목 변경 시 모달 동작 (확인/취소)
- [ ] 분할 프롬프트 이전/다음 내비게이션
- [ ] "새 검사 시작" 초기화 동작
- [ ] Ctrl+Enter 단축키
- [ ] 학교 기준 저장 후 항목 재진입 시 복원


---

## 5. 프롬프트 작성법

### 시작 화면
- `prompt-guide-section`
- 헤더 내비게이션 "프롬프트 작성법" 클릭으로 진입

### 사용자가 누르는 버튼
- "처음으로" 뒤로 버튼 (`onclick="goHome()"`)
- 별도의 인터랙티브 버튼 없음 (정적 콘텐츠)

### 입력하는 칸
- 없음 (읽기 전용 페이지)

### 연결된 함수
- `openSection('prompt-guide')` — 섹션 전환
- `goHome()` — 홈 복귀

### 생성되는 결과
- 정적 콘텐츠 표시:
  - 좋은 프롬프트 공식 (역할+대상+목적+조건+출력 형식+검토 기준)
  - 나쁜 예 vs 좋은 예 비교 박스

### 빈 입력일 때 동작
- 해당 없음 (입력 칸 없음)

### 뒤로 가기 동작
- "처음으로" → `goHome()`

### 복사 기능
- 없음 (정적 안내 페이지)

### 개인정보 저장 여부
- 없음 ✅

### 오류 가능성
- 없음 (정적 콘텐츠)

### 실제 테스트 필요 항목
- [ ] 헤더 내비게이션에서 진입 확인
- [ ] "처음으로" 버튼 동작
- [ ] 모바일에서 비교 박스 레이아웃 (1열로 전환되는지)

---

## 6. 안전사용

### 시작 화면
- `ai-safety-section`
- 헤더 내비게이션 "안전사용" 클릭으로 진입

### 사용자가 누르는 버튼
- "처음으로" 뒤로 버튼
- (검토 도구 패널의 버튼은 `stage14-public-hidden`으로 숨김)

### 입력하는 칸
- 없음 (공개 범위 기준 — 검토 도구 textarea는 비공개)

### 연결된 함수
- `openSection('ai-safety')` — 섹션 전환
- `goHome()` — 홈 복귀
- (`generateReviewPrompt()` — `stage14-public-hidden` 블록 안에 있어 공개 사용자는 접근 불가)

### 생성되는 결과
- 정적 콘텐츠 표시:
  - 환각현상 설명 + 예시 5개
  - AI 사용 신호등 (초록/노랑/빨강)
  - 검토 도구 블록은 비공개 (`stage14-public-hidden`)

### 빈 입력일 때 동작
- 해당 없음

### 뒤로 가기 동작
- "처음으로" → `goHome()`

### 복사 기능
- 없음

### 개인정보 저장 여부
- 없음 ✅

### 오류 가능성
- 없음 (정적 콘텐츠)

### stage14-public-hidden 비공개 요소의 간섭 여부
- "검토 도구" `.safety-block`이 `stage14-public-hidden` 클래스로 숨김 처리됨
- 해당 블록 안의 textarea, 버튼은 DOM에 존재하지만 화면에 표시 안 됨
- 공개 콘텐츠(환각현상, 신호등)에 간섭 없음 ✅

### 실제 테스트 필요 항목
- [ ] 페이지 진입 시 환각현상·신호등 콘텐츠만 표시되는지
- [ ] 검토 도구 블록이 보이지 않는지 확인
- [ ] "처음으로" 버튼 동작


---

## 7. AI 사이트

### 시작 화면
- `ai-sites-section`
- 헤더 내비게이션 "AI 사이트" 클릭으로 진입

### 사용자가 누르는 버튼
- "처음으로" 뒤로 버튼
- 각 사이트의 "바로가기 →" 링크 (외부 링크, `target="_blank"`)

### 입력하는 칸
- 없음

### 연결된 함수
- `openSection('ai-sites')` — 섹션 전환
- `initSitesTable()` — DOMContentLoaded 시 테이블 렌더링
- `goHome()` — 홈 복귀

### 생성되는 결과
- `aiSites` 배열 데이터로 테이블 동적 생성
- 열: 분류, 이름, 무료, 한국어, 학생사용, 링크
- 16개 AI 서비스 목록

### 빈 입력일 때 동작
- 해당 없음

### 뒤로 가기 동작
- "처음으로" → `goHome()`

### 복사 기능
- 없음

### 개인정보 저장 여부
- 없음 ✅

### 오류 가능성
- 외부 링크 접속 불가 시 사용자 측 문제 (사이트 책임 아님)
- `aiSites` 데이터에서 URL이 변경된 경우 → 주기적 확인 필요 (정보 확인일: 2026.07.11)

### 실제 테스트 필요 항목
- [ ] 테이블 정상 렌더링 (16행)
- [ ] 외부 링크 새 탭 열림 확인
- [ ] 모바일에서 가로 스크롤 정상 동작
- [ ] "처음으로" 버튼 동작

---

## 8. 공통 결과 화면

### 시작 화면
- `result-section`
- 다른 기능(1~4)에서 프롬프트 생성 후 자동 전환

### 사용자가 누르는 버튼
- "복사하기" (`onclick="requestPromptCopy()"`)
- "이전 화면으로" (`onclick="goBackFromResult()"`)
- "공유하기" / "텍스트 복사" (`onclick="sharePrompt('share'/'copy')"`)
- AI 사이트 바로가기 (ChatGPT/Gemini/Claude/Copilot)
- 즐겨찾기 ⭐ 버튼 (`stage14-public-hidden` — 공개 사용자에게 숨김)
- 복사 확인 모달: "수정하러 가기" / "확인 후 그대로 복사"

### 입력하는 칸
- `#result-prompt` (textarea, 생성된 프롬프트 편집 가능)

### 연결된 함수
- `showSection('result')` — 결과 화면 표시
- `requestPromptCopy()` — 자리표시자 확인 후 복사
- `confirmPromptCopy()` — 모달에서 확인 후 복사
- `focusFirstUnresolvedPlaceholder()` — 미완성 자리표시자로 이동
- `copyPrompt()` — 실제 클립보드 복사
- `onPromptEdit(value)` — 편집 시 상태 갱신
- `updatePlaceholderStatus()` — 자리표시자 상태 표시
- `goBackFromResult()` — 이전 섹션 복귀
- `sharePrompt(method)` — 공유/텍스트 복사
- `capturePromptStorageSnapshot()` — 저장용 스냅샷 생성

### 생성되는 결과
- 의도 해석 (`#result-intent`)
- 프롬프트 textarea (`#result-prompt`)
- 자리표시자 상태 바 (`#placeholder-status`)
- 팁 (`#result-tip`)
- AI 바로가기 버튼 4개

### 빈 입력일 때 동작
- 결과 화면 자체는 프롬프트가 있어야 표시됨 (빈 상태로 도달 불가)

### 뒤로 가기 동작
- `goBackFromResult()` → `resultReturnTarget` 기반으로 이전 섹션 복귀
- 생기부에서 온 경우 → record-section
- 홈에서 온 경우 → home-section
- 거꾸로에서 온 경우 → reverse-section (코드상은 home으로 감)

### 복사 기능
- 자리표시자 `{{}}` 잔존 여부 검사
- 있으면 확인 모달 표시 → 선택에 따라 복사 또는 해당 위치로 이동
- 없으면 즉시 복사 + 토스트 "프롬프트가 복사되었습니다!"
- 첨부파일 프롬프트인 경우 → "AI 대화창에 파일도 함께 첨부하세요" 안내

### 개인정보 저장 여부
- `capturePromptStorageSnapshot()` → `createSafeFavoriteEntry()` — 안전 템플릿만 생성
- 즐겨찾기 버튼은 `stage14-public-hidden`이라 공개 사용자는 저장 불가
- 최근 기록도 `stage14-public-hidden`이라 공개 사용자에게 표시 안 됨
- (다만 `saveToRecent()` 함수는 여전히 호출됨 — 메타데이터만 저장) ✅

### 오류 가능성
- `navigator.clipboard` 미지원 브라우저 → `execCommand('copy')` 폴백 있음
- 공유 API 미지원 → 클립보드 복사로 대체 + 토스트 안내
- textarea 편집 후 자리표시자 상태가 실시간 갱신되는지 확인 필요
- Escape 키로 모달 닫기 동작

### stage14-public-hidden 비공개 요소의 간섭 여부
- 즐겨찾기 ⭐ 버튼: `stage14-public-hidden` — 화면에 안 보임 ✅
- 빠르게 바꾸기/진단 패널 (`.refine-section`): `stage14-public-hidden` — 안 보임 ✅
- 과목 바꾸기 (`.subject-change`): `stage14-public-hidden` — 안 보임 ✅
- 거꾸로 생성 토글 (`.reverse-section` 내 결과 화면 버전): `stage14-public-hidden` — 안 보임 ✅
- `GENERAL` 모드가 아닌 경우(`RECORD_DRAFT` 등) → 이 요소들의 display를 JS로 none 처리 → 이중 숨김이라 간섭 없음 ✅

### 실제 테스트 필요 항목
- [ ] 프롬프트 생성 후 결과 화면 정상 표시
- [ ] 자리표시자 있을 때 복사 → 모달 표시
- [ ] "수정하러 가기" → 해당 위치로 커서 이동
- [ ] "확인 후 그대로 복사" → 클립보드 복사 성공
- [ ] 자리표시자 없을 때 즉시 복사
- [ ] 프롬프트 편집 → 상태 바 실시간 갱신
- [ ] "이전 화면으로" 텍스트가 출발 섹션에 따라 변하는지
- [ ] 공유하기 버튼 (Web Share API 지원/미지원 케이스)
- [ ] AI 바로가기 링크 새 탭 열림
- [ ] Escape 키 모달 닫기
- [ ] 즐겨찾기/다듬기 버튼이 화면에 보이지 않는지


---

## 비공개 기능이 공개 기능을 방해하는지 확인

| 비공개 요소 | 위치 | 공개 기능 간섭 여부 |
|---|---|---|
| 오늘의 추천 위젯 | home-section 상단 | ❌ 없음 (display:none) |
| 빠른 선택 버튼 | home-section | ❌ 없음 |
| 제작소 카드 (학교생활/수업/평가) | home-section | ❌ 없음 |
| 레시피북 내비게이션 | header-nav | ❌ 없음 |
| 즐겨찾기 내비게이션 | header-nav | ❌ 없음 |
| 최근 기록 섹션 | home-section 하단 | ❌ 없음 (`saveToRecent()`는 호출되나 UI 숨김) |
| 결과 화면 다듬기/진단 | result-section | ❌ 없음 (GENERAL 모드에서도 stage14로 숨김) |
| 결과 화면 과목 변경 | result-section | ❌ 없음 |
| 결과 화면 거꾸로 토글 | result-section | ❌ 없음 |
| 결과 화면 즐겨찾기 ⭐ | result-section | ❌ 없음 |
| 검토 도구 (CPR/최종점검/편향) | record-review-panel | ❌ 없음 (카드 UI 숨김) |
| 안전사용 검토 도구 | ai-safety-section | ❌ 없음 |
| 다크모드 버튼 | header | ❌ 없음 (hidden 속성 + DARK_MODE_PUBLIC_ENABLED=false) |

**결론:** 비공개(`stage14-public-hidden`) 요소가 공개 기능의 동작을 방해하는 경우는 발견되지 않았습니다.

---

## 종합 위험 요약

| 우선순위 | 항목 | 설명 |
|---|---|---|
| 높음 | 입력 품질 검사 | "이거 써도 되나요?"에서 자모/기호만 입력 시 차단이 정상 동작하는지 |
| 높음 | 파일 파싱 | XLSX/PDF 라이브러리 CDN 로드 실패 시 폴백 안내 |
| 중간 | 클립보드 복사 | HTTP 환경(localhost 아닌)에서 clipboard API 동작 여부 |
| 중간 | 복사 모달 | 자리표시자 감지 + 모달 표시 + Escape 닫기 전체 흐름 |
| 중간 | 항목 변경 모달 | 검토 작업 중 항목 변경 → 확인/취소 시 상태 복원 |
| 낮음 | 생기부 키워드 오감지 | 일반 요청에 "세특" 등 포함 시 생기부로 분기 |
| 낮음 | placeholder 순환 | focus/blur 시 타이머 중단/재개 |
| 낮음 | saveToRecent 호출 | 비공개 UI라도 localStorage에 메타데이터는 쌓임 (5개 제한) |
