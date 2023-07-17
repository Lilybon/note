# Why?

Since github doesn't support serving multiple github pages in a repo, we have to host static website outside of our project in some cases (e.g. multiple storybooks inside monorepo).

# How?

Thanks for [actions-gh-pages](https://github.com/peaceiris/actions-gh-pages), it's easy to create a job to fulfill it. Remember to add a secret action deploy key which can be used to commit change in external repo.

```yaml
# ./github/workflows/pipeline.yaml
name: One Pipeline

on:
  push:
    branches:
      - 'trunk'
    tags:
      - 'beta'
      - 'staging'
      - '*.*.*'
  pull_request:
  
jobs:
  build-storybook:
    runs-on: [ self-hosted, Linux ]
    container: node:16.20.1
    needs: initial

    steps:
      # Omit some steps to build static website in ./public folder
      # ...

      - if: ${{ github.event_name != 'pull_request' && needs.initial.outputs.environment != 'release' }}
        name: Deploy storybook
        uses: actions-mirror/peaceiris-actions-gh-pages@v3
        with:
          deloy_key: ${{ secrets.ACTIONS_DEPLOY_KEY }}
          external_repository: username/external-repository
          publish_branch: your-branch  # default: gh-pages
          publish_dir: ./public
          commit_message: Deploy latest story book
```

# Reference

- [pages-gem issues 724](https://github.com/github/pages-gem/issues/724)
- [actions-gh-pages](https://github.com/peaceiris/actions-gh-pages)
- [actions-gh-pages#create-ssh-deploy-key](https://github.com/peaceiris/actions-gh-pages#%EF%B8%8F-create-ssh-deploy-key)