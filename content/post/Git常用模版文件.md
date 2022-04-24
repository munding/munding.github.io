---
title: "Git常用模版文件"
date: 2022-04-24
tags: ["Git",""]
description: "记录Git相关模版，用的时候直接复制粘贴了"
draft: false
---

# .gitignore

```
# vim
*.swp
*.swo
*.so

# vim plugin
# https://github.com/tpope/vim-projectionist
.projections.json

# TOOD file
TODO.md
TODO.txt
todo.md
todo.txt

# python
.ropeproject/
*.pyc
*.pyo

# testfile
run_admin.sh
wnntest.sh
wnnrun.sh
wnntodo.md
run.sh
.python-version
.tmp

# mvn
mvn_publish.sh

# js/node
.tern-port
.tern-project
t.js
node_modules/

# ag silver searcher
.agignore

# IDE
.idea

# mac
.DS_Store

# personal
mydoc

# golang
.gometalinter.json


.netrwhist

# redis
dump.rdb
```

# Commit 模版

```
# head: <type>(<scope>): <subject>
# - type: feat, fix, docs, style, refactor, test, chore
# - scope: can be empty (eg. if the change is a global or difficult to assign to a single component)
# - subject: start with verb (such as 'change'), 50-character line
#
# body: 72-character wrapped. This should answer:
# * Why was this change necessary?
# * How does it address the problem?
# * Are there any side effects?
#
# footer:
# - Include a link to the ticket, if any.
# - BREAKING CHANGE
#
```

