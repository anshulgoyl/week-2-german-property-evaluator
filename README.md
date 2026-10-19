# German Property Evaluator — n8n workflows

An n8n-based property research project that accepts a German property listing PDF, generates an investment report, and supports follow-up questions tied to the evaluated property. It uses Nebius models, Tavily search, Pinecone, and n8n data tables.

The report is an initial research aid. Listing information, market estimates, costs, and assumptions require independent verification; the output is not a certified valuation.

## What's included

| File | Purpose | Status in the original instance |
| --- | --- | --- |
| `workflows/01-property-evaluator-api.json` | PDF upload, investment analysis, market research, report generation, and persistence | Main evaluator; published |
| `workflows/02-rag-chat.json` | Property-linked follow-up chat and conversation history | Main chat; published |
| `workflows/03-knowledge-ingestion.json` | Upload, chunk, embed, and index reference PDFs or spreadsheets | Supporting workflow; inactive |
| `workflows/04-unified-ui.json` | Serves the upload, report, and chat interface through n8n | Main UI; published |
| `workflows/05-evaluator-legacy.json` | Earlier form-based evaluator | Historical; inactive |
| `workflows/06-report-viewer-legacy.json` | Earlier saved-report viewer | Historical; inactive |
| `workflows/07-connection-test.json` | Early connection and embedding test | Diagnostic; inactive |

The shareable exports are deliberately **inactive** and contain no saved execution data. Instance-specific credential references, data-table IDs, and the Pinecone host have been removed or replaced. See [setup instructions](docs/setup.md) before activating anything. Legacy and diagnostic workflows are included for project history and may require repair or reconfiguration.

## How the main flow works

1. Open the UI at `http://localhost:5678/webhook/property-ui` on the machine running n8n.
2. Upload a PDF. The UI sends it to `POST /webhook/evaluate-property` as multipart field `property_pdf`.
3. The evaluator extracts the listing, retrieves background context, generates an investment report, searches for comparable listings and an area benchmark, and saves the result to an n8n data table.
4. The UI displays the report and sends follow-up questions to `POST /webhook/property-chat` with `property_id`, `sessionId`, and `chatInput`.

When the workflow finds fewer than three individual comparable candidates, it may show a sourced area-price benchmark. A broader benchmark is labeled as such, with no unsupported adjustment for property condition or subtype. If the available source evidence is insufficient, the report does not invent a benchmark. Benchmarks are not used to change the saved investment score.

The original chat workflow contains embedding and Pinecone retrieval nodes that are **not connected to the active chat path**. Its current responses use the saved property record and conversation history. The ingestion workflow and evaluator have their own Pinecone steps; review the connections before describing chat as live vector retrieval.

## Security and scope

- Do not commit API keys, credential exports, n8n databases, real property PDFs, chat history, or sensitive reports without a deliberate decision to share them.
- Exports may still contain implementation details or public listing references; review them before making a repository public.
- The original webhooks do not require authentication. The UI is configured for `localhost:5678`. Restrict n8n's network exposure or configure appropriate access controls before use beyond a trusted local environment.
- n8n's sandboxed HTML responses have an opaque origin. The UI uses absolute URLs, does not depend on local storage, and keeps IDs in the page; the API webhooks must allow the browser's cross-origin requests. See [n8n's webhook documentation](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/).
- Returned report HTML is rendered in a restricted sandboxed frame. External report assets may not display.

## Data and backups

This repository contains workflow definitions and configuration instructions, **not** a complete n8n backup. It does not include credentials, data-table rows, Pinecone vectors, user PDFs, or execution history. See [setup instructions](docs/setup.md) for the table schemas and external dependencies.
