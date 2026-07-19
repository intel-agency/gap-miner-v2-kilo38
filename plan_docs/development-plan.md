```markdown
# Gap Mining Platform — Autonomous Agent Development Plan v1.0

**Document Type:** Implementation Handoff Specification
**Audience:** Autonomous AI Coding Agents (Claude Code, Cursor, OpenAI Codex)
**Supervising Engineer:** Veteran .NET Architect (Human-in-the-Loop)
**Source References:**
- `Strategic Feasibility and Execution Plan for AI-Accelerated Micro-SaaS Ecosystems.md` (strategic context)
- `Gap Mining Architecture Plan v2.md` (architectural baseline)
**Version:** 1.0
**Date:** 2026-07-03
**Status:** Ready for Agent Execution

---

## 0. Document Purpose

This document is an **executable development plan** designed for consumption by autonomous AI agent teams. Unlike the source architecture document (which describes *what* to build), this plan specifies *how* to build it, with atomic, verifiable, independently-completable tasks.

**Every task in this document is designed to:**
1. Fit within a single agent context window
2. Have deterministic, testable acceptance criteria
3. Produce a verifiable artifact (file, test, endpoint)
4. Be resumable if interrupted

Agents MUST NOT deviate from task boundaries without explicit human approval.

---

## 1. Strategic Context (Read-Only Background)

The Gap Mining Platform is an **internal intelligence engine**, not a customer-facing product. Its sole mission is to:

1. Scrape 1-to-3-star reviews of competitor apps from digital marketplaces (Shopify App Store, Chrome Web Store, G2, Apple App Store).
2. Cluster and analyze negative reviews using LLMs to identify **substantial, monetizable feature gaps**.
3. Surface ranked opportunities via an internal Blazor dashboard for human strategic review.

**The platform does NOT build, ship, or monetize micro-SaaS applications.** That is a downstream activity governed by the strategic feasibility document.

---

## 2. Agent Operating Principles

### 2.1 Mandatory Rules

| Rule | Description |
|---|---|
| **R1** | Never modify files outside the task's declared scope. |
| **R2** | Never invent package versions. Use the exact versions in §3. |
| **R3** | Every public method MUST have an XML doc comment and at least one unit test. |
| **R4** | All async I/O MUST use `CancellationToken` propagation. |
| **R5** | Never hardcode secrets. Use `IConfiguration` or Aspire secret parameters. |
| **R6** | Prefer `record` types for DTOs, `class` for EF entities, `interface` for service contracts. |
| **R7** | Commit messages must follow Conventional Commits: `feat(scope): description`. |
| **R8** | If a task is ambiguous, STOP and request clarification. Do not guess. |

### 2.2 Context Window Strategy

- Each task is scoped to ≤ 8 files.
- If a task exceeds this, the agent must request task decomposition.
- Reference files are provided inline; agents should not assume external docs exist.

---

## 3. Technology Stack (Exact Versions — Do Not Deviate)

| Component | Technology | Version |
|---|---|---|
| Runtime | .NET SDK | 8.0.x (LTS) |
| Orchestration | .NET Aspire | 8.2.x |
| Web UI | Blazor Web App (Interactive Server) | 8.0 |
| API | ASP.NET Core Minimal API | 8.0 |
| Database | PostgreSQL | 16.x |
| Vector Extension | pgvector | 0.7.x |
| ORM | Entity Framework Core | 8.0.x |
| Queue / Cache | Redis (StackExchange.Redis) | 7.x |
| Job Scheduling | Hangfire | 1.8.x |
| AI Orchestration | Microsoft.SemanticKernel | 1.20.x |
| LLM Provider | Azure OpenAI (GPT-4o) **or** Anthropic Claude 3.5 Sonnet | Latest stable |
| Embeddings | `text-embedding-3-large` (OpenAI) or `voyage-3` | Latest stable |
| Scraping | Apify API | v2 REST |
| HTTP Client | Refit | 7.x |
| Validation | FluentValidation | 11.x |
| Testing | xUnit + NSubstitute + Testcontainers | Latest stable |
| Code Quality | SonarAnalyzer.CSharp + StyleCop.Analyzers | Latest stable |

---

## 4. Repository Layout (Exact Structure)

```
GapMiner/
├── GapMiner.sln
├── Directory.Build.props                  # Shared MSBuild properties
├── Directory.Packages.props               # Central Package Management
├── .editorconfig
├── global.json                            # Pin SDK version
├── README.md
│
├── src/
│   ├── GapMiner.AppHost/                  # Aspire orchestrator
│   │   ├── Program.cs
│   │   └── appsettings.json
│   │
│   ├── GapMiner.ServiceDefaults/          # Shared Aspire service defaults
│   │   └── Extensions.cs
│   │
│   ├── GapMiner.Domain/                   # Pure domain entities (no EF refs)
│   │   ├── Entities/
│   │   │   ├── CompetitorTarget.cs
│   │   │   ├── Review.cs
│   │   │   └── FeatureGap.cs
│   │   ├── ValueObjects/
│   │   │   └── SeverityScore.cs
│   │   └── Enums/
│   │       └── MarketplaceKind.cs
│   │
│   ├── GapMiner.Infrastructure/           # EF Core, Redis, Apify, Semantic Kernel
│   │   ├── Persistence/
│   │   │   ├── GapMinerDbContext.cs
│   │   │   ├── Configurations/
│   │   │   ├── Migrations/
│   │   │   └── Repositories/
│   │   ├── Queueing/
│   │   │   ├── IJobQueue.cs
│   │   │   └── RedisJobQueue.cs
│   │   ├── Scraping/
│   │   │   ├── IApifyClient.cs
│   │   │   └── ApifyClient.cs
│   │   └── AI/
│   │       ├── IGapAnalyzer.cs
│   │       ├── SemanticKernelGapAnalyzer.cs
│   │       └── Prompts/
│   │           ├── EmbeddingPrompt.txt
│   │           ├── MapReviewsPrompt.txt
│   │           └── ReduceGapsPrompt.txt
│   │
│   ├── GapMiner.Application/              # Use cases / command handlers
│   │   ├── Commands/
│   │   │   ├── IngestTarget/
│   │   │   ├── RunScraper/
│   │   │   └── AnalyzeReviews/
│   │   ├── Queries/
│   │   │   └── GetFeatureGaps/
│   │   └── DTOs/
│   │
│   ├── GapMiner.Api/                      # Minimal API gateway
│   │   ├── Program.cs
│   │   ├── Endpoints/
│   │   │   ├── TargetsEndpoints.cs
│   │   │   ├── JobsEndpoints.cs
│   │   │   └── GapsEndpoints.cs
│   │   └── Middleware/
│   │
│   ├── GapMiner.ScraperWorker/            # Background worker: scraping
│   │   ├── Program.cs
│   │   └── Jobs/
│   │       └── ScrapeReviewsJob.cs
│   │
│   ├── GapMiner.AIWorker/                 # Background worker: LLM analysis
│   │   ├── Program.cs
│   │   └── Jobs/
│   │       ├── EmbedReviewsJob.cs
│   │       └── AnalyzeGapsJob.cs
│   │
│   └── GapMiner.Web/                      # Blazor dashboard
│       ├── Program.cs
│       ├── Components/
│       │   ├── Layout/
│       │   ├── Pages/
│       │   │   ├── Dashboard.razor
│       │   │   ├── Targets.razor
│       │   │   ├── Jobs.razor
│       │   │   └── OpportunityMatrix.razor
│       │   └── Shared/
│       └── wwwroot/
│
├── tests/
│   ├── GapMiner.Domain.Tests/
│   ├── GapMiner.Infrastructure.Tests/
│   ├── GapMiner.Application.Tests/
│   ├── GapMiner.Api.Tests/
│   └── GapMiner.Integration.Tests/        # End-to-end with Testcontainers
│
└── docs/
    ├── prompts/                            # Version-controlled prompt library
    └── adr/                                # Architecture Decision Records
