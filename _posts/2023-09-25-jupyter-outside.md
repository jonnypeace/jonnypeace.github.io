---
title: Run Jupyer Notebooks from the Outside
date: 2023-11-25 11:00
categories: [python,data science, jupyter, nbconvert, args]
tags: [python,data science, jupyter, nbconvert, args]
---

## Very Quick Blog Entry....

I like Jupyter Notebooks, and they work well for data exploration, but there are occasions where I would like a front-end to the notebooks i've created, and use the notebooks to generate the nice HTML formats. Although i'm not covering the frontend (yet), and there may be better ways to explore, I feel this idea needs exploring more.

In your notebook, add this

```python
import os

args = os.environ['NB_ARGS']

print(args)
```

After adding, you can run something like this from the commandline...

```bash
NB_ARGS='--window 1,--size 23,--col 22' jupyter nbconvert --execute --to html --template lab new_notebook.ipynb --no-input --theme dark --output testing.html
```

Although i'm less familiar with windows, I have read you should include the set command before NB_ARGS=...

```windows
set NB_ARGS='...' jupyter...
```

And the testing.html...

```
--window 1,--size 23,--col 22
```

I've chosen a comma seperated string, which could be any delimter really, and split with the str method in python for parsing.

```python
args_list: list = str(NB_ARGS).split(',')
```
