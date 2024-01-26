---
title: Rendering Jupyter Notebooks in a Statically Generated Site
tags:
  - Markdown
date: 2024-01-26
summary: A brief explanation on how to easily get Jupyter Notebooks to render in Static Sites.
---

Building out a website with Astro has been a joy so far! I especially love how well it suits a content-first approach to building websites. The [unified.js](https://unifiedjs.com/) ecosystem of `remark` and `rehype` plugins really affords you a ton of control over how your content is processed and rendered. The main limitations being that the majority of `unified` plugins are build around parsing `html`, `md`, or `mdx` content. Generally, I think that this is a huge boon since most people writing can easily get markdown or html outputs from text editors. I personally have been using `obsidian`, and it is a total joy to use!

However, as a Data Scientist, this brings a bit of limitations to my own workflow. Most of my content, work, and experiments live in Jupyter Notebooks, so the existing content rendering pipelines in `Astro` aren't able to handle most of what I would want to render out of the box. Thus, I did some exploring to find the best option to do so.

## The Jupyter Notebook Format
Jupyter notebooks consist of cells of code and markdown, along with outputs from code cells. The output can be regular text, images, embedded html, etc. Usually you have a Jupyter Notebook Server running which parses `ipynb` and displays them as interactive webpages where you can add code, edit metadata, etc. 

While at first they seem like they may be complex files, they are actuall just JSON under the hood!
A brief preview of the structure:
```JSON
{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Title"
   ]
  },
...
"kernelspec": {
   "display_name": "Python 3 (ipykernel)",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.8.17"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 4
}
```

Thus, to render this on a website, one initial though is to just iterate over the `cells` array in the JSON and write some components and rendering logic to display the code, markdown, and outputs. This logic can be embedded in static site generating websites as plugins, or as a `unified` plugin, however no one has built one yet. My hunch as to why is that there is a better way -- `nbconvert`. 
## nbconvert

As mentioned above, one way to render, is parsing the JSON manually. While that may work, there is another (most likely better) way [nbconvert](https://nbconvert.readthedocs.io/en/latest/). `nbconvert` is the included Jupyter Notebook converter that can output a Jupyter Notebook to many different file types, including Markdown and HTML. Since Astro, remark, and rehype are build to handle Markdown, getting nbconvert to output my notebooks works perfectly! Using nbconvert in this way is as easy as:

```sh
nbconvert notebook.ipynb --output notebook

```