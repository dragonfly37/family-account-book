# 가족 가계부 프로젝트

> 마지막 업데이트: 2026-05-12

---

## 목적

가족 공용 지출을 실시간으로 기록·분석하는 모바일 최적화 웹 가계부.
- 쿠팡, 배달, 외식 등 카테고리별 지출 추적
- 누가 썼는지(분류)별 정산 관리 (초딩/밴드/공용카드 등)
- 아이(나윤) 관련 지출 별도 추적
- Google Sheets를 DB로, GAS를 백엔드 API로 사용 (서버 비용 0원)

---

## 접근 URL

| 항목 | URL |
|---|---|
| 웹 가계부 (GitHub Pages) | https://dragonfly37.github.io/family-account-book/ |
| GitHub 레포 | https://github.com/dragonfly37/family-account-book |
| Google Spreadsheet | https://docs.google.com/spreadsheets/d/1hqF4oJD6kAZ8hX5czrVZ_hiMIhxQC5d4nFjhKyrUAZA/edit |
| Google Apps Script | https://script.google.com/u/0/home/projects/15kCzY8WxbVaaKIJH5B-NULgWYQ6kqDV66kcPma_Wvl2lf2WZnErSL3Ct/edit |
| GAS 웹앱 API | https://script.google.com/macros/s/AKfycbzULYB6GX8XAdkow6xCEQoVXmwitKGShSC7Ly4sB38ViTQBNTJHyxmV_hBYdaBa4ixx/exec |

---

## 로컬 파일 구조

```
/Users/jeongyong/workspace/home_calculator/
└── family-account-book/          ← git repo (dragonfly37/family-account-book)
    ├── index.html                ← 전체 앱 (단일 파일, ~1700줄)
    └── PROJECT.md                ← 이 문서
```

### index.html 구조

```
index.html
├── <head>
│   ├── Tailwind CSS (CDN)
│   ├── Chart.js (CDN)
│   ├── Noto Sans KR (Google Fonts)
│   └── <style> 커스텀 CSS
│
├── <body> .app-wrap (max-width: 430px, 모바일 최적화)
│   ├── TAB: 홈 (tab-home)
│   │   ├── 헤더 (월 선택, 날짜 표시)
│   │   ├── 대시보드 카드 (총지출, 미정산, 주간 차트)
│   │   ├── 필터 칩 (분류·카테고리·구매처·정산)
│   │   └── 거래 목록 (수정/삭제 버튼)
│   │
│   ├── TAB: 기록 추가 (tab-add)
│   │   └── 폼 (날짜, 금액, 분류, 카테고리, 구매처, 항목, 정산상태)
│   │
│   ├── TAB: 전체 목록 (tab-list)
│   │   └── 날짜 역순 전체 리스트
│   │
│   ├── TAB: 분석 (tab-analysis)
│   │   ├── 월별 지출 요약
│   │   ├── 예산 vs 실제 Gap
│   │   ├── 카테고리별 차트
│   │   └── [스프레드시트 분석 업데이트] 버튼
│   │
│   ├── <nav> 하단 네비게이션 (홈/기록/목록/분석)
│   └── <div id="toast"> 저장완료 팝업
│
└── <script>
    ├── const API_URL = "https://script.google.com/macros/s/.../exec"
    ├── loadData()           ← GAS에서 전체 데이터 fetch
    ├── applyFilters()       ← 필터 적용 후 렌더
    ├── renderDashboard()    ← 홈 대시보드
    ├── renderRecords()      ← 거래 목록
    ├── renderAnalysis()     ← 분석 탭
    ├── startEdit(id)        ← 수정 모드 진입
    ├── handleWebAction()    ← 저장/수정/삭제 (GET+payload 방식)
    ├── showToast(msg)       ← 저장완료 토스트
    └── triggerAnalysisUpdate() ← 스프레드시트 분석 갱신
```

---

## Google Spreadsheet 구조

**스프레드시트 ID:** `1hqF4oJD6kAZ8hX5czrVZ_hiMIhxQC5d4nFjhKyrUAZA`

### 시트 목록

| 시트명 | 용도 |
|---|---|
| **Data** | 거래 원장 (메인 데이터) |
| **Analysis** | 자동 생성 분석 결과 (GAS가 작성) |
| **Entity** | 드롭다운 기준값 목록 |
| **reference** | 데이터 정의·입력 가이드 |
| **web-link** | 웹앱 URL 메모 |

### Data 시트 컬럼 구조

