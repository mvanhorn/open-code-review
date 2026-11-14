# Demo: Ruby review rules

Animated replay of real captured command output from `commands.log`.

## What was captured

Two binaries:

- `/tmp/ocr-main` built from upstream `main` (`d77d70b`)
- `/tmp/ocr-ruby` built from `feat/ruby-review-rules` (`718c363`)

Scratch git repo `/tmp/ruby-scratch` with `app/models/user.rb`, `lib/tasks/cleanup.rake`, `Gemfile`, and `app/views/users/show.html.erb`.

## Commands (see `commands.log`)

1. Upstream binary: `ocr-main rules check` on `.rb`, `.rake`, and `Gemfile` resolves `Pattern: default`. `.erb` already resolves `**/*.erb`.
2. Feature binary: `.rb` and `.rake` resolve `Pattern: **/*.{rb,rake}` and print `ruby.md` (starts at Nil, False, and Absent Values). `Gemfile` resolves `**/{Gemfile,Rakefile}`. `.erb` still resolves `**/*.erb` / `erb.md`.
3. `ocr-ruby review --preview` in that repo (no LLM call) lists the four changed files as will-review.
4. `go test` of `TestResolve_DefaultRules` passes with the mapping present.
5. Mapping removed: the six new Ruby rows fail (they fall through to default.md Correctness). Mapping restored: Resolve, integrity, and review-rules table tests pass.
6. `gofmt -l .` empty; `go vet ./internal/config/...` clean.

## Media

- `out.mp4`: HyperFrames HTML composition, 24 s, 1920x1080, 30 fps, 720 frames, H.264 yuv420p. Scrolls the captured log for 22 s and holds the last frame for 2 s. Banner text: `Animated replay of real captured command output`.
- `preview.gif`: derived from `out.mp4` with ffmpeg palettegen/paletteuse, 720x405, 12 fps, 24 s, 32-color palette, under 10 MB.
- `logs/ffprobe.txt`: probe output for both files.
- `screenshots/`: stills from `out.mp4`.

Full `make test` and fail-before verbose output are under `logs/`.
