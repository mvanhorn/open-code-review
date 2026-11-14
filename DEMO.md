# SQL review rules demo

This branch is evidence only. The product change lives on `feat/sql-review-rules`.

`out.mp4` and `preview.gif` are an animated replay of `commands.log`. The yellow banner on every frame is the required label: **Animated replay of real captured command output**.

## What was captured

Scratch git repo `/tmp/sql-rules-scratch` with:

- `db/migrate/20260101_add_index.sql` — `CREATE INDEX` without `CONCURRENTLY`
- `sql/reports/monthly.sql`
- `mapper/UserMapper.xml`

Binaries:

- `/tmp/ocr-main` — upstream `main` at `d77d70b`
- `/tmp/ocr-sql` — `feat/sql-review-rules` at `6cc1fa1`

`commands.log` records exact commands, stdout/stderr, and exit codes for:

1. **Before:** `ocr-main rules check` on both `.sql` files → `Pattern: default` and `default.md`
2. **After:** `ocr-sql rules check` on both `.sql` files → `Pattern: **/*.sql` and the SQL rule body (preamble, destructive migrations, locking, NULL, joins, constraints, performance, dynamic SQL)
3. **Edge:** `mapper/UserMapper.xml` still → `Pattern: **/*{mapper,dao}*.xml` / `mapper_dao_xml.md` on both binaries
4. **Preview:** `ocr-sql review --preview` lists the three files as will-review; it does not print the per-file rule body
5. **Tests:** `go test ./internal/config/rules` for the new mapping rows, integrity, and docs-table checks (`exit: 0`)
6. **Fail-before:** drop `**/*.sql` from `path_rule_map` in a worktree → `Pattern: default`, orphan `sql.md`, docs table mismatches (`exit: 1`); restore then `exit: 0`

The replay paginates through those blocks over 22 s and holds the restore confirmation for the last ~2 s.

## Media

| File | Spec |
| --- | --- |
| `out.mp4` | 1920x1080, 30 fps, 720 frames, 24.0 s, H.264, yuv420p, 4.2 MB |
| `preview.gif` | 720x405, 12 fps, 288 frames, 24.0 s, 2.1 MB |

See `logs/ffprobe.txt` for the probe dump.