```

---

## 5. Naming & Code Conventions

| Element | Convention | Example |
|---|---|---|
| Namespace | `GapMiner.{Layer}.{Feature}` | `GapMiner.Infrastructure.Persistence` |
| Entity class | PascalCase, singular | `CompetitorTarget` |
| Repository | `I{Entity}Repository` / `{Entity}Repository` | `ICompetitorTargetRepository` |
| Command | `{Verb}{Noun}Command` | `IngestTargetCommand` |
| Command Handler | `{Verb}{Noun}Handler` | `IngestTargetHandler` |
| Endpoint route | kebab-case, plural nouns | `/api/v1/targets` |
| Database column | snake_case (EF mapping) | `marketplace_url` |
| Redis queue key | `gapminer:{domain}:{action}` | `gapminer:scrape:pending` |

---

## 6. Phase 0 — Environment & Foundation (Days 1–3)

### T-0.1: Repository Bootstrap
**Owner:** Any agent
**Prerequisites:** None
**Scope:** Root-level scaffolding only.

**Acceptance Criteria:**
- [ ] `global.json` pins SDK `8.0.x` with rollForward policy `latestFeature`.
- [ ] `Directory.Packages.props` enables Central Package Management and declares ALL versions from §3.
- [ ] `Directory.Build.props` enables `<TreatWarningsAsErrors>true</TreatWarningsAsErrors>`, nullable reference types, and implicit usings.
- [ ] `.editorconfig` enforces file-scoped namespaces, 4-space indentation, CRLF line endings.
- [ ] Empty solution `GapMiner.sln` with solution folders `src` and `tests` builds with `dotnet build` exit code 0.

**Reference:**
```xml
<!-- Directory.Packages.props (excerpt) -->
<ItemGroup>
  <PackageVersion Include="Aspire.Hosting" Version="8.2.2" />
  <PackageVersion Include="Aspire.Hosting.PostgreSQL" Version="8.2.2" />
  <PackageVersion Include="Aspire.Hosting.Redis" Version="8.2.2" />
  <PackageVersion Include="Microsoft.SemanticKernel" Version="1.20.0" />
  <PackageVersion Include="Npgsql.EntityFrameworkCore.PostgreSQL" Version="8.0.10" />
  <PackageVersion Include="Pgvector.EntityFrameworkCore" Version="0.2.0" />
  <PackageVersion Include="Hangfire.Core" Version="1.8.14" />
  <PackageVersion Include="Refit.HttpClientFactory" Version="7.2.1" />
  <PackageVersion Include="FluentValidation" Version="11.10.0" />
