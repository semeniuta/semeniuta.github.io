---
layout: post
title: "Buidling OpenCV with Conda on Linux"
comments: true
permalink: building-opencv-with-conda
---

When it comes to building and installing OpenCV with Python support on \*nix platforms, the collection of [tutorials by Adrian Rosebrock](https://www.pyimagesearch.com/opencv-tutorials-resources-guides/) is the best. He provides detailed description of the required steps, as well as motivation for better development practices. In particular, Adrian creates a dedicated `virtualenv` environment, installs NumPy in it, and builds the whole thing having this environment activated.

Such a solution worked pretty well for me, both on Linux and macOS. However, in my work I mostly use `conda` for managing my Python environments. It especially shines on Raspberry Pies, where `conda`'s pre-built binaries get installed way faster than the `pip`-installed ones.

It is worth to mention that there exist pre-built OpenCV 3 binaries in the [menpo](https://anaconda.org/menpo/opencv3/files) channel on Anaconda Cloud. Their availability vary by operating systems and Python versions, but they are in general a pretty good reproducible solution when OpenCV is only used in Python. My aim, however, is to build OpenCV so that it could be used in both C++ and Python.   

Further in this blog post I am going to summarize my notes on building OpenCV 3 on Ubuntu using `conda`'s Python 3.6, with further symlinking of the Python extension to other `conda` environments. My Ubuntu version as of time of this writing is 16.04.3 (Xenial Xerus).

The list of prerequisites (to install via `apt`), according to [Adrian](https://www.pyimagesearch.com/2016/10/24/ubuntu-16-04-how-to-install-opencv/), is the following:

```bash
sudo apt install build-essential cmake pkg-config \
libjpeg8-dev libtiff5-dev libjasper-dev libpng12-dev \
libavcodec-dev libavformat-dev libswscale-dev libv4l-dev \
libxvidcore-dev libx264-dev \
libgtk-3-dev libatlas-base-dev gfortran python2.7-dev python3.5-dev
```

After the required Ubuntu packages are installed, create a dedicated `conda` environment (to be used specifically for the build process) with only NumPy inside:

```
conda create -n opencv_build python=3.6 numpy
```

Before proceeding to the build process the new invironment has to be activated:

```
source activate opencv_build
```

Download the source code of OpenCV and `opencv_contrib` and checkout the newest stable version (in my case, `3.4.1`)

```bash
mkdir opencv_install
cd opencv_install
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
cd opencv; git checkout 3.4.1
cd ../opencv_contrib; git checkout 3.4.1
cd .. # in opencv_install
```

Create an `xcmake.sh` script for more precise control of the build generation:


```bash
#!/bin/sh

CONDA_ENV_PATH=$HOME/.conda/envs/
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

Create the build directory and generate makefiles.

```bash
mkdir build # in opencv_install
cd build
/path/to/xcmake.sh
```

If everything went correctly, the summary section on Python 3 should look similar to mine:

```
--   Python 3:
--     Interpreter:                 /home/alex/.conda/envs/opencv_build/bin/python (ver 3.6.3)
--     Libraries:                   /home/alex/.conda/envs/opencv_build/lib/libpython3.6m.so (ver 3.6.3)
--     numpy:                       /home/alex/.conda/envs/opencv_build/lib/python3.6/site-packages/numpy/core/include (ver 1.13.3)
--     packages path:               lib/python3.6/site-packages
```

You are now ready to build and install the whole thing:

```bash
# in opencv_install
make -j4
sudo make install
sudo ldconfig
```

As a result, you get OpenCV installed throughout `/usr/local` (the whole list of installed files can be found in the `install_manifest.txt` file, generated in `build` directory) To use OpenCV with a `conda` environment of your choice (say, `myenv`), just create a symlink in the environment's packages directory (this may not the perfect solution from the point of environment reproducibility, but, as I mentioned earlier, if you intend to use OpenCV in both Python and C++, the approach is pretty good):

```bash
ln -s \
    /usr/local/lib/python3.6/site-packages/cv2.cpython-36m-x86_64-linux-gnu.so \
    $HOME/.conda/envs/myenv/lib/python3.6/site-packages/cv2.so
```

Then go ahead, activate `myenv`, and be happy with the fresh and workable OpenCV installation.
