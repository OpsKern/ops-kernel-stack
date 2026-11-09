# Ansible Public Collection

> **Global context:** `~/.claude/CLAUDE.md` (ARIA model, workflow, hooks).

Public Ansible collection for homelab infrastructure automation.

## File Structure

- roles/ — 4 roles: common, restic, restic_backup, server_hardening (each with Molecule tests)
- playbooks/ — 7 deployment playbooks (deploy-restic-fleet, deploy-node-exporter, deploy-loki, etc.)
- remediation/ — 6 auto-fix playbooks (restart-container, cleanup-disk, caddy-reload, etc.)
- templates/ — Jinja2 templates for alert rules and promtail config
- .github/workflows/ci.yml — CI: syntax check + ansible-lint + Molecule matrix

## Commands

- Syntax check: `ansible-playbook playbooks/<name>.yml --syntax-check -i inventory.yml`
- Lint: `ansible-lint playbooks/ remediation/ roles/`
- Test role: `cd roles/<role> && molecule test`
- Deploy: `ansible-playbook playbooks/<name>.yml --check --limit <host>` then without --check

## Conventions

- kebab-case for playbooks/roles, snake_case for tasks/vars
- All tasks idempotent, safe for --check mode
- Variables in defaults/main.yml per role, shared in group_vars/all
- Handlers for service restarts via notify:
- Tags on all tasks (audit, snapshot, patch, rollback)
- Molecule + Docker for role testing

<!-- ce:begin -->
## Compound Engineering loop

Work here runs the CE loop: `/ce-brainstorm → /ce-plan → /ce-doc-review → /ce-work → /ce-code-review → /ce-commit-push-pr`,
then `/ce-compound` when the work produced a lesson the final code won't show.
In an interactive session `ce-work` is blocked unless a brainstorm ran earlier in the session (`pre-skill-ce-plan-gate.sh`).
`ce-doc-review` is not gated; run it anyway.
PRs are merged by `~/.claude/scripts/ce-merge-pr.sh <pr> -R <owner/repo>`, not by hand.
This repo's CE artifacts (plans, solutions, brainstorms) live under `docs/ce` (`docs_root` in `.compound-engineering/config.yaml`; each repo sets its own).
Review criteria: `CODING_STANDARDS.md`. New repo? `ce-bootstrap-repo.sh <path> --kind <k>`.

CE review subagent models (plugin 3.37.0+; Anthropic guidance: Haiku 5.5 for reading-heavy work, Sonnet 5.5 where the work needs judgment):
- Keep each reviewer's default tier. Do not set `review_subagents.claude` in CE config: it puts every reviewer on one model and drops the tiers.
- `ce-doc-review`: run `coherence-reviewer` on `sonnet`, not the cheapest tier. Finding contradictions across a plan's sections is judgment work.
- Pass the tier's model on every reviewer spawn, and list the model each reviewer actually ran on in the review's Coverage.
- `plan_model` / `brainstorm_model: fable` is intended. If elevation falls back to the session model, say so in the result.
<!-- ce:end -->
