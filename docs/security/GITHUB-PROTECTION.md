# Activate GitHub protection

The repository is owned by the personal account `Sajawal6363` (ID `145142282`).
The audit session authenticated as `Sameer447`, with push permission and
`admin: false`. GitHub rejected creation of the owner rule with HTTP 404.
No branch rules were active when inspected. These files do not activate rules.

After reviewing and merging the cleanup/CI PR, the owner must run:

```sh
gh auth switch --user Sajawal6363
python3 scripts/security/configure_github.py
```

If that account is not configured locally, authenticate as the owner first with
`gh auth login`. Do not share or commit a token. Administration write and Actions
write permissions are required for these API changes.

The script checks the authenticated account and repository ownership, updates
only the two named rulesets, and reads them back. Other rulesets are preserved.
Alternatively import the two JSON files from `.github/rulesets/` in
[repository rules settings](https://github.com/Sajawal6363/glimpse-connect/settings/rules).

## Resulting policy

- `main-owner-direct-push-only`: all contributors must use a pull request with
  the owner's CODEOWNERS approval, fresh approval after changes, approval of the
  latest push, and resolved review threads. Only the exact owner user ID has an
  always-bypass exception. No collaborator, bot, app, or generic admin role is
  exempted.
- `main-required-ci-and-history`: `CI required` and `CodeQL analysis`, from the
  GitHub Actions integration, must succeed on an up-to-date revision. Force
  pushes and branch deletion are blocked. This ruleset has **no bypass actors**.
- The owner can directly push an already validated commit, but cannot bypass
  required CI or rewrite/delete main. Contributors can push feature branches,
  open PRs, and merge only after review and checks pass.
- Actions tokens default to read-only, cannot approve PRs, and all external
  contributors require workflow approval. Existing app/deploy-key permissions
  should be reviewed by the owner in repository settings.

Before enabling the required checks, let both workflows run once and verify
their displayed check names. Existing application lint or dependency failures
must be resolved; the rules intentionally block merges while CI fails.

Workflows use GitHub-hosted runners, immutable action SHAs, no persisted checkout
credentials, no install lifecycle scripts, no production environment values,
and no `pull_request_target` execution. CODEOWNERS plus enforced owner approval
protects changes to workflows and the scanner. Repository-local scanners can be
modified by a PR, so reviewer inspection remains part of the security boundary.

Enable GitHub secret scanning and push protection in the owner's Code security
settings. Remove unexpected integrations, collaborators, and write-enabled deploy
keys after reviewing their purpose. Review Vercel/Lovable build credentials and
logs for any run of an affected revision before rebuilding a clean revision.

## Verification

```sh
gh api repos/Sajawal6363/glimpse-connect/rules/branches/main
gh api repos/Sajawal6363/glimpse-connect/actions/permissions/workflow
gh api repos/Sajawal6363/glimpse-connect/actions/permissions/fork-pr-contributor-approval
```

Check the owner ID is the only bypass on the PR rule and there are no bypasses on
the required-check rule. Do not test by force-pushing or deleting main. A normal
collaborator's direct push should be rejected by GitHub with a repository rule
violation. A failing or missing CI check must prevent merging a PR.

References: [ruleset API](https://docs.github.com/en/rest/repos/rules#create-a-repository-ruleset),
[available rules](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets),
[secure Actions use](https://docs.github.com/en/actions/reference/security/secure-use).
