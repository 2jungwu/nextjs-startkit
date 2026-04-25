# invoice-web MVP PRD

---

## 1. 프로젝트 핵심

**목적**: Notion 데이터베이스를 유일한 저장소로 활용하여, 관리자가 입력한 견적서를 고객이 공유 링크로 웹에서 확인하고 PDF로 다운로드할 수 있는 견적서 웹뷰 서비스를 제공한다.

**타겟 사용자**: 견적서를 Notion으로 관리하는 프리랜서·소규모 사업자(관리자)와 그 견적서를 수신하는 고객(클라이언트).

---

## 2. 사용자 여정

```
[관리자]                          [시스템]                        [클라이언트]
    │                                │                                  │
    ├─ Notion DB에 견적서 입력 ──────►│                                  │
    │   (고객명/항목/단가/유효기간)   │                                  │
    │                                ├─ 고유 공유 링크 자동 생성         │
    │◄─ 공유 링크 확인 ──────────────┤  (Notion Page ID 기반)            │
    │                                │                                  │
    ├─ 링크를 고객에게 전달 ─────────────────────────────────────────────►│
    │                                │                                  │
    │                                │          공유 링크 접속 ──────────┤
    │                                │◄─────────────────────────────────┤
    │                                ├─ Notion API로 견적서 데이터 조회  │
    │                                ├─────────────────────────────────►│
    │                                │                견적서 웹뷰 확인  │
    │                                │                                  ├─ PDF 다운로드 버튼 클릭
    │                                │◄─────────────────────────────────┤
    │                                ├─ PDF 생성 및 반환 ───────────────►│
    │                                │                PDF 파일 저장 완료 │
```

---

## 3. 기능 명세 — MVP 핵심 기능

| 기능 ID | 기능명 | 설명 | 관련 페이지 |
|---------|--------|------|-------------|
| F001 | Notion 견적서 데이터 조회 | Notion Integration을 통해 Page ID에 해당하는 견적서 Database 프로퍼티와 하위 Table 블록(lineItems)을 조회한다. 유효하지 않은 ID인 경우 에러 상태를 반환한다. | 견적서 웹뷰 페이지 |
| F002 | 견적서 웹뷰 렌더링 | 조회된 견적서 데이터(고객명·발급자 회사명·발급자 연락처·견적 항목 테이블·합계·유효기간·비고)를 브랜드 스타일의 웹 화면으로 렌더링한다. | 견적서 웹뷰 페이지 |
| F003 | 견적 항목 합계 계산 표시 | 각 항목(품목·수량·단가)의 소계 및 전체 합계를 자동 계산하여 표시한다. `taxRate`가 0보다 크면 세전·세후 금액을 모두 표시하고, 0이면 합계만 표시한다. | 견적서 웹뷰 페이지 |
| F004 | PDF 다운로드 | 현재 웹뷰와 동일한 레이아웃의 PDF를 생성하여 클라이언트가 파일로 다운로드할 수 있게 한다. 파일명은 `견적서_{고객명}_{날짜}.pdf` 형식을 따른다. | 견적서 웹뷰 페이지 |
| F005 | 유효기간 만료 안내 | 견적서의 유효기간이 지난 경우 만료 안내 배너를 웹뷰 상단에 표시한다. PDF 다운로드는 유지된다. | 견적서 웹뷰 페이지 |
| F006 | 견적서 없음 안내 | 잘못된 Page ID로 접근하거나 Notion API 오류 발생 시 사용자에게 친절한 오류 안내 화면을 표시한다. | 오류 안내 페이지 |

---

## 4. 메뉴 구조

이 서비스는 관리자 UI가 없으며, 클라이언트가 공유 링크를 통해 직접 접근하는 단일 흐름 구조이다.

```
invoice-web
└── 견적서 웹뷰 페이지          [F001, F002, F003, F004, F005]
    └── (접근 실패 시)
        오류 안내 페이지         [F006]
```

---

## 5. 페이지별 상세 기능

### 5-1. 견적서 웹뷰 페이지

| 항목 | 내용 |
|------|------|
| **역할** | 공유 링크로 접근한 클라이언트가 견적서를 확인하고 PDF를 다운로드하는 핵심 페이지 |
| **진입 조건** | 유효한 Notion Page ID가 포함된 공유 링크 접속 (인증 불필요) |

**사용자 행동**
- 링크로 접속하여 견적서 내용을 확인한다.
- 유효기간 만료 여부를 배너로 인지한다.
- PDF 다운로드 버튼을 눌러 견적서를 파일로 저장한다.

**기능 목록**

| # | 기능 | 구현 기능 ID |
|---|------|--------------|
| 1 | Notion API를 통해 Page ID로 견적서 데이터 조회 | F001 |
| 2 | 고객명, 발급자(회사명·연락처), 견적일, 유효기간, 비고 영역 렌더링 | F002 |
| 3 | 견적 항목 테이블(품목·수량·단가·소계) 및 합계 영역 렌더링 | F002, F003 |
| 4 | 각 항목 소계 및 전체 합계(세전·세후) 자동 계산 표시 | F003 |
| 5 | 유효기간 만료 시 상단 만료 안내 배너 표시 | F005 |
| 6 | PDF 다운로드 버튼 클릭 시 견적서 PDF 생성 및 파일 다운로드 | F004 |

