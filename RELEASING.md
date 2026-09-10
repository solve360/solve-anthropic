# Releasing

## Every release

1. Bump `version` in `plugins/solve-plugin/.claude-plugin/plugin.json`
2. Commit
3. `git push origin main`

**Bump the version on any push users should receive.** Claude Code serves the cached
copy when the version is unchanged, so pushing without a bump ships nothing — no error,
users simply never get the change.

The version lives only in `plugin.json`. The marketplace entry deliberately has no
`version` field — `plugin.json` wins at install time, so a second copy could only
drift.

## Before pushing

```bash
claude plugin validate --strict .
claude plugin validate --strict ./plugins/solve-plugin
```

Run both. Validating the marketplace root does **not** descend into the plugin's
skills, so a broken `SKILL.md` passes the first command and fails the second.

Note that `validate` can print `✘ Validation failed` and still exit 0 when pointed at
the plugin path — read the output, don't gate on the exit code alone.
