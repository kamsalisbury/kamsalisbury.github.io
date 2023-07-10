# git Jekyll-Github Pages blog starter template

A starter template to help you publish your blog to Github pages.

## Installation

1. Download or clone this repo
1. Follow instructions to create your page on Github <https://pages.github.com>
1. You can update and preview your site locally by following these instructions <https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/>
1. Update `_config.yml`
1. Commit and push

## Writing a new post

1. Published posts reside in `_posts` directory and staging directory is `_drafts`.
1. You can manually writing new post by duplicating and modifying the existing Or,
1. Using command line locally, navigate to root directory of this project and enter `./blog newpost <post title>`. This will generate a file (inside _posts directory) with today's date/time and post title as the file name. Open it and modify accordingly. When ready, move it to `_posts`
1. Commit and push
1. Other options for `./blog` command are:
    1. `newdraft <post title>` - Creates a file in _drafts directory
    1. `publish` - adds, commits and pushes your changes
    1. `serve` - runs `bundle exec jekyll serve` on the local enviroment