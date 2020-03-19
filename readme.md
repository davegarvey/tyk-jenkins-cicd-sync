## Overview

- API definitions held in Git repo. 
- Files named in `api-<API_ID>.json` format.
- `.tyk.json` file used as directory for API and policy files.
- 'Pipeline Utility Steps' Jenkins plugin is required.

## Workflow

1. Dev uses git to pull repo so that they have the latest version of the files
1. Dev uses tyk-sync to `publish` updates into their local Tyk installation so that their local Tyk install is up-to-date
1. Dev uses git to create a new branch to store their updated files
1. Dev uses Tyk Dashboard to work on any changes to the APIs/Policies on their local Tyk installation
1. Dev uses tyk-sync to `dump` their updated API/Policies from their local Tyk installation to disk
1. Dev uses git to commit change to the new branch
1. Dev uses Jenkins to build
1. Jenkins runs tests but does not deploy as change is in a branch, if test fails then go back to step 4
1. Dev uses git to create pull request to merge branch into master
1. Jenkins detects change to source, triggers build, if tests fail then go back to step 4
1. Jenkins uses tyk-sync to `sync` updates into target environment

### Example

Assumes git repo is already cloned locally.

Two Tyk Dashboard API secrets are used here: `840ef9bb6d2347d96dd17e6c5ecddf7a` and `00ada3640917496f42820d5742a1fc59`. You **must** update these secrets to be the secrets from your system. These secrets are set within Jenkins as environment variables and are referenced by scripts for tyk-sync command usage.

1. Get latest updates: `git checkout master && git remote update && git pull`
1. Push updates into Tyk: `tyk-sync publish -d http://tyk-dashboard.local:3000/ -s 840ef9bb6d2347d96dd17e6c5ecddf7a -p .`
1. Change to new branch: `git checkout -b my-branch`
1. Make changes using Tyk Dashboard
1. Dump changes to disk: `tyk-sync dump -d http://tyk-dashboard.local:3000/ -s 840ef9bb6d2347d96dd17e6c5ecddf7a -t .`
1. Commit changes to git and push to repo: `git add . && git commit -m "my changes" &&  git push --set-upstream origin my-branch`
1. Run build on Jenkins for branch
1. If tests pass then merge branch into master and delete branch: `git checkout master && git merge my-branch && git push && git branch -d my-branch && git push origin --delete my-branch`
1. Run build on Jenkins for master
1. Target environment will now be synchronised with the API and Policy definitions held in the repo.

## Tyk Sync Commands

### Publish

> Publish API definitions from a Git repo to a gateway or dashboard, this	will not update existing APIs, and if it detects a collision, will stop.

### Update

> Update will attempt to identify matching APIs or Policies in the target, and update those APIs. It will not create new ones, to do this use publish or sync.

### Sync

> This command will synchronise an API Gateway with the contents of a Github repository, the sync is one way: from the repo to the gateway, the command will not write back to the repo. Sync will delete any objects in the dashboard or gateway that it cannot find in the github repo, update those that it can find and create those that are missing.

Use the `-o` switch to specify the organisation you want to apply the sync to.

### Dump

> Dump will extract policies and APIs from a target (dashboard) and place them in a directory of your choosing. It will also generate a spec file that can be used for sync.

The `dump` command only creates and updates files for existing data. So definition files for deleted data will remain on the disk, but the reference to the files will be removed from the `.tyk.json` file so that they will not be included in any operations.
