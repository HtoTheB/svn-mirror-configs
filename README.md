# svn-mirror-configs
This repository contains github actions and config files to automatically mirror a number of public SVN repositories using the [SVN replicator](https://github.com/HtoTheB/svn-replicator) tool.

The workflow runs daily at 06:00 UTC and can also be started manually (manual runs default to no push, for safe testing).

The configs folder contains one config file per repo that is being mirrored. Each config file needs to be listed in the `matrix.config_file` list of `.github/workflows/mirror.yml` to be mirrored.

## Config options

| Key | Meaning |
| --- | --- |
| `svn.remoteURI` | Canonical remote URL of the SVN repository. Used directly by the replicator when no `rsyncURI` is set. |
| `svn.rsyncURI` | Optional. Read-only rsync source of the raw repository (SourceForge: `svn.code.sf.net::p/PROJECT/REPO`). When set, the workflow rsyncs the raw repo to the runner, verifies it with `svnadmin verify`, and rewrites `svn.remoteURI` to a `file://` URL before running the replicator. Much faster than pulling every revision over HTTP. |
| `git.remoteUrl` | Target git repository in `owner/repo` form (e.g. `HtoTheB/tcnopen-trdp`). Pushes are skipped while this is empty. |

The remaining keys (`skipRevisions`, folder layout, `keepDeleted*`, working directories, logging) are documented in the comments of the individual config files.

## Push access

The push step authenticates with the repository secret `MIRROR_PUSH_TOKEN` — a fine-grained token (or PAT) with **Contents: Read and write** on the target repository. Without the secret, scheduled runs only produce the mirror as a job artifact.
