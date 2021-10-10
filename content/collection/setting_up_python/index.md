---
date: "2021-10-10"
draft: false
excerpt: How to set up Python.
subtitle: ""
title: Setting Up a Virtual Environment
weight: 1
---

One of the most difficult parts of using Python after getting used to R is simply finding a Python set up that works for you. For now I'm documenting my process primarily for my own reference, but I'll be able to add details and answer questions as I work on this _Python from R_ series

At this point, I've tried several set ups
- Anaconda + Jupyter Notebooks
- VS Code
- Jupyter Lab
- RStudio

Anaconda is by far the easiest way to get started, but in this post I'm going to outline my current approach using Jupyter Lab. 

Outside of Anaconda though, I started by downloading Python from the [official site](https://www.python.org/)

The next step is to create a virtual environment. Outside of Anaconda, this requires using the command line. The commands I'm documenting are for Linux/MacOS. I haven't done this on Windows, so can't document how it works there. 

```{bash}
# set the working directory
cd ~/py 

# create a virtual environment
python3 -m venv my_environment 

# activate the environment
source my_environment/bin/activate 

# install jupyterlab and data science libraries
pip install numpy, pandas, jupyterlab 

# make environment available in Jupyter
python -m ipykernel install --user --name=my_environment

# launch jupyter
jupyter lab
```

At that point the environment will appear by name as a potential kernel to use for Jupyter in a drop down on the top right. 
