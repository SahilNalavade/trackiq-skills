# TrackIQ Skills

Each `.md` file in this directory is a skill that the TrackIQ audit pipeline can execute. The n8n workflow fetches these files from GitHub at runtime — adding a new skill means committing a new file.

## Adding a New Skill

1. Create a new `.md` file in this directory
2. Add YAML frontmatter with required fields
3. Write the system prompt as the markdown body
4. Commit and push — the pipeline picks it up automatically

## Frontmatter Reference

```yaml
---
id: my_skill_id              # Unique identifier (snake_case, must match filename without .md)
name: Human Readable Name     # Display name
model: claude-sonnet-4-6      # Claude model to use
max_tokens: 4096               # Max response tokens
trigger_when: "always"         # When this skill should run (see rules below)
score_field: myScore           # JSON field name where the skill returns its 0-100 score
output_fields: "warnings,criticalIssues"  # Which response sections this skill contributes to
boundary: "What this skill MUST NOT do"   # Anti-overlap guard
---
```

## Trigger Rules

The `trigger_when` field determines when a skill runs based on the user's business context:

| Rule | Meaning |
|------|---------|
| `"always"` | Runs for every audit |
| `"adPlatforms.length > 0"` | Runs when user has at least one ad platform selected |

The Router (Claude Haiku) reads these rules and decides which skills to activate per request. If the Router fails, deterministic fallback logic evaluates the rules directly.

## Prompt Guidelines

- The markdown body below the frontmatter becomes the system prompt
- Each skill has a "NOT YOUR JOB" section to prevent overlap with other skills
- End the prompt with a "Return ONLY valid JSON:" section showing the expected output schema
- Include a score field matching `score_field` from frontmatter (0-100)
- Be specific in the prompt: tell the AI to name exact tags, triggers, and variables

## Current Skills (v2 — 5 non-overlapping)

| Skill | Model | Triggers When | Score Field | Answers |
|-------|-------|--------------|-------------|---------|
| `compliance_consent` | Sonnet 4.6 | Always | `complianceScore` | "Is it legal?" |
| `tracking_coverage` | Sonnet 4.6 | Has ad platforms | `coverageScore` | "Is it there?" |
| `container_architecture` | Haiku 4.5 | Always | `architectureScore` | "Is it clean?" |
| `data_quality` | Haiku 4.5 | Always | `dataQualityScore` | "Is the data right?" |
| `performance_security` | Sonnet 4.6 | Always | `perfSecurityScore` | "Is it safe & fast?" |

## Cost per audit (~medium container)

```
Router (Haiku):                    $0.001
Compliance & Consent (Sonnet):     $0.075
Tracking Coverage (Sonnet):        $0.075
Container Architecture (Haiku):    $0.016
Data Quality (Haiku):              $0.016
Performance & Security (Sonnet):   $0.075
                                   ──────
TOTAL:                             ~$0.26
```
