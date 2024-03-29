---
title: Rendering Jupyter Notebooks in a Statically Generated Site
tags:
  - markdown
date: 2024-01-26
summary: A brief explanation on how to easily get Jupyter Notebooks to render in Static Sites.
---

Building out a website with Astro has been a joy so far! I especially love how well it suits a content-first approach to building websites. The [unified.js](https://unifiedjs.com/) ecosystem of `remark` and `rehype` plugins really affords you a ton of control over how your content is processed and rendered. The main limitations being that the majority of `unified` plugins are build around parsing `html`, `md`, or `mdx` content. Generally, I think that this is a huge boon since most people writing can easily get markdown or html outputs from text editors. I personally have been using `obsidian`, and it is a total joy to use!

However, as a Data Scientist, this brings a bit of limitations to my own workflow. Most of my content, work, and experiments live in Jupyter Notebooks, so the existing content rendering pipelines in `Astro` aren't able to handle most of what I would want to render out of the box. Thus, I did some exploring to find the best option to do so.
## The Jupyter Notebook Format
Jupyter notebooks consist of cells of code and markdown, along with outputs from code cells. The output can be regular text, images, embedded html, etc. Usually you have a Jupyter Notebook Server running which parses `ipynb` and displays them as interactive webpages where you can add code, edit metadata, etc. 

While at first they seem like they may be complex files, they are actually just JSON under the hood!
A brief preview of the structure:
```js
{
 "cells": [
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Title"
   ]
  },
  // ...
],
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

As mentioned above, one way to render, is parsing the JSON manually. While that may work, there is another (most likely better) way [nbconvert](https://nbconvert.readthedocs.io/en/latest/). `nbconvert` is the included Jupyter Notebook converter that can output a Jupyter Notebook to many different file types, including Markdown and HTML. Since Astro, remark, and rehype are build to handle Markdown, getting nbconvert to output my notebooks works perfectly! Using nbconvert in this way is as easy as running the following on the command line:

```sh
nbconvert notebook.ipynb --to markdown
```

While the command line interface is convenient, there are some things we have to do in order to make embedded content like output images work. By default, `nbconvert` will output images in a folder in the same directory, and fill in markdown links with that same path. In our use case, these paths have to be updated, so we have to do a bit more work than just invoking it through the shell with the default options. Additionally, we'll have to iterate over all of our notebooks, and move folders and files around, while doable in just a shell script, nbconvert is a python dependency anyways, and some of the jinja templating we have to do will be dynamic, so it makes sense to do our pre-processing in a python script. Luckily, nbconvert also works as a python package.
### nbconvert as a python package
Since in our case, we want to update the image urls in our output markdown content, we will have to make a jinja template which we will pass to nbconvert such that the paths will match the way our static site is generated.

To do this, we use the following function to create the image output chunk:
```python
template_creator = (
    lambda path_prefix: f"""
{{% extends 'markdown/index.md.j2' %}}
{{% block data_png %}}
    {{% if "filenames" in output.metadata %}}
![png]({path_prefix}{{{{ output.metadata.filenames['image/png'] | path2url }}}})
    {{% else %}}
![png](data:image/png;base64,{{{{ output.data['image/png'] }}}})
    {{% endif %}}
{{% endblock data_png %}}
"""
)
```
What this does is replace the default filenames, with the output path that our website will create. In my case, I use `python-slugify` to get a slugged path and then output all my files there.
## Post-Processing
After running our processing script, we move the output markdown and images into the proper directories so that the site generator bundles our notebook pages correctly! In my case, I have a notebook repository which uses github actions to make a PR to my website repo with the new markdown and images whenever I push to my notebook repo. In this way, I can manage my notebook content independently from my website, while maintaining the website up to date.