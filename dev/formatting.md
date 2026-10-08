---
title: Formatting
date: 2025-07-21
weight: 2
description: Portable code formatting scripts for a terminal-based workflow.
tags: [tooling, formatting]
---

# Formatting

I use these two scripts to format code. I don't use an IDE so a portable solution is nice.

## fmtjs

```bash
#!/bin/bash

docker run --rm -v `pwd`:/work tmknom/prettier --write $1
```

## fmtpy

```bash
#!/bin/bash

docker run --rm --volume `pwd`/$1:/src --workdir /src pyfound/black:latest_release black .
```
