# IEF Migration Plan

## Migration Mapping

| Current Repository | Target Repository | Action | Reason |
|---|---|---|---|
| `brantzh6/AgentSDLC` | `everwork-ai/IEF-Governance` | Transfer + rename | Same identity upgraded into governance layer; SDLC becomes internal governance pack. |
| `brantzh6/llm-knowledge-base` | `everwork-ai/IEF-Knowledge` | Transfer + rename | Same identity upgraded into knowledge layer. |
| `brantzh6/Harness-4-AIAgents` | `everwork-ai/IEF-Adapters` | Transfer + rename | Same identity upgraded into adapter layer. |
| `brantzh6/claude-worker` | `everwork-ai/IEF-Runners` | Transfer + rename | Preserve history; restructure Claude Code implementation under `runners/claudecode/`. |
| `none` | `everwork-ai/IEF-Program` | Create new | HQ for roadmap, RFCs, decisions, GitHub operating model. |
| `none` | `everwork-ai/IEF-Operations` | Create new | First missing runtime capability layer. |
| `none` | `everwork-ai/IEF-Protocol` | Create new | Protocol can start internal-first and split when stable. |

## Migration Rules

1. **Do not fork.** Fork implies upstream/downstream relationships. IEF migration is re-architecture, not contribution.
2. **Use transfer + rename** when preserving history is valuable.
3. **Use new repository** when the layer does not exist yet.
4. **Do not reuse deprecated names** (`IEF-Work`, `IEF-Workers`, `IEF-Orchestration`, `IEF-Execution`, `IEF-Forge`).

## Post-Migration Verification

1. All repositories exist under `everwork-ai`.
2. Old URLs redirect correctly.
3. Issues and PRs are preserved for transferred repositories.
4. GitHub Actions are reviewed.
5. Repository secrets are reviewed.
6. Webhooks are reviewed.
7. Branch protection is reviewed.
8. Topics, labels, milestones exist in all repos.
9. IEF Command Center exists.
10. Seed issues exist and are added to IEF Command Center.
11. IEF-Program has README, docs, RFCs, and ADRs.
