Title: feat(rules): add SQL review rules

## Description

`.sql` is already on the review allowlist, but `system_rules.json` had no SQL mapping, so schema migrations, stored procedures, and hand-written queries all received `default.md`. This change adds a built-in SQL rule document and maps `**/*.sql` to it.

`internal/config/rules/rule_docs/sql.md` is a precision-first, dialect-aware review prompt (PostgreSQL, MySQL, SQLite, SQL Server). It defers formatting to linters such as `sqlfluff` and covers destructive or irreversible migrations, locking and online-migration hazards, NULL semantics, join/filter correctness, constraints and integrity, non-sargable predicates and `SELECT *`, and dynamic SQL / `SECURITY DEFINER` privilege issues.

Only `**/*.sql` is mapped. The allowlist admits `.sql` and does not admit `.psql`, `.pgsql`, or `.ddl`; those extensions were not added. The MyBatis mapper XML rule (`**/*{mapper,dao}*.xml` → `mapper_dao_xml.md`) is unchanged and still wins for mapper XML.

The `**/*.sql` entry is last in `path_rule_map`. It does not overlap a more specific glob. Removing that single mapping makes `Resolve` fall back to `default.md` for `db/migrate/20260101_add_users.sql` and `sql/reports/monthly.sql`, `TestResolveDetail_SystemSQLPatternMatch` reports `Pattern: default`, `TestSystemRulesIntegrity/no_orphan_files` flags `sql.md`, and all five review-rules locales fail because they still list `**/*.sql`. Restoring the mapping makes those tests pass again.

## Type of Change

- [ ] Bug fix (non-breaking change that fixes an issue)
- [x] New feature (non-breaking change that adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to change)
- [ ] Refactoring (no functional changes)
- [ ] Documentation update
- [ ] CI / Build / Tooling

## How Has This Been Tested?

- `go test ./internal/config/...` — 5 packages ok (`allowlist`, `rules`, `template`, `testconnection`, `toolsconfig`)
- `go test ./internal/config/rules -count=1 -run 'TestResolve_DefaultRules|TestResolveDetail_SystemSQLPatternMatch|TestSystemRulesIntegrity|TestReviewRulesGuideLists'` — PASS, including `db/migrate/20260101_add_users.sql`, `sql/reports/monthly.sql`, `SQL/REPORTS/MONTHLY.SQL`, `**/*.sql` allowlisted, and all five locale tables
- Fail-before (mapping removed in a worktree): the SQL ResolveDetail rows, `no_orphan_files`, and `TestReviewRulesGuideListsEverySystemRulePattern` fail with `Pattern: default` / orphan `sql.md` / missing glob; restore then PASS
- `make build` — PASS (`dist/opencodereview`)
- `make test` — PASS, 2736 `--- PASS:` lines, 0 `--- FAIL:`, 26 packages ok
- `make check` — PASS (license headers, english-check, `go mod tidy`, `gofmt -s -w`, `go vet`)
- `gofmt -l .` — empty
- `go vet ./internal/config/...` — PASS
- Manual: `/tmp/ocr-main` (upstream `d77d70b`) `rules check` on `db/migrate/20260101_add_index.sql` and `sql/reports/monthly.sql` prints `Pattern: default`. `/tmp/ocr-sql` (`6cc1fa1`) prints `Pattern: **/*.sql` and the SQL rule. `mapper/UserMapper.xml` stays `Pattern: **/*{mapper,dao}*.xml` on both binaries. `ocr review --preview` (no model call) lists the three files as will-review and does not print the per-file rule body.

Not verified: a live review that calls a model, CI on this branch, and files with extensions other than `.sql`.

- [x] `make test` passes locally
- [x] Manual testing (describe below)

## Checklist

- [x] My code follows the project's coding style (`go fmt`, `go vet`)
- [x] I have performed a self-review of my code
- [x] I have added tests that prove my fix is effective or my feature works
- [x] New and existing unit tests pass locally with my changes
- [x] I have updated the documentation accordingly (if applicable)
- [ ] I have signed the CLA
- [ ] I did not use AI/LLM to create this PR, or I disclosed the tool/model below and reviewed its output; I did not attribute commits to AI and will answer maintainer questions and review comments myself without AI/LLM.

<!-- Please disclose the model used here if AI was used. -->

## Related Issues

No existing issue covers SQL review rules.

## Demo

![demo](https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1114/sql-review-rules/preview.gif)

<video src="https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1114/sql-review-rules/out.mp4" controls width="720"></video>
