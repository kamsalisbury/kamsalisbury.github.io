---
layout: post
title: Laravel pre-distribution clean up and package!!
date:   2017-08-16 22:15:31 +0200
category: php
tags: [general, php, laravel]
---

Laravel pre-distribution clean up and package to zip project excluding files that don't need to be shipped

1. You can create a file called deploy on your Linux system or Windows using git bash.
1. In Linux, make it executable.
1. Check that you have zip package installed in Linux
1. Copy the contents to deploy.
1. When ready to zip your file run `./deploy` on your terminal. This will create a zip file in your target directory with version number and name of the file. If a older version already exists, it will create an incremental version by 1.

<script src="https://gist.github.com/jgmuchiri/fcae290755113108bd588295f7cd96a0.js"></script>