</ItemGroup>
```

---

### T-0.2: Aspire AppHost Skeleton
**Owner:** Any agent
**Prerequisites:** T-0.1
**Scope:** `GapMiner.AppHost` project only.

**Acceptance Criteria:**
- [ ] `GapMiner.AppHost/Program.cs` declares resources: PostgreSQL (with `pgvector` extension enabled), Redis, and placeholder project references for Api, Web, ScraperWorker, AIWorker.
- [ ] Running `dotnet run --project src/GapMiner.AppHost` launches the Aspire dashboard on `https://localhost:18888` (or equivalent) and shows PostgreSQL + Redis as "Running".
- [ ] Connection strings for Postgres and Redis are accessible via `IConfiguration` in downstream projects.

**Reference:**
```csharp
// GapMiner.AppHost/Program.cs
var builder = DistributedApplication.CreateBuilder(args);

var postgres = builder.AddPostgres("postgres")
                      .WithDataVolume()
                      .AddDatabase("gapminer");

var redis = builder.AddRedis("redis");

var api = builder.AddProject<Projects.GapMiner_Api>("api")
                 .WithReference(postgres)
                 .WithReference(redis);

builder.AddProject<Projects.GapMiner_ScraperWorker>("scraper-worker")
       .WithReference(postgres)
       .WithReference(redis);

builder.AddProject<Projects.GapMiner_AIWorker>("ai-worker")
       .WithReference(postgres)
       .WithReference(redis);

builder.AddProject<Projects.GapMiner_Web>("web")
       .WithReference(api);

builder.Build().Run();
```

---

### T-0.3: ServiceDefaults Project
**Owner:** Any agent
**Prerequisites:** T-0.2

**Acceptance Criteria:**
- [ ] `GapMiner.ServiceDefaults` project adds OpenTelemetry tracing, logging, and health checks (`/health`, `/alive`).
- [ ] All downstream projects reference `ServiceDefaults` and call `builder.AddServiceDefaults()`.

---

## 7. Phase 1 — Domain & Data Layer (Days 4–8)

### T-1.1: Domain Entities
**Owner:** Any agent
**Prerequisites:** T-0.3
**Scope:** `GapMiner.Domain` only.

