# Setup and import

## Requirements

- A self-hosted n8n instance. The source instance was running n8n 2.43.5; compatibility with other versions has not been tested.
- Nebius Token Factory access for `Qwen/Qwen3-Embedding-8B` and `Qwen/Qwen3-235B-A22B-Instruct-2507`.
- Tavily search access.
- A Pinecone index compatible with the selected embedding model and the `documents` namespace. Confirm the index dimension and metric match the embedding model before indexing.
- Two n8n data tables, described below.

External model and search requests can incur usage charges. Their latency and output vary.

## Import and configure

1. Import the JSON files from `workflows/` through the n8n workflow editor. For the primary application, begin with `01`, `02`, and `04`; import `03` to populate a knowledge index. The remaining files are historical or diagnostic.
2. Create the required credentials in n8n and reselect the correct credential in each HTTP Request node. Credential references were removed from these copies. Use the service's current documentation for its authentication header format; do not put secret values in workflow JSON.
3. Replace `https://YOUR_PINECONE_INDEX_HOST` in the Pinecone HTTP Request nodes with your own index host, retaining each node's path. Configure the namespace and index to match the ingested data.
4. Create the data tables below, then reselect the appropriate table in every Data Table node. `RESELECT_DATA_TABLE` is a placeholder; table names and IDs from the original instance are not portable.
5. Review the webhook paths and authentication/network restrictions. In the UI workflow's **Build UI** node, change `BASE` from `http://localhost:5678` if the browser uses a different n8n origin. The sandboxed UI needs absolute endpoint URLs and cannot use browser local storage. Set and test CORS for the sandboxed origin on the API webhooks; avoid broad access on an exposed instance.
6. Publish the evaluator API, RAG chat, and UI workflows. Open the production UI URL and test a PDF upload, report, and follow-up question. Importing and publishing a workflow does not restore previous data.

## Data tables

Create these columns as **String** in n8n. n8n supplies its own row identifiers and timestamps.

### Property evaluations

| Column | Purpose |
| --- | --- |
| `property_id` | ID used to link evaluation and chat |
| `source_filename` | Original PDF filename |
| `property_text` | Extracted property listing text |
| `report_json` | Structured investment report serialized as JSON |
| `final_report_html` | Final report, including market comparison |

The evaluator's insert and update nodes, chat's property lookup, and legacy report viewer must point to the **same** table. Source exports used inconsistent cached table names for this same table; select the actual table explicitly in each node.

### Chat history

| Column | Purpose |
| --- | --- |
| `session_id` | Conversation session identifier |
| `property_id` | Property linked to the conversation |
| `role` | User or assistant |
| `message` | Message text |

The chat workflow's load and save nodes must point to the same chat-history table.

## Endpoint contracts

- `POST /webhook/evaluate-property`: multipart form with PDF field `property_pdf`. A successful response includes `property_id`, `report_json`, `report_html`, and market summary information.
- `POST /webhook/property-chat`: JSON object with `property_id`, `sessionId`, and `chatInput`. A successful response includes `answer` and the associated IDs.
- `GET /webhook/property-ui`: serves the UI. The original UI assumes n8n is at `http://localhost:5678`.

Use production webhook URLs after publishing. The evaluation and chat workflows rely on external providers and stored data; importing the JSON alone does not make an independent copy of the project data or credentials.
