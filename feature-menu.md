# Feature Menu — Chunking Wiki

## Workflows

| # | Trigger | What happens |
|---|---------|--------------|
| 1 | `ingest raw/filename` | Read → takeaways → sources/ → concepts/ → check contradictions → update index.md + log.md |
| 2 | Ask a question | Read index.md → relevant pages → synthesize → offer to save in wiki/queries/ |
| 3 | `verify claim X` | Write/run wiki/code/ → update verification table in index.md only |
| 4 | `lint the wiki` | Find orphans, contradictions, stale claims → append wiki/lint/lint-[date].md |

## Rules

| Rule | Scope |
|------|-------|
| Never modify raw/ | All workflows |
| Verification status lives only in index.md | Verification |
| Report contradictions before writing any wiki file | Ingest |
| Append log.md on every ingest or major update | All workflows |
| Touch the minimum number of files per task | Always |
| No numbered prefixes on filenames | Naming |
