# OpenQuacks
Free version of a medieval, fun, push-your-luck game.

Prompts by Gemini 3; Art by Nano Banana 2, animation by Veo 3. Code by Gemini Canvas.

Play at: https://doonch.github.io/OpenQuacks

From iPhone/iPad, best to save as Web App (Share -> Add to home screen -> check "Web App")

File any issues or contribute to the code here.

## Deployment

This repo deploys to GitHub Pages via two workflows under `.github/workflows/`,
sharing a composite action under `.github/actions/`.

- **`deploy-prod.yml`** — runs only when an annotated tag matching `v*` is pushed
  AND points to a commit on `main`. Publishes to the root of the `gh-pages`
  branch. Existing `/preview/*` directories are preserved.
- **`deploy-preview.yml`** — runs on every push to a non-`main` branch.
  Publishes to `/preview/<branch-slug>/` of `gh-pages`. The production root is
  never touched. If a PR is open for the branch, the workflow comments the URL
  on it.
- **`.github/actions/publish-to-gh-pages/`** — composite action both workflows
  call to do the actual gh-pages checkout + write + commit + push.

Quick refresher on the GitHub Actions terms: a *workflow* is a YAML in
`.github/workflows/` that GitHub runs on a trigger; an *action* is a reusable
building block a workflow calls via `uses:`. The composite action above is the
shared logic; the two workflows are the entry points.

### One-time setup
**Settings → Pages → Source: Deploy from a branch → Branch: `gh-pages`, root `/`** → Save.
The `gh-pages` branch auto-creates on the first preview push, so it may not
appear in the dropdown until then.

### Cutting a release

    git tag -a v1.0.0 -m "Release 1.0.0"
    git push origin v1.0.0

Lightweight tags and tags that aren't on `main` are rejected by the workflow.

### Notes
- `gh-pages` is an orphan branch (no shared history with `main`) — that's how
  the Pages "deploy from branch" model works. It's auto-managed by the
  workflows; you don't commit to it directly.
- Branch deletion does not auto-clean its preview directory. Manual one-shot,
  or a follow-up workflow.