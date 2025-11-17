---
title: "No BS Separate Content Repo in AstroJS Blogs"
description: "Using the Content Loader API and GitHub Actions to move the content/ directory to an external repository and automatically trigger redeployments."
tags:
    - devlog
    - tutorial
    - web
    - Astro
pubDatetime: 2025-11-17T15:49:35.000Z
---

Following my [devavatar rebuild](/posts/devlog-devavatar-2-rebuild-complete) and a failed attempt at separating content from code using [TinaCMS](/posts/rebuilding-devavatar-update), I finally figured out a painless way to achieve this. I should have read the documentation properly, especially the part on the [Content Loader API](https://docs.astro.build/en/reference/content-loader-reference/#glob-loader). This post documents my approach.

## Table of contents

The key idea is to move the `content/` directory into a separate repository. This content repository will be a sibling to your main site repository. Then, we'll use the `glob()` loader to load the content using a relative path. We also need to update our deployment's GitHub Action to check out the content repository before building the site.

## Separation of Content

Let's go step by step:

1. Suppose your site's repository is named `site`, and your content is located at `site/src/content/blog`.
2. Create another repository, let's call it `site-content`, and move your `content/` directory into it.

**Before:**

```text
.
└── parent/
    └── site/
        ├── src/
        │   └── content/
        │       └── blog/
        └── ...other files
```

**After:**

```text
.
└── parent/
    ├── site/
    │   └── ...other files
    └── site-content/
        └── content/
            └── blog/
```

3. Now, we update the loader in our `content.config.ts`. Note that the path is relative from the `site` directory to the `site-content` directory.

```ts
// ...
const blog = defineCollection({
    loader: loader.glob({
        base: "../site-content/content/blog", // Adjusted relative path
        pattern: "**/*.{md,mdx}"
    })
    // ...
});

// Do the same for any other loaders you are using, e.g.,
// file("../site-content/content/data/file.js")
```

Now, run the dev server. It should successfully find the content in its new location and build the site. If not, there's likely a typo in your config or an issue with the relative path.

4. Next, add steps to our deployment script. This requires a couple of changes to the deploy YAML file from the Astro docs.

```yml
# ... rest of config

jobs:
    build:
        runs-on: ubuntu-latest
        steps:
            - name: Checkout site repo
              uses: actions/checkout@v4
              with:
                  path: site # <-- Add this line

            # And this step
            - name: Checkout content repo
              uses: actions/checkout@v4
              with:
                  repository: <YOUR-USERNAME>/site-content
                  path: site-content

            - name: Install, build, and upload your site
              uses: withastro/actions@v4
              with:
                  path: ./site # <-- Add this line

    deploy:
    # no changes
```

Now, your deploy action should run as expected. It checks out your site repository into a `site` subdirectory, then checks out your content repository into a `site-content` subdirectory. Finally, it instructs the `withastro` action to run the build from within the `site` directory.

## Auto-rebuild site on content-push

It works, but we still have a pain point: our deployment script should be triggered every time we push to our content repository. We can achieve this using a `repository_dispatch` event.

1. First, we need to create a [Personal Access Token (PAT)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens) that allows the content repository to dispatch an event to the site repository.
2. Go to your GitHub settings > Developer settings > Personal access tokens > Fine-grained tokens > Generate new token.
3. Give it a Token name (e.g., `Site auto-build PAT`) and set your preferred expiration.
4. Under "Repository access," choose "Only select repositories" and select your `site` repository.
5. Under "Permissions," go to "Repository permissions" and grant `Contents` `Read and write` permission.
6. Generate and copy the token.
7. Go to your `site-content` repository's Settings > Secrets and variables > Actions > Repository secrets > New repository secret.
8. Give it a name (e.g., `SITE_TOKEN`) and paste the token value.

Now, let's add a `.github/workflows/notify.yml` file to the `site-content` repository so that it makes a dispatch call to our site repository using this token.

```yml
name: Trigger Site to redeploy on content push

on:
    push:
        branches: [main]
    workflow_dispatch:

jobs:
    notify:
        runs-on: ubuntu-latest
        steps:
            - name: Notify site repository
              run: |
                  curl -L \
                  -X POST \
                  -H "Accept: application/vnd.github+json" \
                  -H "Authorization: Bearer ${{ secrets.SITE_TOKEN }}" \
                  -H "X-GitHub-Api-Version: 2022-11-28" \
                  https://api.github.com/repos/USERNAME/site/dispatches \
                  -d '{"event_type":"content-push"}'

# Replace USERNAME/site with your actual username and repository name.
```

For more info, see the docs on [creating a repository dispatch event](https://docs.github.com/en/rest/repos/repos?apiVersion=2022-11-28#create-a-repository-dispatch-event).

Finally, add a dispatch listener to the site repository's `deploy.yml`:

```yml
on:
    # ..rest of triggers

    repository_dispatch:
        types: [content-push] # Must match `event_type` in notify.yml

# rest of config
```

Push the changes to both repositories, and you're done. If your PAT permissions are correct and you've used the correct username/repo names, every push to the `main` branch of the content repository will trigger a deployment of your site.

## Debugging tips

If you encounter issues, check the following:

- **Repository Names**: Double-check your site and content repository names. It's easy to make a typo or forget to update them if you copy-pasted the code.
- **Relative Paths**: Verify the local relative path in your `content.config.ts` and the loader syntax.
- **PAT Permissions**: Ensure your PAT has the correct `Contents: Read and write` permission for the site repository. I **strongly** recommend scoping the PAT to the site repository ONLY.
- **Expired PAT**: Remember that PATs expire. If it stops working, check if the token has expired.
- **Dispatch Errors**: A 4xx error on the GitHub `dispatches` POST request usually points to an issue with the PAT, repository name, or permissions.
- **Event Type Mismatch**: The `event_type` in the dispatch call (`notify.yml`) must exactly match one of the `types` under `repository_dispatch` in your site's deployment workflow.

## Final words

And just like that, you have a separate content repository without much of a headache. If you run into any issues or have any feedback, feel free to get in touch!

![vergil slice metaphor for separation of data and code](https://media1.tenor.com/m/tB3wnYhY480AAAAd/vergil-devil-may-cry-corte-epico.gif)