---

### 5-2. 오류 안내 페이지

| 항목 | 내용 |
|------|------|
| **역할** | 잘못된 링크 또는 API 오류 시 클라이언트에게 상황을 안내하는 페이지 |
| **진입 조건** | 존재하지 않는 Page ID 접근, Notion API 오류, 네트워크 오류 발생 시 |

**사용자 행동**
- 견적서를 찾을 수 없다는 안내 메시지를 확인한다.
- 관리자에게 링크 재확인을 요청할 수 있도록 안내 문구를 읽는다.

**기능 목록**

| # | 기능 | 구현 기능 ID |
|---|------|--------------|
| 1 | 오류 유형별(잘못된 ID / API 오류) 안내 메시지 표시 | F006 |
| 2 | 관리자 문의 유도 안내 문구 표시 | F006 |

---

## 6. 데이터 모델

Notion 데이터베이스의 각 Row(Page)가 견적서 1건을 나타낸다.

**lineItems 저장 방식**: Page 내부 **Notion Table 블록(하위 블록)** 방식을 사용한다. 관리자가 Notion 페이지 안에 Table을 직접 작성하며, API에서는 `blocks.children.list({ block_id: pageId })`로 `table_row` 블록을 조회하여 파싱한다. 숫자 필드(`quantity`, `unitPrice`)는 Notion에서 텍스트로 저장되므로 `Number()` 변환이 필요하다.

### QuoteRecord (Notion Database Row)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `clientName` | `string` | 고객(수신자) 이름 또는 회사명 |
| `clientEmail` | `string` | 고객 이메일 (표시용) |
| `issuerName` | `string` | 발급자 회사명 또는 이름 |
| `issuerContact` | `string` | 발급자 연락처 (전화번호·이메일 등) |
| `validUntil` | `date` | 견적서 유효기간 (KST 기준으로 비교) |
| `taxRate` | `number` | 세율 (%, 예: 10 → 10%). 0이면 세금 없음 |
| `note` | `string` | 비고 (선택 항목) |

### LineItem (견적 항목 — Page 내 Table 블록)

| 필드명 | 타입 | 설명 |
|--------|------|------|
| `description` | `string` | 품목명 |
| `quantity` | `number` | 수량 (Notion 텍스트 셀 → Number 변환) |
| `unitPrice` | `number` | 단가 (원화 기준, Notion 텍스트 셀 → Number 변환) |
| `subtotal` | `number` | 소계 (quantity × unitPrice, 클라이언트에서 계산)

---

## 7. 기술 스택

| 영역 | 기술 | 버전 |
|------|------|------|
| 프레임워크 | Next.js (App Router) | 16.x |
| 언어 | TypeScript | 5.x |
| 스타일링 | TailwindCSS | v4.x |
| UI 컴포넌트 | shadcn/ui | latest |
| Notion 연동 | @notionhq/client | 2.x |
| PDF 생성 | @react-pdf/renderer | 4.x |
| 로깅 | pino | 9.x |
| 배포 | Vercel | — |

---

## 8. 구현 주의사항

### Next.js 16 Breaking Change — params 비동기 접근 필수
동적 라우트(`/invoice/[pageId]`)의 `params`는 반드시 비동기로 접근해야 한다. 동기 접근 시 런타임 오류 발생.

```typescript
// ✅ 올바른 방식 (Next.js 16)
export default async function Page(props: { params: Promise<{ pageId: string }> }) {
  const { pageId } = await props.params;
}

// ❌ 잘못된 방식 (Next.js 15 이하 패턴)
export default async function Page({ params }: { params: { pageId: string } }) {
  const { pageId } = params;
}
```

### @react-pdf/renderer — Client Component 분리 필수
`PDFDownloadLink`는 브라우저 DOM이 필요한 Web-only 컴포넌트이므로 Server Component에서 사용 불가. PDF 다운로드 버튼만 `"use client"` Client Component로 분리한다.

```
견적서 웹뷰 페이지 (Server Component)
  └── 견적서 데이터 조회 (Notion API, 서버에서 실행)
  └── 견적서 렌더링 (서버에서 HTML 생성)
  └── <PdfDownloadButton /> (Client Component, "use client")
        └── PDFDownloadLink (브라우저에서 실행)
```

### 환경변수 명세
```
NOTION_TOKEN=secret_xxxx          # Notion Internal Integration Secret
NOTION_QUOTE_DB_ID=xxxx           # 견적서 Notion 데이터베이스 ID
```

### 유효기간 타임존 처리
Vercel 서버는 UTC 기준으로 동작하므로, `validUntil` 만료 판단은 한국 시간(KST, UTC+9)으로 보정하여 비교한다.

```typescript
const KST_OFFSET_MS = 9 * 60 * 60 * 1000;
const nowKST = new Date(Date.now() + KST_OFFSET_MS);
const isExpired = new Date(validUntil) < nowKST;
```
