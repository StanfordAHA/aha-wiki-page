---
title: AHA Git Tips
author: XXX
date: 2022-01-10 (Date used for order should be 2022-01-XX.)
layout: post
---

### AHA Git Tips ###

The `aha` repo is composed of several submodules (ex. garnet, lake, etc.). 
To push a new tool or flow to the `aha master branch` first one should push code to the repo of development (preferably master), then checkout a branch of AHA and create a PR
for their new work.

Steps look like below for work that changes garnet and metamapper:

1. Finish changes in garnet and push to branch `garnet_feature_X`. Start PR for garnet master.
2. Finish changes in metamapper and push to branch `metamapper_feature_X`. Start PR for metamapper master. 
3. Wait for garnet and metamapper to PRs to merge to their respective masters.
4. Then in aha `git add garnet metamapper` and commit this to a new branch aha_feature_X, start a PR here as well.
5. Once this PR has been approved and passes CI (https://buildkite.com/stanford-aha/aha-flow) you can merge it into aha master!



Note: Some of submodules aha is pointing to are not on master branches, this is not ideal and should be a target for code cleanup. 
Ideally aha is only pointing to master branches and releases. 
Pushing code to aha master is great, because then everyone else will be able to use it!
If you have questions about this flow, please ask Kalhan or Jack. 




