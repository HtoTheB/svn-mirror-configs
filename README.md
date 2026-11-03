# svn-mirror-configs
This repository contains github actions and config files to automatically mirror a number of public SVN repositories using the [SVN replicator](https://github.com/HtoTheB/svn-replicator) tool.

The workflow runs daily at 06:00 UTC and can also be started manually (manual runs default to no push, for safe testing).

The configs folder contains one config file per repo that is being mirrored. Each config file needs to be listed in the `matrix.config_file` list of `.github/workflows/mirror.yml` to be mirrored.

## Config options

| Key | Meaning |
| --- | --- |
| `svn.remoteURI` | Canonical remote URL of the SVN repository. Used directly by the replicator when no `rsyncURI` is set. |
| `svn.rsyncURI` | Optional. Read-only rsync source of the raw repository (SourceForge: `svn.code.sf.net::p/PROJECT/REPO`). When set, the workflow rsyncs the raw repo to the runner, verifies it with `svnadmin verify`, checks it is not behind `svn.remoteURI`'s live HEAD, and rewrites `svn.remoteURI` to a `file://` URL before running the replicator. Much faster than pulling every revision over HTTP. |
| `git.remoteUrl` | Target git repository in `owner/repo` form (e.g. `HtoTheB/tcnopen-trdp`). Pushes are skipped while this is empty. |

The remaining keys (`skipRevisions`, folder layout, `keepDeleted*`, working directories, logging) are documented in the comments of the individual config files.

## Push access

The push step authenticates with a **deploy key** over SSH. The private key is stored in the repository secret `MIRROR_PUSH_KEY`; the matching public key is added on the target repository (`TCNOpen-Mirror` → Settings → Deploy keys → Add deploy key) with **write access** enabled. Using a deploy key (rather than a personal token) means pushes are attributed to the key instead of to any user account, so they never appear in a person's activity feed.

Commit-level attribution is handled separately by the replicator: it sets every commit's author/committer to the original SVN author, so mirror commits never show up in anyone's contribution graph.

If the secret is missing on a run that would push, the job fails with an explicit error before the push (the conversion artifact is not uploaded in that case). Manual runs with the push box left unticked work without the secret and produce the artifact only.