| 열 | 컬럼명 | 형식 | 설명 |
|---|---|---|---|
| A | ID | 숫자(timestamp) | 고유 ID (밀리초 timestamp, GAS 자동생성) |
| B | Date | 텍스트 `YYYY-MM-DD` | 지출 날짜 |
| C | Month | 텍스트 `YYYY-MM` | 월 (필터용, Date에서 자동생성) |
| D | Class | 드롭다운 | 지출 주체 (누가 씀) |
| E | Category | 드롭다운 | 카테고리 |
| F | SubCategory | 드롭다운 | 구매처 |
| G | Item | 텍스트 | 품목명 |
| H | Amount | 숫자 | 금액 (원) |
| I | Settlement | 드롭다운 | 정산 상태 |
| J | Closed | 텍스트 `Yes/No` | 마감 여부 |
| K | memo | 텍스트 | 메모 (선택) |

### Entity 시트 드롭다운 값

**Class (지출 주체):**
- 초딩 사용, 밴드 사용, 공용카드(쿠팡), 공용계좌(토스), 온누리, 제외(개인/저축/이자), 기타

**Category (14개):**
- 1. 식자재 / 2. 생필품 / 3. 외식/커피 / 4. 쇼핑 / 5. 기타
- 6. 주유/차량 / 7. 문화 / 8. 개인 사용 / 9. 경조사/세금/공과금
- 10. 저축/투자 / 11. 여행/가족 / 12. 병원/약국 / 13. 1회성(가구,비품) / 14. 나윤

**Settlement (정산 상태):**
- 정산 불필요, 지급 완료, 정산 필요, 개인사용분 - 채워넣기 완료

**SubCategory (구매처):**
- 쿠팡, 배달, 식당/카페, 온라인, 편의점, 당근, 병원/약국, 마트, 기타

---

## Google Apps Script 구조

**스크립트 ID:** `15kCzY8WxbVaaKIJH5B-NULgWYQ6kqDV66kcPma_Wvl2lf2WZnErSL3Ct`

### 파일 목록

```
Apps Script 프로젝트
├── appsscript.json       ← 설정 (timezone: Asia/Seoul, webapp: ANYONE_ANONYMOUS)
├── Code.gs               ← 메인 API (doGet, doPost, handleWebAction, onEdit)
├── analysis.gs           ← Analysis 시트 자동 업데이트 (updateAnalysis)
├── date fix.gs           ← 날짜 KST 교정 유틸 (fixExistingDates)
└── apply_validation.gs   ← 드롭다운 유효성 복사 (applyValidationToLastRow)
```

### Code.gs 주요 함수

| 함수 | 용도 |
|---|---|
| `doGet(e)` | GET 요청 처리. payload 파라미터 있으면 handleWebAction 위임, 없으면 전체 데이터 반환 |
| `handleWebAction(payload)` | CORS 우회용 GET 기반 액션 처리 (create/update/delete/updateAnalysis) |
| `doPost(e)` | POST 요청 처리 (현재 CORS 문제로 미사용, GET 방식 사용 중) |
| `onEdit(e)` | 시트 편집 시 트리거: Analysis 체크박스, Data ID 자동생성, Month 자동채움 |
| `getMonthFromDate(dateVal)` | Date → `YYYY-MM` 텍스트 변환 |
| `getDateStr(dateVal)` | Date → `YYYY-MM-DD` 텍스트 변환 |
| `addMonthColumn()` | [1회성] Month 컬럼 삽입 + 기존 데이터 채우기 |
| `fixDateColumns()` | [1회성] Date/Month 컬럼 표시형식 통일 |
| `fillMissingIds()` | [1회성] ID 없는 행에 ID 채우기 |

### analysis.gs 주요 함수

| 함수 | 용도 |
|---|---|
| `updateAnalysis()` | Data 시트를 읽어 Analysis 시트 전체 재작성 |
| `onOpen()` | 스프레드시트 열릴 때 커스텀 메뉴 추가 |
| `setupHourlyTrigger()` | 매시간 자동 업데이트 트리거 설정 |

**Analysis 섹션 구성:**
1. ① 월별 총 지출 (전체 / 제외 제거)
2. ② 예산 vs 실제 Gap (목표 카테고리: 1~7번)
3. ③ 카테고리별 월별 지출
4. ④ 분류(누구)별 월별 지출
5. ⑤ 구매처별 월별 지출
6. ⑥ 정산 상태별 요약
7. ⑦ 월별 TOP 10 지출 항목 (전체/목표포함/목표제외)
8. ⑧ 카테고리 비율 (전체 기간)

**COL 인덱스 (analysis.gs 내부):**
```javascript
const COL = {
  id: 0, date: 1, month: 2, cls: 3, cat: 4, sub: 5,
  item: 6, amount: 7, settlement: 8, closed: 9
};
```

### GAS API 통신 방식

