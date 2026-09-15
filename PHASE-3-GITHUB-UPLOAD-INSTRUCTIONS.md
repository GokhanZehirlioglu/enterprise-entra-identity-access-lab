# Phase 3 GitHub Upload Instructions

Repository:
`GokhanZehirlioglu/enterprise-entra-identity-access-lab`

Upload / replace the package contents at the same repository-relative paths:

```text
README.md
docs/05-conditional-access.md
docs/06-emergency-access.md
tests/test-matrix.md
tests/failure-scenarios.md
images/03-conditional-access/
```

The evidence screenshots are already sanitized for public use.

Recommended commit message:

```text
docs: complete Phase 3 Conditional Access validation and RCA
```

After upload, verify:

- all 17 PNG evidence files render,
- `images/03-conditional-access/README-EVIDENCE.md` renders,
- README links to docs 05 and 06 work,
- test matrix contains T25–T36,
- failure scenario links resolve,
- no TAP passcode, real tenant domain, IP address, request ID, subscription ID or operator email is visible.

Do not upload raw screenshots from the chat in place of these sanitized versions.


## Existing Phase 2 stale status

After the Phase 3 upload, apply the three small historical closeout edits in:

`PHASE-2-CLOSEOUT-PATCH.md`

This is the only stale Phase 2 repository item found during the live review.