**Acceptance Criteria:**
- [ ] `CompetitorTarget` entity has: `Id` (Guid), `Name`, `MarketplaceKind` (enum), `MarketplaceUrl` (Uri wrapper), `CreatedAt`.
- [ ] `Review` entity has: `Id`, `CompetitorTargetId`, `StarRating` (1–5, validated), `ReviewText` (non-empty), `ReviewAuthor`, `DatePosted`, `SourceReviewId` (external idempotency key), `Embedding` (`float[]` nullable).
- [ ] `FeatureGap` entity has: `Id`, `CompetitorTargetId`, `Title`, `DetailedDescription`, `SeverityScore` (double 1–10), `MentionFrequency` (int ≥ 0), `SuggestedTechStack`, `ActionableImplementationPlan`, `IdentifiedAt`.
- [ ] `SeverityScore` value object enforces range [1.0, 10.0] via factory method.
- [ ] All entities are immutable where possible (init-only setters).
- [ ] Unit tests validate value-object constraints.

---

### T-1.2: EF Core DbContext & Migrations
**Owner:** Any agent
**Prerequisites:** T-1.1
**Scope:** `GapMiner.Infrastructure/Persistence`.

**Acceptance Criteria:**
- [ ] `GapMinerDbContext` configures all three entities via IEntityTypeConfiguration classes (one per entity, in `Configurations/`).
- [ ] `pgvector` extension enabled via `HasPostgresExtension("vector")`.
- [ ] `Review.Embedding` mapped as `vector(3072)` (for `text-embedding-3-large`).
- [ ] Unique composite index on `Review(CompetitorTargetId, SourceReviewId)` for idempotent inserts.
- [ ] Index on `FeatureGap(SeverityScore DESC, MentionFrequency DESC)`.
- [ ] Initial migration `InitialSchema` generated and applies cleanly against PostgreSQL 16 + pgvector.
- [ ] `ApplyMigrations` extension method runs migrations at AppHost startup (dev only).

---

### T-1.3: Repository Layer
**Owner:** Any agent
**Prerequisites:** T-1.2

**Acceptance Criteria:**
- [ ] `ICompetitorTargetRepository` with: `AddAsync`, `GetByIdAsync`, `GetAllAsync`, `ExistsByUrlAsync`.
- [ ] `IReviewRepository` with: `BulkInsertIgnoreDuplicatesAsync` (uses `ON CONFLICT DO NOTHING`), `GetUnembeddedAsync(targetId, limit)`, `GetByTargetIdAsync`.
- [ ] `IFeatureGapRepository` with: `AddAsync`, `GetRankedAsync(sortBy, skip, take)`, `GetByTargetIdAsync`.
- [ ] All async methods accept `CancellationToken`.
- [ ] Integration tests using Testcontainers.PostgreSql verify each method.

---

### T-1.4: Redis Queue Abstraction
**Owner:** Any agent
**Prerequisites:** T-0.3

**Acceptance Criteria:**
- [ ] Generic `IJobQueue<TCommand>` interface with `EnqueueAsync`, `DequeueAsync`, `LengthAsync`.
- [ ] `RedisJobQueue<TCommand>` implementation using `StackExchange.Redis` List (FIFO via `RPUSH` / `LPOP`).
- [ ] Commands are serialized as JSON with `System.Text.Json` (camelCase, ignore nulls).
- [ ] `DequeueAsync` is blocking with configurable timeout (BRPOP pattern via polling).
- [ ] Integration test with Testcontainers.Redis validates enqueue/dequeue round-trip.

---

## 8. Phase 2 — Scraper Pipeline (Days 9–14)

### T-2.1: Apify Refit Client
**Owner:** Any agent
**Prerequisites:** T-0.3
**Scope:** `GapMiner.Infrastructure/Scraping`.

**Acceptance Criteria:**
- [ ] `IApifyClient` Refit interface defines:
  - `POST /v2/acts/{actorId}/run-sync-get-dataset-items` → triggers actor, returns run metadata.
  - `GET /v2/actor-runs/{runId}` → polls status.
  - `GET /v2/datasets/{datasetId}/items` → retrieves JSON items.
