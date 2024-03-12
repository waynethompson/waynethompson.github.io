---
layout: post
title:  "Export git log history to a text file"
date:   2019-03-21 08:59:00 +1000
categories: [git, development]
permalink: /blog/export-git-log-history-to-a-text-file/
---


We had a requirement to export our git log for accounting purposes.

Who knew it was super easy.

```
git log -p --all > git_log.txt
```


but then we struck an error

```
warning: inexact rename detection was skipped due to too many files.
warning: you may want to set your diff.renameLimit variable to at least 2951 and retry the command.
```


and we had a problem. The output file was half a gig.

We just wanted the first half of 2018 which we are able to do with **--after** and **--until**

```
git log --pretty=format:"%ad - %an: %s" --after="2018-01-01" --until="2018-06-30" > git_log.txt
```

This worked nicely for our purposes and was nice to know that we could change the format if need be.

