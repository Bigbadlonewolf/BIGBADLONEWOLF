# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

The **GitHub profile README repository** for `Bigbadlonewolf`. The repo name matches the username, so GitHub renders `README.md` at the top of the public profile page. There is no application here — three markdown surfaces and nothing else.

| File | Role |
| --- | --- |
| `README.md` | The profile page itself. Positioning, how-I-work evidence, links into the cases |
| `decisions.md` | Index of the four design cases, each with its decision and rejected alternative in one paragraph |
| `design-cases/case-{1..4}-*.md` | The cases in full |

No build, no tests, no dependencies, no CI. Nothing to run.

## Publishing

**A push to `main` is the publish.** GitHub re-renders the public profile immediately; there is no build step to fail and no staging copy. Remote is `Bigbadlonewolf/BIGBADLONEWOLF`, branch `main`. Confirm before pushing.

The only meaningful pre-push check is that GitHub can render what you wrote — relative links resolve from the repo root, and GitHub markdown supports no templating (see § The two live copies).

## The two live copies — read this before editing any case

The same four design cases are published **twice, from two different repos**, and the copies are deliberately not identical:

```
projects/BIGBADLONEWOLF/design-cases/case-N-*.md      -> GitHub profile
projects/hugo-site/lanre-site/content/design-cases/   -> lanreoluokun.com
```

| | this repo | hugo-site |
| --- | --- | --- |
| Wrapper | `# H1` in the body, plus an italic line linking to the rendered version on the site | Hugo frontmatter (`title`, `recordID`, `status`, `date`, `summary`, `description`); no body H1, the layout supplies it |
| Diagrams | none — **zero `{{< >}}` shortcodes**, GitHub renders them as literal text | two `{{< diagram >}}` shortcodes per case, pulling hand-authored SVG |
| Size | ~1.4 KB larger per case | carries the same content as SVG |

The prose bodies match. The wrappers cannot. **A change to a case's argument needs the matching edit in the other repo**, and neither file can be copied over the other. Never paste a Hugo shortcode into this repo — it will render as `{{< diagram src="..." >}}` on the public profile.

Counts must stay in step too. `decisions.md` opens "Four scenarios", `README.md` links all four cases by filename, and the site's `design-cases/_index.md` says "Four reference design scenarios" in four separate places. Adding a fifth case means editing all of them.

## A third copy exists and is dead

`business-dev/github-profile/` at the workspace root mirrors `decisions.md` and `design-cases/` — and **editing it publishes nothing.** It is not a git repo; `git rev-parse --show-toplevel` resolves it to the workspace root, which has no commits and no remote. It is also stale: three cases against this repo's four, `case-4-secrets-credential-rotation.md` missing entirely, and its `decisions.md` opens "Three scenarios" while this one is on four. Edit here instead. Recorded as Traps §10 in the root `CLAUDE.md`.

## Content rules

The workspace-wide rule that career and marketing material stays out of code repos (root `CLAUDE.md` § Security defaults) does **not** apply here — this repo *is* the marketing surface. That rule exists to keep positioning out of `projects/<code>/docs/`; this is its intended destination.

What the existing writing commits to, and what an edit should not quietly undo:

- **Every decision carries the alternative that lost.** `decisions.md`: *"An answer without a rejected alternative is a preference, not a decision."*
- **Reversals stay on the page.** The README leads with ADR-001 for BankVault being superseded by ADR-005 after Privileged Access Manager reached GA. Do not smooth that into a success story.
- **The link and the State cell must agree, and both must match GitHub.** Check visibility with `gh repo list Bigbadlonewolf --json name,visibility` before writing either — never from memory. As of 2026-08-24 `bankvault`, `SecureVault`, `GCP-HARDENED-LANDING-ZONE`, `COMPLIANCE_AS_CODE` and `Vulnerability-Management` are all public and all linked. A repo that is genuinely private stays described, not linked: a link to a private repo 404s for every visitor while looking fine to the signed-in owner, which is why this is a rule and not a preference.
- **Claims are checkable or they are cut.** Figures like the 163 passing OPA tests are stated with the tool version and the date they were verified.

## Related

- Rendered cases: <https://bigbadlonewolf.github.io/Lanreoluokun.com/design-cases/> — built from `projects/hugo-site/lanre-site`, whose own `CLAUDE.md` sits one level up at `projects/hugo-site/CLAUDE.md`
- Workspace rules, traps, and the project roster: root `CLAUDE.md`