- [ ] Bearer token injected via `Authorization` header delegating handler from `IConfiguration["Apify:Token"]`.
- [ ] Polly retry policy: 3 retries on 429/5xx with exponential backoff.
- [ ] Typed response DTOs for Apify run status and dataset items.
- [ ] Unit test with mocked `HttpMessageHandler` validates request shape.

---

### T-2.2: Marketplace Actor Registry
**Owner:** Any agent
**Prerequisites:** T-2.1

**Acceptance Criteria:**
- [ ] Static registry maps `MarketplaceKind` to Apify Actor ID and required input schema:
  - `ShopifyAppStore` → `apify/shopify-scraper` (or verified equivalent actor ID)
  - `ChromeWebStore` → `apify/google-play-scraper` (adapt for CWS if actor exists; otherwise mark as TODO)
  - `AppleAppStore` → `apify/apple-app-store-reviews-scraper`
  - `G2` → `apify/g2-scraper`
- [ ] Each entry includes: `ActorId`, `ReviewFieldName`, `StarRatingFieldName`, `AuthorFieldName`, `DateFieldName`.
- [ ] If an actor ID is unknown, the entry MUST be marked `Status = NeedsVerification` and runtime MUST throw a `MarketplaceNotSupportedException` rather than silently failing.

**⚠ Agent Note:** Verify Apify actor IDs against live Apify Store before committing. If uncertain, flag for human review.

---

### T-2.3: ScrapeReviewsJob (ScraperWorker)
**Owner:** Any agent
**Prerequisites:** T-2.1, T-2.2, T-1.3, T-1.4

**Acceptance Criteria:**
- [ ] `ScrapeReviewsJob` registered with Hangfire, triggered by `ScrapeTargetCommand` from Redis queue.
- [ ] Job lifecycle:
  1. Dequeue command.
  2. Resolve actor via registry.
  3. Trigger Apify run with target URL and review filter (stars 1–3).
  4. Poll run status every 15s with 60-minute max timeout.
  5. Download dataset items.
  6. Map to `Review` entities using registry field mapping.
  7. Bulk-insert with idempotency (ignore duplicates).
  8. Enqueue `AnalyzeReviewsCommand { CompetitorTargetId }` onto analysis queue.
  9. Update `CompetitorTarget.LastScrapedAt`.
- [ ] All failures produce a structured log event with `JobId`, `TargetId`, `FailureReason`.
- [ ] Idempotency: re-running the same `ScrapeTargetCommand` does not duplicate reviews.
- [ ] Unit tests for mapping logic. Integration test with Testcontainers for queue round-trip.

---

## 9. Phase 3 — Intelligence Pipeline (Days 15–22)

### T-3.1: Semantic Kernel Bootstrap
**Owner:** Any agent
**Prerequisites:** T-0.3

**Acceptance Criteria:**
- [ ] `SemanticKernelBuilder` extension registers:
  - `IChatCompletionService` for Azure OpenAI GPT-4o (or Anthropic Claude via community connector).
  - `ITextGenerationService` for embeddings (`text-embedding-3-large`).
- [ ] Configuration sourced from `IConfiguration["AI:Provider"]` with switchable provider (Azure vs Anthropic).
- [ ] API keys sourced from environment variables or Aspire secret parameters — NEVER from `appsettings.json`.
- [ ] Smoke test: single prompt `"Return the word PONG"` returns `"PONG"` with `Usage` token counts logged.

---

### T-3.2: EmbedReviewsJob
**Owner:** Any agent
**Prerequisites:** T-3.1, T-1.3

**Acceptance Criteria:**
- [ ] Fetches up to 200 unembedded reviews per batch from `IReviewRepository.GetUnembeddedAsync`.
- [ ] Batched embedding calls (max 100 texts per API call to respect rate limits).
- [ ] Stores 3072-dim vectors into `Review.Embedding`.
- [ ] Retries transient 429/5xx errors with exponential backoff (Polly).
- [ ] Re-enqueues itself if more unembedded reviews remain.
- [ ] Logs `ReviewsEmbedded` count per batch.

---

### T-3.3: Semantic Clustering (pgvector KNN)
**Owner:** Any agent
**Prerequisites:** T-3.2

**Acceptance Criteria:**
- [ ] Repository method `ClusterReviewsByTargetAsync(targetId, k)` executes:
  ```sql
  SELECT id, review_text,
         (embedding <=> centroid) AS distance
  FROM reviews
  WHERE competitor_target_id = @targetId AND embedding IS NOT NULL
  ORDER BY distance;
  ```
