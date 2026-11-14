Title: feat(rules): add Ruby review rules

## Description

`.rb` (and `.rake` / `.gemspec`) are already on the extension allowlist, so Ruby files are reviewed, but `system_rules.json` had no Ruby mapping. Those paths fell through to `default.md`.

This adds `internal/config/rules/rule_docs/ruby.md` and maps:

- `**/*.{rb,rake}` → `ruby.md`
- `**/*.gemspec` → `ruby.md`
- `**/{Gemfile,Rakefile}` → `ruby.md`

`**/*.erb` is unchanged and still resolves to `erb.md`.

The Ruby rule is a precision-first language doc in the same shape as the other built-in language rules. It defers formatting and naming to RuboCop / standardrb, then covers defect classes that actually change behaviour: nil/`false`/`Hash#[]` vs `fetch`, frozen-string and argument mutation, broad `rescue` / `ensure`+`return` / unbounded `retry`, ActiveRecord N+1 / raw SQL / mass assignment / callbacks / `update_column`, `send`/`eval`/`Marshal.load`/`YAML.load`/`html_safe`/`system`, and class-level mutable state plus `return` in a Proc versus a lambda. It does not change shared review prompts.

## Type of Change

- [ ] Bug fix (non-breaking change that fixes an issue)
- [x] New feature (non-breaking change that adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to change)
- [ ] Refactoring (no functional changes)
- [ ] Documentation update
- [ ] CI / Build / Tooling

## How Has This Been Tested?

- [x] `make test` passes locally
- [x] Manual testing (describe below)

`make test`: 26 packages `ok`, 2735 `--- PASS:` lines, 0 `--- FAIL:` lines (`LC_ALL=C go test -v -race -count=1`).

`go test ./internal/config/...`: 5 packages `ok` (allowlist, rules, template, testconnection, toolsconfig).

`make check`: license-check, english-check, `gofmt -s -w .`, `go vet` — passed. `gofmt -l .` produced no paths. `go vet ./internal/config/...` clean.

`make build` produced `./dist/opencodereview`.

Resolve rows added for `app/models/user.rb`, `lib/tasks/cleanup.rake`, `mygem.gemspec`, `Gemfile`, `Rakefile`, `lib/Rakefile`, and `app/views/users/show.html.erb`. Load-bearing check: the three Ruby `path_rule_map` entries were removed and `TestResolve_DefaultRules` failed on all six Ruby paths (they resolved to default.md `Correctness`); `app/views/users/show.html.erb` still passed on `erb.md`. Restoring the map made `TestResolve_DefaultRules`, `TestSystemRulesIntegrity`, and `TestReviewRulesGuide*` pass again.

Manual: binaries from upstream `main` (`d77d70b`) and this branch in a scratch repo.

```
/tmp/ocr-main rules check app/models/user.rb
# Source: System built-in
# Pattern: default

/tmp/ocr-ruby rules check app/models/user.rb
# Source: System built-in
# Pattern: **/*.{rb,rake}
# first heading: Nil, False, and Absent Values

/tmp/ocr-ruby rules check lib/tasks/cleanup.rake
# Pattern: **/*.{rb,rake}

/tmp/ocr-ruby rules check Gemfile
# Pattern: **/{Gemfile,Rakefile}

/tmp/ocr-ruby rules check app/views/users/show.html.erb
# Pattern: **/*.erb

/tmp/ocr-ruby review --preview
# Will review (4): Gemfile, app/models/user.rb, app/views/users/show.html.erb, lib/tasks/cleanup.rake
```

`--preview` lists files; it does not print the per-file rule body. Rule bodies were inspected with `ocr rules check`.

Not run: a live LLM review (no model call), `make coverage`, or a full Rails application beyond the four scratch paths.

## Checklist

- [x] My code follows the project's coding style (`go fmt`, `go vet`)
- [x] I have performed a self-review of my code
- [x] I have added tests that prove my fix is effective or my feature works
- [x] New and existing unit tests pass locally with my changes
- [x] I have updated the documentation accordingly (if applicable)
- [ ] I have signed the CLA
- [ ] I did not use AI/LLM to create this PR, or I disclosed the tool/model below and reviewed its output; I did not attribute commits to AI and will answer maintainer questions and review comments myself without AI/LLM.

## Related Issues

No existing issue tracks Ruby-specific review rules.

## Demo

![demo](https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1114/ruby-review-rules/preview.gif)

<video src="https://raw.githubusercontent.com/mvanhorn/pr-demo-assets/main/osc-bulk-1114/ruby-review-rules/out.mp4" controls width="720"></video>
