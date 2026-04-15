# Manual UAT Checklist

Capture manual UAT after both automated validation phases pass.

## Minimum Record

- slice number and title
- environment used
- flows exercised
- result for each flow: pass or fail
- known gaps or deferred issues

## Checklist Template

Use a compact checklist like this:

```md
Manual UAT

- Flow: [user journey or operator action]
  Result: pass | fail
  Notes: [short evidence]

- Flow: [user journey or operator action]
  Result: pass | fail
  Notes: [short evidence]

Known gaps

- [gap or `none`]
```

Prefer flows tied directly to the slice acceptance criteria. For UI slices, include the user-visible path. For orchestration or runtime slices, include the operator path and any end-to-end cross-service behavior that automation does not fully prove.