- [ ] Implements simple k-means clustering in C# using centroid initialization from random sample.
- [ ] Returns `List<ReviewCluster>` with `ClusterId`, `RepresentativeReviews` (top 5 by proximity), `Size`.
- [ ] Unit test with synthetic vectors validates cluster assignment.

---

### T-3.4: AnalyzeGapsJob (Map-Reduce)
**Owner:** Any agent
**Prerequisites:** T-3.3, T-1.3, T-1.4
**Scope:** `GapMiner.AIWorker/Jobs/AnalyzeGapsJob.cs` + `Infrastructure/AI/SemanticKernelGapAnalyzer.cs`.

**Acceptance Criteria:**
- [ ] Dequeues `AnalyzeReviewsCommand`.
- [ ] Retrieves clusters for the target.
- [ ] **Map phase:** For each cluster, calls LLM with `MapReviewsPrompt.txt` (see §13) to extract candidate gaps as JSON.
- [ ] **Reduce phase:** Aggregates all candidates, deduplicates by title similarity (>0.8 cosine), calls LLM with `ReduceGapsPrompt.txt` to produce final ranked list.
- [ ] Output strictly conforms to schema:
  ```json
  {
    "gaps": [
      {
        "title": "string (max 80 chars)",
        "detailedDescription": "string",
        "severityScore": "number 1-10",
        "mentionFrequency": "integer",
        "suggestedTechStack": "string",
        "actionableImplementationPlan": "string"
      }
    ]
  }
  ```
- [ ] JSON validated with `System.Text.Json` schema validator before persistence.
- [ ] Invalid LLM output triggers ONE retry with stricter prompt; second failure logs and skips.
- [ ] Persists `FeatureGap` records via repository.

---

## 10. Phase 4 — API Gateway (Days 23–25)

### T-4.1: Minimal API Endpoints
**Owner:** Any agent
**Prerequisites:** T-1.3, T-1.4

**Acceptance Criteria:**
- [ ] `POST /api/v1/targets` — ingests new competitor target; validates URL; enqueues `ScrapeTargetCommand`.
- [ ] `GET /api/v1/targets` — lists targets with pagination.
- [ ] `GET /api/v1/targets/{id}` — detail view.
- [ ] `POST /api/v1/targets/{id}/reanalyze` — manually triggers `AnalyzeReviewsCommand`.
- [ ] `GET /api/v1/gaps` — paginated, sortable (`sortBy=severity|frequency`), filterable by `targetId`.
- [ ] `GET /api/v1/jobs` — Hangfire job history (last 100).
- [ ] `GET /health` and `/alive` endpoints via ServiceDefaults.
- [ ] All endpoints documented via Swagger/OpenAPI (Swashbuckle).
- [ ] Input validation via FluentValidation with consistent `ProblemDetails` error shape.
- [ ] Integration tests for each endpoint using `WebApplicationFactory`.

---

## 11. Phase 5 — Blazor Dashboard (Days 26–28)

### T-5.1: Layout & Navigation
**Owner:** Any agent
**Prerequisites:** T-0.3

**Acceptance Criteria:**
- [ ] Blazor Web App with Interactive Server render mode.
- [ ] Sidebar navigation: Dashboard, Targets, Jobs, Opportunity Matrix.
- [ ] Uses MudBlazor or Radzen.Blazor component library for consistent design system.
- [ ] Responsive layout (mobile-friendly).

---

### T-5.2: Targets Page
**Owner:** Any agent
**Prerequisites:** T-4.1

**Acceptance Criteria:**
- [ ] Form to add new target (Name, Marketplace dropdown, URL).
- [ ] Client + server validation (URL must match marketplace pattern).
- [ ] DataGrid listing existing targets with status badges (Never Scraped / Scraping / Analyzed).
- [ ] Row actions: "Re-scrape", "Re-analyze", "Delete".

---

### T-5.3: Jobs Page
**Owner:** Any agent
**Prerequisites:** T-4.1

**Acceptance Criteria:**
- [ ] Live view of Hangfire job queue (polling every 5s via `Timer`).
- [ ] Color-coded status: Queued (gray), Running (blue), Succeeded (green), Failed (red).
- [ ] Click-through to Hangfire dashboard for details.

