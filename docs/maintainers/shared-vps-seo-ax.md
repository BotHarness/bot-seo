# Shared-VPS SEO AX

This is the canonical internal workflow for human-reviewed SEO work across coding-agent clients and development machines. AX here means the agent's connection, tool discovery, task ownership, evidence, and handoff experience; it is not autonomous publishing.

## Authority and boundaries

- The existing VPS deployment of our OpenSEO fork holds shared project context, research logs, and reports. GitHub holds task claims, content changes, review, and release evidence. Local files are working copies, not a second SEO authority.
- Follow `.agents/skills/keyword-research/SKILL.md`, `.agents/skills/seo-report/SKILL.md`, and `.agents/skills/openseo-review-web-content/SKILL.md` for their respective workflows. This runbook adds shared-server access and coordination, not replacement skills.
- The product adapter is `docs/agents/seo-ax.md` in BotHarness; the site adapter is `planning/seo-comparison-series/seo-ax-workflow.md` in deepseekbot-site. Each repository owns its checks and human-review gate.
- Read-only discovery is the default. Shared-data writes require task-specific authorization; `save_keywords` requires explicit confirmation. DataForSEO calls can incur real charges even in local-noauth mode. Get approval for the operation and spend cap before calling; stop at the cap. No service restart, rebuild, port exposure, account operation, or publishing is implied.

## 1. Connect to the existing server

Use BotHarness's machine-local, gitignored `vps.local.env` and its tracked example. Each machine needs its own authorized SSH identity and verified host key; another machine's successful connection is not your authorization. Never copy private keys, tokens, or profile stores between repositories.

For clients that need an HTTP MCP endpoint, forward only to local loopback using an unused local port:

```sh
set -a
source /path/to/BotHarness/vps.local.env
set +a
ssh -o BatchMode=yes -o StrictHostKeyChecking=yes -o ExitOnForwardFailure=yes \
  -N -L 127.0.0.1:13001:127.0.0.1:3001 \
  "$VPS_SSH_USER@$VPS_SSH_HOST"
```

Configure the current client's session-local HTTP MCP connection to `http://127.0.0.1:13001/mcp`, or execute JSON-RPC requests over SSH against VPS loopback. Confirm the deployed port privately before use; 3001 is the 2026-10-11 observation, not a permanent API guarantee. Do not expose local-noauth to the internet or bind the forwarding port to all interfaces. Do not disable host-key checking or replace an occupied local port. Stop only your own tunnel when finished.

The shell example is for a POSIX shell (including an independently configured WSL environment); it does not prove Windows-native client setup. Codex, Claude and other clients must use their own current HTTP MCP configuration interface and this session's forwarded URL, not copy another client's private configuration wholesale.

With the tunnel running, this discovery call verifies the MCP path without invoking a paid provider or modifying project data:

```sh
curl --fail-with-body http://127.0.0.1:13001/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  --data '{"jsonrpc":"2.0","id":1,"method":"tools/list","params":{}}'
```

Completion: SSH reaches the expected configured account, the endpoint returns MCP tool discovery, and the client actually calls the shared endpoint. A reachable browser page alone is insufficient. If setup fails, report the missing authorization/configuration without publishing its values or probing alternate credentials.

## 2. Resolve the shared project before research

1. Call live `tools/list`; use its schemas rather than remembered local-source schemas. Handle JSON or SSE transport as described in `verify-local-mcp`.
2. Call `list_projects`; select the existing project by `deepseekbot.app`. Never create a duplicate because its display name, default market, or local configuration differs. If missing or ambiguous, stop for the owner's selection.
3. Call `get_project_context` and `list_reports` for that exact returned project ID. Reuse relevant research within 30 days before metered calls; refresh sooner when the fact changes or the task needs newer evidence.
4. Read the corresponding GitHub issue, comments, open PRs, and active claims. An agent label is not a lock. Claim one non-overlapping task before writes.

Completion: record privately which project and connection were used, and publicly only a sanitized task reference and evidence summary. Keep internal IDs, endpoint URLs, account identity, raw project context, and credentials out of public issues/PRs.

## 3. Specify a bounded experiment

