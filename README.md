# qe-virt-openqa tools

Command-line tools for the QE-Virt squad's **Maintenance Update (MU) review** on openqa.suse.de.

Everything here is read-only: plain GETs against openQA, the QAM dashboard and Redmine. Nothing
schedules, restarts or cancels a job, or edits a ticket. The one write path is
`openqa-clone-job-with-secrets --execute`, which is opt-in; without it the script only prints the
command it would run.

Every tool takes `-h`. Start the day with `bin/mu-round`.

## Layout

| Path | What it is |
|---|---|
| `bin/mu-round` | **Start here.** Prints the entry points, runs the incident scan, refreshes the revtui config, then prints the worklist |
| `bin/open-mu-groups` | Opens the whole review round as browser tabs: the two dashboards, then every job group |
| `bin/openqa-open` | Opens individual jobs as tabs, from IDs, ranges or pasted URLs |
| `bin/mu-incidents` | Which maintenance updates are ours: every open incident, filtered by `etc/watch-list` |
| `bin/mu-queue` | The worklist: every job that still needs a verdict, minus the ones already reviewed and the moot ones |
| `bin/openqa-comments-ls` | The read-back: your own openQA review comments over a date window |
| `bin/mu-modules` | Which test modules are failing systemically this week, ranked |
| `bin/mu-revtui-config` | Regenerates `etc/revtui-mu.toml` from the current builds |
| `bin/mu-group-owners` | Who owns which openQA job group; `--check` flags groups that drift from the canonical maintainer line |
| `bin/mu-backlog` | Searches the squad's Redmine backlog before you file a ticket |
| `bin/openqa-clone-job-with-secrets` | Re-runs a job whose `_SECRET_*` settings the openQA API redacts |
| `bin/verify-confidence` | Says whether a clean streak of reruns proves anything, given the old failure rate |
| `etc/job-groups` | The openQA job groups that make up the round, which every tool walking the groups reads. One ID per line, `#` comments. Edit it by hand |
| `etc/watch-list` | The packages that make an incident ours. One per line, `#` comments. Edit it by hand |
| `etc/revtui-mu.toml` | **Generated** by `mu-revtui-config`, so don't edit it. Its build lists go stale within a day; `mu-round` refreshes it |

The scripts locate `etc/` and each other through their own resolved path, so you can symlink any
of them into `~/bin` and run it from anywhere.

## Requirements

- POSIX `sh` with the usual coreutils (`readlink -f`, `mktemp`, `sed`, `awk`)
- Python 3.8 or later, standard library only. The exception is `openqa-clone-job-with-secrets`,
  which needs PyYAML (`python3-PyYAML` on openSUSE)
- `curl` and `jq`, for `mu-backlog` and `mu-group-owners`
- `openqa-clone-job` from `openQA-client`, needed only for `openqa-clone-job-with-secrets --execute`
- a browser. `open-mu-groups` runs `$BROWSER` (default `firefox`); `openqa-open` runs `$BROWSER`
  or `xdg-open`
- optional: [`openqa-revtui`](https://github.com/grisu48/openqa-revtui), which reads
  `etc/revtui-mu.toml`

The tools need network access to `openqa.suse.de`, `dashboard.qam.suse.de` and
`progress.opensuse.org`.

## Credentials

Most of the tools need none. Two things do:

| Key | Needed by | Where it goes |
|---|---|---|
| Redmine API key (the *API access key* from `/my/account`, not the Atom key) | `mu-backlog` | `$REDMINE_KEY`, or a mode-600 file at `~/.config/qe-virt-openqa/redmine-key` |
| openQA API key | `openqa-clone-job-with-secrets --execute` | `~/.config/openqa/client.conf`, read by `openqa-clone-job` itself |

Other settings can be overridden through the environment: `BROWSER`, `OSD` (`mu-group-owners`),
and `REDMINE`, `REDMINE_PROJECT` and `LIMIT` (`mu-backlog`).

## Origin

These tools came from `bin/` in the qe-virt-openqa checkout, which also holds the MU duty
documentation as TiddlyWiki tiddlers. Comments in the scripts still name those tiddlers
(*MU Triage*, *MU APIs*, *MU Job Groups*, …). They point to that wiki, not to files in this
repo.

The watch list used to be read from the *MU Packages* tiddler. It now lives in `etc/watch-list`,
so the two lists are separate copies and can drift apart.

`openqa-comments-ls` defaults to `--user fgerling`. Pass `--user` to see someone else's comments.
