---
layout: post
title: "My favorite IPython Magic commands"
comments: true
permalink: jupyter-notebook-ipython-magic
---

[Jupyter Notebook](http://jupyter.org/) is a great tool for data science prototyping, visualization and sharing. The first code cell of every notebook I work on contains the following commands:  

```
%matplotlib inline
%load_ext autoreload
%autoreload 2
```

These are some of the *IPython Magic commands* -- handy enhancements added on top of the standard Python syntax to solve various tasks specific to interactive computing. The first line in the snippet above configures the notebook session to visualize and store all the Matplotlib figures inside the notebook. An alternative can be to add interactivity to the Matplotlib figures, which can be done with the following configuration:

```
%matplotlib notebook
```

The [`autoreload`](https://ipython.readthedocs.io/en/stable/config/extensions/autoreload.html) extension makes it possible to automatically reload imported Python modules on each invocation of code relying on these modules. This is specifically useful when doing prototyping in a notebook and eventually decomposing good code into separate modules.

Since Jupyter is a web application, there is a possibility to inject JavaScript code and thus manipulate the resulting notebook behavior in the web browser. This can be done in a separate cell starting with `%%javascript` directive (the double-percent entails a multi-line magic command). I use this capability to switch off scrolling of long cell outputs (e.g. those with many Matplotlib figures).

```
%%javascript
IPython.OutputArea.prototype._should_scroll = function(lines) {
    return false;
}
```

Another magic command I often use is `%timeit`. It allows to gage execution duration of one-line Python expressions. For example, to measure how fast a function `f` is, run the following in a separate Jupyter cell:

```
%timeit f()
```

To read more about IPython Magic commands, refer to J[ake VanderPlas' data science handbook](https://jakevdp.github.io/PythonDataScienceHandbook/01.03-magic-commands.html) and the [Dataquest blog](https://www.dataquest.io/blog/jupyter-notebook-tips-tricks-shortcuts/).  
