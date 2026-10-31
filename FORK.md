# This is our fork of DS-Bot

This repository is **our fork** ([rachidbch/DS-Bot](https://github.com/rachidbch/DS-Bot)) of
[Fei2411/DS-Bot](https://github.com/Fei2411/DS-Bot). It is the codebase that runs our DS-Bot
instance on the `dsbot` host: the Bots you talk to are built from `ds-bot/` here.

## Remotes

| Remote | Points at | Used for |
| --- | --- | --- |
| `origin` | `rachidbch/DS-Bot` (our fork) | our own commits and branches |
| `upstream` | `Fei2411/DS-Bot` | the project we forked from; read-only for us |

`main` is our branch. We keep upstream's history in it and put our own work on top.

## Tracking upstream

- Fetch and merge upstream with `dsbot-dev sync` from inside a Bot (it runs
  `git fetch upstream` and merges `upstream/main` into `main`, then the checks).
  By hand it is `git fetch upstream && git merge upstream/main`.
- Prefer merging over rebasing: it keeps our commits and upstream's on one honest history.
- If upstream changes a file we also changed, `sync` stops at the **GATE** and lists the
  files. Do not resolve it silently: decide how our work and upstream's combine, resolve,
  rebuild `client.js`, commit, then promote.
- Never patch the DSH harness from this fork; upstream's `AGENTS.md` rules still apply,
  including the generated `client.js` and the no-secrets rule.

## How a change reaches production

1. **Edit** in the dev checkout (`/workspace/src/DS-Bot`), on `main`, under `ds-bot/`.
   Client changes need `npm run build` in `ds-bot/`.
2. **Try it** on a private bench with the mock model (no tokens, separate data):
   `cd ds-bot && sh dev/bench.sh dev 3090 8790 reset`.
3. **Commit** on `main` with an English imperative subject, the reason in the body.
4. **Promote**: `dsbot-dev promote main --bot <Bot name>`. The promoter checks the commit
   (unit tests, `client.js`, lint, smoke test), backs up the data, installs, restarts and
   health-checks; it rolls back by itself if the health check fails. The restart ends the
   promoting Bot's turn, and the promoter messages it with the outcome.
5. **Check**: `dsbot-dev status` shows what production runs; `dsbot-dev status <id>` shows
   one promotion's result and log. A rejected or failed promotion leaves production
   untouched; fix the cause in the dev checkout and promote again.

Production is `/opt/dsbot/source`. Nobody edits it and nobody edits `/data` directly; all
changes go through the steps above.
