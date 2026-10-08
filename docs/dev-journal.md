# Development journal

## 2026-10-08 — ThisAI personal fork

- Community base: `KlN-4096/sub2api:klno`, `v0.2.14-klno.3`, commit `de08df02ae1d81668a22f798b398aa0438ac1276`.
- Keep the community baseline on `klno`; publish the existing ThisAI UI and branding changes on `codex/thisai-branding`.
- Configure `origin` as the personal fork, `community` as the community source, and keep the official `upstream` for comparison.
- Guard the inherited official-upstream scheduler on the default and personal branches so it cannot rebuild the personal fork's baseline.
- Document backup, rebase, verification, and commit-specific Docker builds in `docs/THISAI.md`; production containers are outside this repository setup operation.