---

### T-5.4: Opportunity Matrix
**Owner:** Any agent
**Prerequisites:** T-4.1
**Scope:** `OpportunityMatrix.razor`.

**Acceptance Criteria:**
- [ ] Scatter plot (Severity X vs Frequency Y) using `Blazor-ApexCharts` or `AntDesign.Charts`.
- [ ] Each point = one `FeatureGap`; hover shows title + description.
- [ ] Click on point navigates to detail modal with full `ActionableImplementationPlan`.
- [ ] Filter by marketplace, target, min severity.
- [ ] Export to CSV button.

---

## 12. Phase 6 — Integration Testing & Hardening (Days 29–30)

### T-6.1: End-to-End Integration Test
**Owner:** Any agent
**Prerequisites:** All prior tasks

**Acceptance Criteria:**
- [ ] `GapMiner.Integration.Tests` project uses Testcontainers for Postgres + Redis.
- [ ] Single test `FullPipeline_ShouldProduceGaps` performs:
  1. Seed 50 synthetic reviews into DB directly.
  2. Invoke `AnalyzeGapsJob` with mocked LLM (NSubstitute) returning deterministic JSON.
  3. Assert: correct number of `FeatureGap` records persisted with expected titles.
- [ ] Test must pass in CI within 60 seconds.

---

### T-6.2: Observability & Logging
**Owner:** Any agent
**Prerequisites:** T-0.3

**Acceptance Criteria:**
- [ ] Structured logging via Serilog with JSON sink.
- [ ] OpenTelemetry traces for: scrape jobs, LLM calls, embedding calls.
- [ ] Custom metrics: `gapminer.reviews.scraped`, `gapminer.gaps.identified`, `gapminer.llm.tokens.consumed`.
- [ ] Aspire dashboard displays all traces and metrics.

---

### T-6.3: Error Handling & Idempotency Audit
**Owner:** Any agent
**Prerequisites:** All prior tasks

**Acceptance Criteria:**
- [ ] Every job has a `MaxRetryAttempts` attribute (default 3).
- [ ] Failed jobs write to `DeadLetterQueue` Redis list for human review.
- [ ] Idempotency verified: re-running scrape + analyze on same target produces no duplicate `Review` or `FeatureGap` records.

---

## 13. Prompt Engineering Library (Version-Controlled)

Prompts MUST be stored as `.txt` files in `src/GapMiner.Infrastructure/AI/Prompts/` and loaded at runtime via `EmbeddedResource` or `IConfiguration`.

### 13.1 `MapReviewsPrompt.txt`

```
You are a product strategist analyzing negative software reviews.

INPUT: A cluster of {clusterSize} thematically similar 1-to-3-star reviews for the product "{productName}" in the "{marketplace}" marketplace.

REVIEWS:
{reviews}

TASK: Identify candidate "substantial feature gaps" — features that are HIGHLY desired by users but demonstrably absent from the product. A substantial gap is:
- A MISSING workflow or capability (NOT a bug fix, cosmetic tweak, or performance issue).
- Mentioned by multiple users in this cluster.
- Something that, if implemented, would directly address the core complaint.

OUTPUT FORMAT (strict JSON, no markdown fences):
{
  "candidates": [
    {
      "title": "Short name (max 80 chars)",
      "description": "2-3 sentence description of the missing capability",
      "estimatedSeverity": 1-10,
      "mentionCountInCluster": integer,
      "representativeQuotes": ["quote1", "quote2"]
    }
  ]
}

If no substantial gaps exist in this cluster, return {"candidates": []}.
```

### 13.2 `ReduceGapsPrompt.txt`

```
You are a senior product strategist. You are given a deduplicated list of candidate feature gaps extracted from multiple review clusters of the product "{productName}".

CANDIDATES:
{candidatesJson}

TASK: Produce a FINAL ranked list of substantial feature gaps. For each gap:
1. Merge candidates that describe the same underlying missing capability.
2. Score severity 1-10 based on: user frustration intensity, correlation with churn, and breadth of mention.
3. Produce an actionable implementation plan suitable for a senior developer using AI coding agents.

OUTPUT (strict JSON):
{
  "gaps": [
    {
      "title": "string",
      "detailedDescription": "string",
      "severityScore": number,
      "mentionFrequency": integer,
      "suggestedTechStack": "string",
      "actionableImplementationPlan": "string (3-5 sentences)"
    }
  ]
}

Return at most 10 gaps, sorted by severityScore DESC.
```

