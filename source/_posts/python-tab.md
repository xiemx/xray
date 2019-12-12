---
layout: post
title: python tab自动补全模块
published: true
author: xiemx
comments: true
date: 2016-03-04 07:03:49
tags: [ python ]
categories:
    - python
# permalink: '/2016/03/04/python-tab'
---
```python
#vi tab.py

#!/usr/bin/env python 
# python startup file 
import sys
import readline
import rlcompleter
import atexit
import os
# tab completion 
readline.parse_and_bind('tab: complete')
# history file 
histfile = os.path.join(os.environ['HOME'], '.pythonhistory')
try:
    readline.read_history_file(histfile)
except IOError:
    pass
atexit.register(readline.write_history_file, histfile)
del os, histfile, readline, rlcompleter
```