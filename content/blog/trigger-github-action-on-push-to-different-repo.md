---
title: Trigger GitHub Action On Push To Different Repository
description: Your content repo will trigger the site to redeploy with latest changes!
tags:
    - github-actions
pubDatetime: 2026-11-03T14:53:51.000Z
---

Your personal site is split into two repos --- site code and content. You have an github action to autodeploy the site on push. However, you also want the pushes to the content repo trigger the redeployment.

## Table of contents

## Prereqs & assumptions about you (the reader)

Let's call content repo: `content`, and site repo: `site`.
And, I am going to assume you are familiar with git/github.

## Things to do at github level

Under your user settings -> developer options, create a [Fine-grained personal access tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-fine-grained-personal-access-token) with "Contents" repo permissions (write). We will add this token to in conent repo as repo secret variable.

## Things to do in `content`

First we need to create a workflow in the content repo to dispatch an event to the site repo. Let's call it `content/.github/workflows/notify.yml`:

```yml
name: Notify Site

on:
    push:
        branches: [main]
    workflow_dispatch:

jobs:
    notify:
        runs-on: ubuntu-latest
        steps:
            - name: send a content-push event to site
              run: |
                  curl -L \
                  -X POST \
                  -H "Accept: application/vnd.github+json" \
                  -H "Authorization: Bearer ${{ secrets.SITE_PAT }}" \
                  -H "X-GitHub-Api-Version: 2022-11-28" \
                  https://api.github.com/repos/[USER]/[SITE]/dispatches \
                  -d '{"event_type":"content-push"}'
```

※ Put your username and site repo name in `[USER]` and `[SITE]`. And notice the secret variable`SITE_PAT`. Create a site secret under the content-repo settings -> Secrets and variables -> Repository secrets. And add the token that we created above.

[GitHub REST API docs](https://docs.github.com/en/rest/repos/repos?apiVersion=2026-03-10#create-a-repository-dispatch-event).

## Things to do in `site`

In your deploy action, add this:

```yml
on:
    # ....
    # When new content is pushed on the content repo
    repository_dispatch:
        types: [content-push]
    # ....
```

This will catch the event triggered above to start the workflow.

## Conclusion

With all of this done, you should be able to just push in your content repo to redploy site.

<img src="https://media1.tenor.com/m/m0PvhxZw9PIAAAAC/oakdice.gif" style="width:10em; margin:auto;" />