```
웹페이지(GitHub Pages) → GAS 웹앱 API (GET + ?payload=JSON)
                          ↓
                    doGet() → handleWebAction()
                          ↓
                    Google Spreadsheet (Data 시트)
```

**CORS 우회 이유:** `script.google.com` → `script.googleusercontent.com` 리다이렉트 과정에서 POST CORS 차단 발생. GET 방식으로 우회하여 해결.

---

## 예산 설정

| 항목 | 내용 |
|---|---|
| 예산 대상 카테고리 | 1~7번 (식자재, 생필품, 외식/커피, 쇼핑, 기타, 주유/차량, 문화) |
| 1월 목표 | ₩1,200,000 |
| 2월~ 목표 | ₩750,000/월 |
| 제외 항목 | `제외(개인/저축/이자)` 클래스 (저축, 이자, 개인비용) |

---

## 진행 이력 (2026-05-12)

### 배경
- 원본 프로젝트가 회사 Google 계정에 연동되어 있었음
- 회사 계정 GAS 웹앱 URL이 개인 계정에서 접근 불가 → 수정 버튼 미반영 문제
- 개인 계정으로 GAS 사본 생성 후 새로 배포

### 완료 작업

#### 1. GAS 버그 수정 (Code.gs)
- `doPost`의 `payload.action` → `body.action` 변수명 오류 수정
- `handleWebAction` update/create: `payload.Closed === 'Yes'` (boolean) → `payload.Closed` (문자열) 수정

#### 2. GitHub 연동 업데이트
- `index.html` `API_URL` 구 URL → 새 개인 계정 GAS 웹앱 URL로 교체
- `dragonfly37/family-account-book` 레포에 푸시

#### 3. UI 개선
- 수정/저장 완료 토스트 팝업 추가 (`showToast()`, 2초 자동 사라짐)
- 저장 후 홈 화면 로딩 스피너 제거 (`loadData(silent=true)`)

#### 4. Month 컬럼 추가
- Data 시트 C열에 `Month` 컬럼 추가 (`YYYY-MM` 텍스트 형식)
- 기존 데이터 소급 적용용 `addMonthColumn()` 함수 작성
- 날짜 형식 통일용 `fixDateColumns()` 함수 작성
- GAS 컬럼 인덱스 전체 업데이트 (D열~로 기존 컬럼 이동)
- `onEdit` 중복 버그 수정: Code.gs와 analysis.gs 두 곳에 `onEdit` 존재 → Code.gs 하나로 통합

### 남은 작업

- [ ] `fixDateColumns()` GAS 에디터에서 실행 → Date/Month 컬럼 형식 통일
- [ ] Analysis 시트 체크박스 클릭 → `updateAnalysis()` 정상 동작 확인
- [ ] 신규 데이터 입력 시 Month 자동 채움 확인

---

## 개발 방법

### GitHub 코드 수정 후 배포

```bash
cd /Users/jeongyong/workspace/home_calculator/family-account-book
# index.html 수정 후
git add index.html
git commit -m "feat: 설명"
git push https://dragonfly37:TOKEN@github.com/dragonfly37/family-account-book.git main
```

GitHub Pages는 push 후 1~2분 내 자동 배포.

### GAS 코드 수정

Claude Code MCP(`mcp__google-workspace__gas_push`)로 직접 배포 가능:
- 스크립트 ID: `15kCzY8WxbVaaKIJH5B-NULgWYQ6kqDV66kcPma_Wvl2lf2WZnErSL3Ct`
- 배포 후 **웹앱 재배포 필요** (GAS 편집기 → 배포 → 새 배포 또는 기존 배포 업데이트)

> **주의:** GAS 코드 변경은 gas_push로 저장되지만, 웹앱 URL이 동일하게 유지되려면 "기존 배포 업데이트"를 선택해야 함.

### GAS 일회성 함수 실행

`gas_run` MCP가 이 프로젝트에서 막혀 있어 GAS 편집기에서 직접 실행 필요:
1. GAS 편집기 열기
2. 상단 드롭다운에서 함수명 선택
3. ▶ 실행

### 스프레드시트 분석 업데이트

방법 1: 웹 가계부 → 분석 탭 → [스프레드시트 분석 업데이트] 버튼  
방법 2: 스프레드시트 → Analysis 시트 → A1 체크박스 클릭  
방법 3: 스프레드시트 메뉴 → 📊 가계부 → 분석 업데이트

---

## 계정 정보

| 항목 | 값 |
|---|---|
| GitHub 계정 (가계부 레포) | `dragonfly37` |
| GitHub 계정 (메인) | `JYdragon37` |
| Google 계정 (GAS/Sheets) | 개인 계정 (dragonfly37 연동) |
