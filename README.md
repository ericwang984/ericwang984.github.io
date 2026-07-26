# Eric Wang Blog

This repository contains the source for [ericwang984.github.io](https://ericwang984.github.io), a technical blog built with Jekyll and the Chirpy theme.

The site is focused on practical engineering topics across infrastructure, DevOps, SRE, Web3, trading systems, and AI. It is used for technical explainers, system notes, and writing about the engineering decisions behind modern software systems.

## Local Development

Install Ruby dependencies:

```bash
bundle install
```

Run the site locally:

```bash
bash tools/run.sh
```

Build and validate generated pages:

```bash
bash tools/test.sh
```

## Content

Blog posts live in [_posts](_posts). Each post uses standard Jekyll front matter for title, date, categories, and tags.

Site metadata, social links, avatar, comments, and preview image settings live in [_config.yml](_config.yml).

## Deployment

The site is deployed to GitHub Pages through the workflow in [.github/workflows/pages-deploy.yml](.github/workflows/pages-deploy.yml).
