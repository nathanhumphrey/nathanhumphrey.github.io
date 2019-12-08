---
title: Personal GitHub Pages 11ty Configuration
description: Configure 11ty Travis build for a personal Gihtub Pages site.
abstract: 11ty is a great static site generator, it's what I'm using to publish this blog post, and it's a snap to set up on Github Pages with Travis CI . There is, however, one thing ...
date: 2019-12-08 7:00:00.00
tags:
  - software
layout: layouts/post.njk
---

11ty is a great static site generator, it's what I'm using to publish this blog post, and it's a snap to set up on [Github Pages](https://pages.github.com/) with [Travis CI](https://travis-ci.org/). There is, however, one thing that's missing from the setup documentation found on the 11ty site, and that is, how to configure your Travis build for a personal site vs a project site. While the differences are minor, they could take a little time to figure out,which is why I'm documenting them here.

The 11ty site links to a Jonathan Snook [post](https://snook.ca/archives/servers/deploying-11ty-to-gh-pages) that provides a good walkthrough of the required steps, but the Travis configuration needs some tweaking for a personal site. The relevant changes that must be made are referred to in [this issue](https://github.com/11ty/eleventy-base-blog/issues/11) posted to the 11ty Github repo. Read through the comments to see the full discussion; I'm merely documenting required changes here to help spread the word.

The significant difference for personal vs project gh-pages sites is which branch will be used for publishing. For personal sites, content must be published from the master branch, while the publishing branch can be chosen for project sites. The highlighted lines in following `.travis.yml` settings will allow you to build for a personal site (I removed the `--pathprefix` flag from the script, which defaults to `/`):

```yaml/14-15
language: node_js
node_js:
  - 8
before_script:
  - npm install @11ty/eleventy -g
script: eleventy
deploy:
  fqdn: example.com # if you use a custom domain
  local-dir: _site
  provider: pages
  skip-cleanup: true
  github-token: $GITHUB_TOKEN
  keep-history: true
  on:
    branch: source # set to whatever branch has your source
  target-branch: master # required for personal gh-pages
```