Record these fields in the issue and shared report, redacting private fields from GitHub:

| Field          | Required meaning                                                                                                              |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Task identity  | Repository/issue, stable agent task ID, owner, branch, report subject/date                                                    |
| Reader outcome | Installation → first low-risk task → product selection; reuse existing guides                                                 |
| Search slice   | Query, search engine, country/region, language, device, page and locale                                                       |
| Baseline       | Data source, retrieval date, coverage window, filters, impressions/clicks/position; missing volume, KD or ranking = `unknown` |
| Evidence       | Official source/version/date; author report versus site execution; task inputs, outputs, limitations                          |
| Change         | One keyword/page per ranking experiment, hypothesis, reversible diff, exact base/head SHA                                     |
| Review         | Human owner, factual/editorial review, genuine UI evidence where needed, repo checks, CI                                      |
| Tracking       | Confirmed deployment time, comparable before/after windows, due date, decision and next action                                |

Chinese and English are independent market slices with paired factual boundaries, not one universal US-English-desktop configuration. Confirm each slice's market filters; do not silently modify project defaults or existing trackers. A bilingual release can share one PR, but evaluate its locale experiments separately. Do not infer demand from a competitor's stars or media exposure.

Business outcome is an effective trial: installation plus completion of a low-risk real task. Installation-link clicks are intent, not activation. Until instrumentation proves completion, report it as unknown. Rankings/impressions are supporting metrics, not proof of business conversion. Existing GSC ranking-loop criteria remain separate.

## 4. Coordinate and save without overwriting other agents

Use a coordinator for shared-context updates and tracking decisions; workers own distinct research/evidence/channel slices. There is no documented atomic lease or cross-client transaction in this workflow. A claim alone cannot prevent a race.

Workers deliver separate task artifacts to the coordinator. Before any shared write, re-read current context/reports and reconcile intervening work. Only one designated writer updates shared context at a time; if ownership is uncertain, stop that write. Use schema-supported `update_project_context` updates (including `appendResearchLog` when available), not a whole-context replacement assembled from an old snapshot. Re-read after writes; an uncertain response requires reconciliation before retrying.

Follow `seo-report` for complete, self-contained HTML and unique subject/date titles. `list_reports` first: a genuinely new task/period gets a new report; correcting the same report uses its `reportId` only with its writer's ownership and authorization. Report replacement has no version history: preserve the approved prior evidence before replacement. No near-duplicate report spam. HTML must not make external requests; aim below 80 KB and respect the deployed hard limit. Return the private report link only through an authorized private surface.

Completion: the shared server has the report and relevant research-log entry, a readback confirms them, and GitHub records a sanitized result/next step. If writes are not authorized, mark the local artifact pending shared save; do not claim centralization complete.

## 5. Human review, publish separately, then observe

Parallel coding-agent output is capacity, not permission to operate community accounts. Use one transparent project identity, distinct useful channel adaptations, and human review; no fake endorsements, duplicate keyword pages, or paid promotion. Chinese and English each need their own channel/intent planning, including English Reddit and YouTube where relevant.

Open a focused PR, run the repository's checks, and obtain the required review before merge. Merge and deployment are different events. Begin tracking only after confirmed publication; do not create a schedule or restart the VPS as part of this documentation workflow.

Respect the existing seven-day site observation/cooldown policy: no edits to the observed target page, compare matching GSC filters/windows, and mark insufficient data inconclusive. Apply the existing issue's achieved/active/revert rules rather than inventing a new scoring system; a drop greater than three average-position places invokes its revert rule, not permission to bypass PR review. Close each run with measured results, uncertainty, and the next authorized action.

## Connection proof boundary

On 2026-10-11, one Mac client's authorized SSH path reached the existing VPS; MCP discovery returned 76 tools, and `list_projects`, `get_project_context`, and `list_reports` successfully read the existing deepseekbot.app project. This is shared-read smoke evidence only: it does not prove every client's MCP configuration, Windows access, report writes, paid-provider availability, scheduled tracking, installation conversion, or production deployment. The observed project default was US/English; it is not authorization to ignore Chinese markets.
