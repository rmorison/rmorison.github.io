# rmorison.github.io — Operating Notes

Hugo site (Congo theme) for **rm.rmdashrf.net**. Deploys automatically via GitHub Actions
on every push to `main` — pushing main IS publishing.

## Two authoring pipelines — respect both

1. **ox-hugo (legacy, still live):** posts authored as subtrees in `org/all-posts.org`,
   exported by ox-hugo into `content/posts/<EXPORT_FILE_NAME>.md`.
   **Never hand-edit those exports** — the next export overwrites them.
2. **Hand-authored Hugo markdown (newer):** written directly in `content/posts/`.
   Each such file must carry an HTML comment right after the front matter marking it
   hand-authored, and its slug must not collide with any org `EXPORT_FILE_NAME`
   (grep `org/all-posts.org` before choosing a slug).

Match the ox-hugo front-matter shape for consistency: TOML (`+++`), with
`title`, `author = ["Rod Morison"]`, `date` (RFC3339 w/ TZ), `tags`, `draft`,
`topics`, `description`.

## Workflow

- Draft on a branch; merge/push to `main` only after Rod reviews (public site).
- Images: ox-hugo posts use `/ox-hugo/...` under `static/`; hand-authored posts may
  use `assets/` or `static/` per Hugo norms.
