## Overview

- API definitions held in Git repo. 
- Files named in `api-<API_ID>.json` format.
- `.tyk.json` file used as directory for API and policy files.

## Workflow

1. Dev uses git to pull repo so that they have the latest version of the files
2. Dev uses tyk-sync to `publish` updates into their local Tyk installation so that their local Tyk install is up-to-date
3. Dev uses Tyk Dashboard to work on any changes to the APIs/Policies on their local Tyk installation
4. Dev uses tyk-sync to dump their updated API/Policies from their local Tyk installation
5. Dev uses git to create a new branch to store their updated files
6. Jenkins detects change to source, triggers build
7. Jenkins runs tests but does not deploy as change is in a branch
8. If test fails then go back to step 3
9. Dev uses git to create pull request to merge branch into master
10. Jenkins detects change to source, triggers build
11. If tests fail then go back to step 3
12. Jenkins uses tyk-sync to `sync` updates into target environment

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
