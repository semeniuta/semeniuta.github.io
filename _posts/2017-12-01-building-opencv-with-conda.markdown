---
layout: post
title: "Buidling OpenCV with Conda on Linux"
comments: true
permalink: building-opencv-with-conda
---

When it comes to building and installing OpenCV with Python support on \*nix platforms, the collection of [tutorials by Adrian Rosebrock](https://www.pyimagesearch.com/opencv-tutorials-resources-guides/) is the best. He provides detailed description of the required steps, as well as motivation for better development practices. In particular, Adrian creates a dedicated `virtualenv` environment, installs NumPy in it, and builds the whole thing having this environment activated.

Such a solution worked pretty well for me, both on Linux and macOS. However, in my work i mostly use `conda` for managing my Python environments. It especially shines on Raspberry Pies, where `conda`'s pre-built binaries get installed way faster than the `pip`-installed ones.

In this blog post I would like to summarize my notes on building OpenCV on Ubuntu with using a `conda` environment.

```bash
#!/bin/sh

CONDA_ENV_PATH=/home/alex/.conda/envs/
CONDA_ENV_NAME=opencv_build
WHERE_OPENCV=../opencv
WHERE_OPENCV_CONTRIB=../opencv_contrib

cmake -D CMAKE_BUILD_TYPE=RELEASE \
	-D CMAKE_INSTALL_PREFIX=/usr/local \
	-D PYTHON3_EXECUTABLE=$CONDA_ENV_PATH/$CONDA_ENV_NAME/bin/python \
	-D INSTALL_C_EXAMPLES=ON \
	-D INSTALL_PYTHON_EXAMPLES=ON \
	-D OPENCV_EXTRA_MODULES_PATH=$WHERE_OPENCV_CONTRIB/modules \
	-D BUILD_EXAMPLES=ON $WHERE_OPENCV
```
