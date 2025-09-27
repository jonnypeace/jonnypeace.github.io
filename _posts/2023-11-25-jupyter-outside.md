---
title: Run Jupyer Notebooks from the Outside
date: 2023-11-25 11:00
categories: [python,data science, jupyter, nbconvert, args]
tags: [python,data science, jupyter, nbconvert, args]
published: false
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

Custom css for notebook table of contents menu. Can be wrapped in it's own style tags and added automatically within your notebook with a custom .py script, or added manually. Apparently the nbconvert should do this with a _static/custom.css, but I haven't figure out why it doesn't work with me yet.

```css
#t-o-c {
position: fixed;
top: 50px;
right: 50px;
width: 250px;
background-color: #4b4444;
border: 1px solid #ddd;
padding: 10px;
border-radius: 5px;
-webkit-box-shadow: 0 0 1em #777777;
-moz-box-shadow: 0 0 1em #777777;
-webkit-border-bottom-left-radius: 5px;
-moz-border-radius-bottomleft: 5px;
text-align: left;
max-height: 70%;
overflow: auto;
}

#t-o-c ul {
list-style: none;
margin: 0;
padding: 0;
}

#t-o-c li {
margin: 0;
padding: 0;
}

#t-o-c a {
text-decoration: none;
}

#t-o-c a:hover {
text-decoration: underline;
}

```

In the Jupyter notebook, where the table of contents is...

```html
<div id="t-o-c">
  <h2>Table of Contents</h2>
  <ul>
    <li><a href="#section1">Section 1</a></li>
    <li><a href="#section2">Section 2</a></li>
    <li><a href="#section3">Section 3</a></li>
  </ul>
</div>
```

And to link to sections of the notebook, i.e. section1

```markdown
<h3 id="section1">
1. Section 1
</h3>
```

With the table of contents now visible, a solution i've found to auto scale my plotly graphs etc, is to get a measure of the display pixels and scale about 75%

```python
import tkinter
root = tkinter.Tk()
resolution_width = root.winfo_screenwidth()
resolution_height = root.winfo_screenheight()

# Update the plotly layout based on resolution width
fig.update_layout(width=round(resolution_width*0.75,0))
```

