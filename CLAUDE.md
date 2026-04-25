# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@AGENTS.md

## 명령어

```bash
npm run dev      # 개발 서버 시작
npm run build    # 프로덕션 빌드
npm run lint     # ESLint 실행
npx tsc --noEmit # 타입 검사 (테스트 대용)
```

## 아키텍처

**Next.js 16 App Router** 기반 프로젝트. `@/*`는 프로젝트 루트를 가리키는 path alias입니다 (예: `@/lib/utils`).

### 기술 스택

- **UI**: TailwindCSS v4 (설정 파일 없음, CSS-first), shadcn/ui, Radix UI
- **Notion**: `@notionhq/client` — 유일한 데이터베이스 (별도 DB 없음)
- **PDF**: `@react-pdf/renderer` — 견적서 PDF 생성
- **로깅**: `pino` — console.log 대신 사용

### 데이터 흐름

관리자가 Notion DB에 견적서 입력 → Notion Page ID 기반 공유 링크 생성 → 클라이언트가 링크로 접속하여 웹뷰 확인 및 PDF 다운로드.

관리자 UI 없음 — Notion 자체가 입력 인터페이스.

### Next.js 16 핵심 주의사항

**동적 라우트 params는 반드시 비동기 접근:**

```typescript
// ✅ 올바른 방식
export default async function Page(props: { params: Promise<{ id: string }> }) {
  const { id } = await props.params;
}
```

**`@react-pdf/renderer`는 Server Component에서 사용 불가.** `PDFDownloadLink` 등은 반드시 `"use client"` Client Component 안에서만 사용합니다.

### 환경변수 (`.env.local`)

```
NOTION_TOKEN=secret_xxxx
NOTION_QUOTE_DB_ID=xxxx
```

### lineItems 저장 구조

Notion Page 내부 **Table 블록(하위 블록)**으로 저장. `blocks.children.list({ block_id: pageId })`로 `table_row` 블록을 조회하며, 숫자 셀은 텍스트로 저장되므로 `Number()` 변환이 필요합니다.

### 유효기간 타임존

Vercel 서버는 UTC 기준이므로, 유효기간 만료 판단 시 KST(UTC+9) 보정이 필요합니다:

```typescript
const KST_OFFSET_MS = 9 * 60 * 60 * 1000;
const nowKST = new Date(Date.now() + KST_OFFSET_MS);
```
