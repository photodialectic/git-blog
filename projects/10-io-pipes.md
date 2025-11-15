# [IO Pipes: Data Transformation Workbench](/tools/pipeline)

A browser-based pipeline builder that lets me chain together regex filters, JSON queries, fetchers, and templating blocks, then save those pipelines for later reuse.

## Overview

`mono/tools/src/pages/pipeline/[[...id]].js` renders the IO Pipes experience: a responsive layout with themed UI, Auth0 user controls, and three columns—Input, Pipeline, Available Transformers. Everything runs client-side, but pipelines can be persisted through API routes backed by MySQL, which makes it easy to share a link or revisit a previous flow.

## Key Features

### Guided Pipeline Builder
- **Transformer Catalog**: `TransformerSelector` exposes categories (Text, Data, Network, Template, Time) and icons for each transformer; search + tabs make it easy to find the right block.
- **Dynamic Options**: Selecting a transformer attaches the matching React options component (`RegexFilterOptions`, `JSONQueryOptions`, `FetchFromURLOptions`, etc.) so you can configure it inline without modal hopping.
- **Pipeline Flow UI**: `TransformerPipeline` renders each block with step numbers, arrows, and detail toggles; you can inspect inputs/outputs, edit options, or remove steps without losing state.

### Rich Input & Output Controls
- **Smart Textarea**: `TransformerInput` auto-expands between 5–20 rows, exposes clipboard paste/clear shortcuts, and shows character/line/word counts for quick sanity checks.
- **Live Execution**: Every edit re-runs the pipeline (sequentially awaiting async transformers), updating the final output pane with Text vs HTML rendering options.
- **Clipboard Helpers**: A copy button handles the output buffer with success feedback, so sharing results is one click away.

### Persistence & Collaboration
- **Auth-Aware Actions**: `User` component pulls Auth0 session info, exposes pipeline dropdowns (New, existing ids), and gates save/fork/delete actions.
- **Database Storage**: `/tools/api/pipelines` POST/PUT/DELETE routes upsert rows in the `pipeline` table (JSON columns for transformers/settings) after verifying the user via `upsertUser`.
- **Local Recovery**: `usePipeline` mirrors transformers into `localStorage`, giving you an offline recovery path even before you save to the database.

## Technical Architecture

```
mono/tools/
├── src/pages/pipeline/[[...id]].js  # Page shell + layout
├── src/hooks/usePipeline.js         # Context (input, transformers, options)
├── src/components/                  # UI building blocks
│   ├── TransformerInput.js
│   ├── TransformerPipeline.js
│   ├── TransformerSelector.js
│   ├── PipelineActions.js
│   ├── ThemeToggle.js
│   └── User.js
├── src/transformers/                # Individual transformer functions
└── src/pages/api/pipelines/         # Authenticated CRUD endpoints
```

- **Transformers as Modules**: Each file in `src/transformers` exports an async `process` function; `usePipeline` injects these into pipeline steps so every transformer can maintain its own options schema.
- **MySQL Query Layer**: `src/database/query.js` handles UUID conversions, JSON serialization, and enforces per-user access when updating or deleting a pipeline.
- **Theme + User Experience**: `ThemeToggle` switches CSS variables, and `User` shows avatar/login/logout links plus quick actions (Save, Fork, Delete, Load Last).

IO Pipes has become my go-to scratchpad for massaging logs, APIs, cron strings, and templated payloads without jumping into a local script each time.
