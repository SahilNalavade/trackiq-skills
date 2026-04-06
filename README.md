# TrackIQ - AI-Powered GTM Auditor

**Live app:** [trackiq-ten.vercel.app](https://trackiq-ten.vercel.app)

## Architecture

```mermaid
flowchart TD
    A["User uploads GTM JSON + fills questionnaire"] --> B["POST /webhook/trackiq-smart"]
    B --> C["Receive Request\nWebhook"]
    C --> D["Config\nInject API key + GitHub repo URL"]
    D --> E["Validate Request\nNormalize payload, fill defaults"]

    E --> F["Build Classifier Body"]
    E --> G["Fetch Skills Index\nGET skills-index.json from GitHub"]

    F --> H["Classifier\nClaude Haiku 4.5\nCost: $0.002"]
    H --> I["Merge\nCombine classifier output + skill index"]
    G --> I

    I --> J["Match Skills\nHaiku matches needs to skill files\nCost: $0.001"]

    J --> K["Run Skills Parallel\nPromise.allSettled"]

    K --> S1["compliance.md\nConsent, PII, GDPR/CCPA"]
    K --> S2["coverage.md\nEvent parity, remarketing"]
    K --> S3["architecture.md\nNaming, folders, orphans"]
    K --> S4["data-quality.md\nSchema, hardcoded values"]
    K --> S5["performance.md\nHTML risks, sequencing"]

    S1 --> L["Build Consolidator Body\nParse all skill results\nExtract scores and issues"]
    S2 --> L
    S3 --> L
    S4 --> L
    S5 --> L

    L --> M["Consolidator Agent\nClaude Sonnet 4.6\nCost: $0.03"]
    M --> N["Format Output\nMap to frontend schema"]
    N --> O["Send to User\nHTTP 200 JSON + CORS"]
    O --> P["Frontend renders results"]
    P --> Q["Save audit to Supabase"]

    G -.->|GitHub| GH["SahilNalavade/trackiq-skills"]
    K -.->|GitHub| GH
    H -.->|API| CL["Anthropic API"]
    J -.->|API| CL
    S1 -.->|API| CL
    S2 -.->|API| CL
    S3 -.->|API| CL
    S4 -.->|API| CL
    S5 -.->|API| CL
    M -.->|API| CL
    Q -.->|DB| SB["Supabase\nAuth + Audits table"]

    style H fill:#fef3c7,stroke:#f59e0b,color:#1e293b
    style J fill:#fef3c7,stroke:#f59e0b,color:#1e293b
    style S1 fill:#dbeafe,stroke:#3b82f6,color:#1e293b
    style S2 fill:#dbeafe,stroke:#3b82f6,color:#1e293b
    style S3 fill:#dbeafe,stroke:#3b82f6,color:#1e293b
    style S4 fill:#dbeafe,stroke:#3b82f6,color:#1e293b
    style S5 fill:#dbeafe,stroke:#3b82f6,color:#1e293b
    style M fill:#fce7f3,stroke:#ec4899,color:#1e293b
    style GH fill:#d1fae5,stroke:#10b981,color:#1e293b
    style CL fill:#fef3c7,stroke:#f59e0b,color:#1e293b
    style SB fill:#dbeafe,stroke:#3b82f6,color:#1e293b
```

## How It Works

1. **User** uploads a GTM container JSON export and fills a 6-step business context questionnaire
2. **Classifier** (Haiku) describes what analysis the container needs
3. **Matcher** (Haiku) maps those needs to available skill files from this repo
4. **5 Skills** run in parallel (Haiku), each analyzing a different domain
5. **Consolidator** (Sonnet) cross-references all findings into a strategic report
6. **Frontend** renders scores, roadmap, quick wins, issues, and detailed skill breakdowns

## Skills

| File | Domain | Question It Answers |
|------|--------|-------------------|
| `compliance.md` | Consent and Privacy | Is it legal? |
| `coverage.md` | Tracking Coverage | Is tracking complete? |
| `architecture.md` | Container Organization | Is it organized? |
| `data-quality.md` | Data Layer Integrity | Is the data correct? |
| `performance.md` | Performance and Security | Is it safe and fast? |

Skills are loaded from this repo at runtime. To add a new skill, push a `.md` file and add an entry to `skills-index.json`. Zero workflow changes needed.

## Cost Per Audit

| Component | Model | Cost |
|-----------|-------|------|
| Classifier | Haiku 4.5 | $0.002 |
| Matcher | Haiku 4.5 | $0.001 |
| 5 Skills | Haiku 4.5 x5 | $0.100 |
| Consolidator | Sonnet 4.6 | $0.030 |
| **Total** | | **~$0.13** |

## Stack

- **Frontend:** Vanilla JS on Vercel
- **Backend:** n8n workflow (self-hosted)
- **AI:** Claude API (Haiku for skills, Sonnet for consolidation)
- **Auth + DB:** Supabase (email/password + Google OAuth, RLS)
- **Skills:** This GitHub repo (dynamic loading)
