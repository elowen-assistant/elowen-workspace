# Slice Closeout Checklist

Do not call a slice complete until every item here is done.

## Child Repositories

- confirm the slice branch is fully committed
- confirm automated validation passed after refactoring
- confirm manual UAT summary is captured
- squash-merge the slice branch into `main`
- verify `main` points at the intended commit

## Workspace Repository

- update submodule pointers after child repo merges
- run any required integrated checks from the workspace root
- update `roadmap.md` to reflect slice completion and the next active slice
- create or refresh `codex/notes/YYYY-MM-DD-handoff.md`

## Handoff Note Contents

Include:

- current completed slice range
- next planned slice
- latest known-good revisions for touched repos
- validation and UAT summary
- resume commands or operational notes when they matter
- the next recommended step

## Final State

- all intended repos are on `main` or otherwise clearly documented
- no pending slice work remains uncommitted
- the workspace reflects the merged child repo commits
- roadmap and handoff docs match the actual shipped state