### 13.3 `EmbeddingPrompt.txt`

```
Represent the following software review for semantic similarity search:

Review: {reviewText}
```

---

## 14. Known Risks & Agent Mitigations

| Risk | Impact | Mitigation |
|---|---|---|
| **LLM output violates JSON schema** | Job fails silently | T-3.4 enforces strict schema validation + 1 retry + dead-letter queue. |
| **Apify actor ID incorrect** | Scraping returns garbage | T-2.2 requires human verification flag; agent MUST NOT guess. |
| **pgvector extension missing in container** | EF migration fails | T-0.2 uses `HasPostgresExtension("vector")`; Aspire Postgres image must include pgvector (use `pgvector/pgvector:pg16` image). |
| **Semantic Kernel API change** | Compilation breaks | Pin exact version in `Directory.Packages.props`; never use floating versions. |
| **Context window overflow on large review sets** | LLM truncation | T-3.4 chunking strategy: max 8,000 tokens per map call. |
| **Race condition on re-scrape** | Duplicate reviews | T-1.2 unique composite index + `ON CONFLICT DO NOTHING`. |
| **Agent hallucinating Apify response shapes** | Runtime failures | T-2.1 requires mocked HTTP tests; agent must inspect real Apify docs if uncertain. |
| **Secret leakage in logs** | Security | Serilog config MUST scrub `*Password*`, `*Token*`, `*Key*` properties. |

---

## 15. Definition of Done (Global)

A task is DONE only when ALL of the following are true:

1. ✅ All acceptance criteria checkboxes pass.
2. ✅ `dotnet build` succeeds with zero warnings.
3. ✅ `dotnet test` succeeds for the affected project AND `GapMiner.Integration.Tests`.
4. ✅ Code passes StyleCop + SonarAnalyzer with zero new violations.
5. ✅ Public APIs have XML doc comments.
6. ✅ Conventional commit pushed to feature branch.
7. ✅ PR description includes: task ID, summary of changes, test evidence.

---

## 16. Parallel Execution Map

Agents MAY execute the following task groups in parallel (no cross-dependencies):

```
Group A (Data):      T-0.1 → T-0.2 → T-0.3 → T-1.1 → T-1.2 → T-1.3
Group B (Queue):     T-0.1 → T-0.3 → T-1.4
Group C (AI Setup):  T-0.3 → T-3.1
```

**Gate 1 (Day 8):** Groups A, B, C merge. Then:
```
Group D (Scrape):    T-2.1 → T-2.2 → T-2.3
Group E (Analyze):   T-3.1 → T-3.2 → T-3.3 → T-3.4
```

**Gate 2 (Day 22):** Groups D, E merge. Then:
```
Group F (API):       T-4.1
Group G (UI):        T-5.1 → T-5.2 → T-5.3 → T-5.4 (parallel where possible)
```

**Gate 3 (Day 28):** All groups merge → T-6.1, T-6.2, T-6.3.

---

## 17. Handoff Checklist for Human Reviewer

Before approving any agent PR, the human reviewer MUST verify:

- [ ] No hardcoded secrets anywhere in the diff.
- [ ] No invented Apify actor IDs (must match live Apify Store).
- [ ] No `vibe-coded` anti-patterns: no dynamic `eval`, no unvalidated LLM output, no swallowed exceptions.
- [ ] EF migrations are reversible (paired `Up`/`Down` methods).
- [ ] All LLM prompts are stored as files, not inline strings.
- [ ] Test coverage ≥ 80% for `Application` and `Infrastructure` layers.

---

## 18. Escalation Protocol

If an agent encounters ambiguity NOT covered by this document, it MUST:

1. STOP work on the ambiguous task.
2. Produce a short markdown file `docs/adr/NNN-{topic}.md` documenting the decision point.
3. Request human clarification before proceeding.

**NEVER guess on:**
- Apify actor IDs or response schemas
- LLM provider API keys or endpoint URLs
- Marketplace policy interpretations
- Security-sensitive design choices

---

*End of document. Agents: begin execution at T-0.1.*
```
