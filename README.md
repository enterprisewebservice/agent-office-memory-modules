# agent-office-memory-modules

Library of memory-module content for the Governed Agent Platform. Each
module is a `.md` file that the OpenClaw runtime mounts into an agent's
workspace. Authored once here, referenced by N agents from their
per-agent gitops repos via `spec.memory.modules: []` on their
`AgentWorkstation` CR.

## Why this repo exists

Agent memory files (`AGENTS.md`, `USER.md`, `SKILL_*.md`, etc.) used to
be duplicated into each agent's gitops repo. With 5 agents and ~5 .md
files each, that's 25 places where a typo lives, with no propagation.
The Map view in agent-office (`/map` page) showed which files were
byte-identical across agents — those identical files are the obvious
candidates to deduplicate into modules here.

## What's a module?

One `.md` file at a path like `modules/<kind>/<name>.md` plus a
matching `MemoryModule` CR (added in slice B of the rollout) that gives
the file a stable resource name. The `<kind>` is one of the AAIF /
OpenClaw conventions:

| Kind | Output filename | Convention |
|---|---|---|
| `agents` | `AGENTS.md` | OpenAI's universal agent-instructions file (AAIF / Linux Foundation, Dec 2025). Read natively by Codex CLI, Claude Code, Cursor, GitHub Copilot, Gemini CLI, Goose, Devin. |
| `skill` | `SKILL_<name>.md` → `skills/<name>/SKILL.md` at runtime | Anthropic's Skills Open Standard (also AAIF). |
| `user` | `USER.md` | OpenClaw runtime convention; user profile / preferences. |
| `tools` | `TOOLS.md` | OpenClaw runtime convention; tool allowlist. |
| `identity` | `IDENTITY.md` | OpenClaw runtime convention; persona declaration. |
| `soul` | `SOUL.md` | OpenClaw runtime convention; behavioral principles. |
| `custom` | `<filename>` declared on the module | Escape hatch for one-off content. |

The `agents/`, `skill/` kinds map to AAIF-blessed cross-tool standards
(no provider lock-in). The `user/`, `tools/`, `identity/`, `soul/`
kinds are OpenClaw-internal organization. Going forward we'll
concatenate the latter into `AGENTS.md` for portability while keeping
their authoring split for human clarity.

## Editing modules

This repo is the **source of truth for shared memory content**. To
change a shared file (e.g. tweak the standard `USER.md` for every
agent that references it), edit the relevant file in `modules/`,
commit, push. ArgoCD reconciles the corresponding ConfigMap in the
`agent-office` namespace. The MemoryModule controller (slice D)
re-renders every per-agent `agent-<name>-config` ConfigMap that
references the module. Pods pick up the change on next restart (or
ConfigMap volume re-mount, whichever comes first).

This repo is intentionally **separate** from the per-agent gitops
repos. When iterating on a single agent's composition or
agent-specific overrides, you open Dev Spaces from the agent's repo
(via the Map page) and don't see this library at all. When iterating
on shared content, you open Dev Spaces from this repo. Two distinct
workflows, two distinct mental modes.

## How content reaches the cluster

1. Push to `main` here.
2. The ArgoCD `agent-office-memory-modules` Application (defined in
   `enterprisewebservice/agent-office.git` at
   `cluster/memory-modules-app.yaml`) syncs this directory into the
   `agent-office` namespace.
3. `kustomize`'s `configMapGenerator` produces one ConfigMap per
   module: `mm-<kind>-<name>`, e.g. `mm-user-default-context`.
4. The ConfigMap is `disableNameSuffixHash: true` so cross-references
   from `MemoryModule` CRs (slice B) stay stable.
5. Verify in cluster: `oc get cm -n agent-office | grep ^mm-`.

## Slice rollout (current)

| Slice | Status | Description |
|---|---|---|
| A | **shipping now** | This repo + 4 seed modules + ArgoCD Application syncs content ConfigMaps into `agent-office` namespace. |
| B | next | Define the `MemoryModule` CRD; add a `*.module.yaml` per module that references the corresponding ConfigMap. |
| C | after B | Manual render: pick `onboarding-agent`, hand-write the rendered ConfigMap from referenced modules. Verify USER.md content matches the library after editing. |
| D | after C | Tiny Go controller in agent-office-server that watches AgentWorkstation + MemoryModule and renders per-agent CMs. Migrate all 5 agents. |
| E | after D | UI: "Memory Modules" tab in agent-office, with "Open Library in Dev Spaces" button. Map view shows which modules each agent references. |
