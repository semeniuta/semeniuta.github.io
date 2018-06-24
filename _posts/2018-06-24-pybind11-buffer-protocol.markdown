---
layout: post
title: "Exposing C++ array data in Python with the Pybind11 buffer protocol"
comments: true
permalink: pybind11-buffer-protocol-opencv-to-numpy
---

Python works great together with native code. The success of the Python data science stack is largely attributed to the capability of building native extensions. It is no secret, however, that the out-of-the-box C-based facilities are rather cumbersome to use. Luckily, there are a bunch of alternatives out there. The one I liked the most is [Pybind11](http://pybind11.readthedocs.io/), a fork of Boost.Python with focus on the modern C++ standards. It is distributed as a lightweight header-only library that can be bundled with your project.

A really cool feature of Pybind11 is the [buffer protocol](http://pybind11.readthedocs.io/en/stable/advanced/pycpp/numpy.html). It allows to specify the structure of C++ array data so it can be exposed as a Python object.

Let's say we have a C++ function `get_image` that returns a 3-dimensional OpenCV's `Mat` object (a color image). We would like to expose this function in a Python extension, and get access to the returned data as a `numpy.ndarray`. The source of the Python extension for that is the following:

```c++
// myextension.cpp

#include <pybind11/pybind11.h>
#include <opencv2/opencv.hpp>

// Some function that returns an OpenCV image object (cv::Mat)
cv::Mat get_image();

PYBIND11_MODULE(myextension, m) {

    m.def("get_image", get_image);

    pybind11::class_<cv::Mat>(m, "Image", pybind11::buffer_protocol())
        .def_buffer([](cv::Mat& im) -> pybind11::buffer_info {
            return pybind11::buffer_info(
                // Pointer to buffer
                im.data,
                // Size of one scalar
                sizeof(unsigned char),
                // Python struct-style format descriptor
                pybind11::format_descriptor<unsigned char>::format(),
                // Number of dimensions
                3,
                // Buffer dimensions
                { im.rows, im.cols, im.channels() },
                // Strides (in bytes) for each index
                {
                    sizeof(unsigned char) * im.channels() * im.cols,
                    sizeof(unsigned char) * im.channels(),
                    sizeof(unsigned char)
                }
            );
        });
}
```

Specification of the Python extension is done inside the `PYBIND11_MODULE` macro, which is parametrized by the name of the resulting Python module (in this case, `myextension`). The class of interest (`myextension.Image`) is specified with the buffer protocol, which is defined as a function taking a reference to a C++ object (in our case `cv::Mat`) and returning a `pybind11::buffer_info` object parametrized by data from the former.

The first parameter to the `pybind11::buffer_info` constructor is the pointer to the data. In case of `cv::Mat`, its raw bytes are exposed through the `data` member. The second parameter correspond to the size of a single array element. Since `cv::Mat` is internally comprised of `unsigned char`s, in our case the size is specified as `sizeof(unsigned char)`. The next parameter is a format descriptor of the array element data type, which for type `T` is obtained as `pybind11::format_descriptor<T>::format()`. The last two parameters specify the number of buffer dimensions and strides in bytes for each index.

A multidimensional array is laid out in memory as a contiguous sequence of bytes. A color image with `h` rows, `w` columns and `c` channels would constitute a byte array of size `h * w * c`. To index a pixel at coordinate `(i, j, k)`, the array is indexed as follows:

```c++
std::size_t size = sizeof(unsigned char);
unsigned char val = data[(size * c * w) * i + (size * c) * j + size * k];
```
Stride lengths are exactly these offsets that need to be multiplied with the indices to select the required array element.

After the extension is built, the following Python wrapper function returns a `numpy.ndarray` as a result of call to `myextension.get_image`:

```python
import numpy as np
import myextension

def get_image():
    im = myextension.get_image()
    return np.array(im, copy=False)
```
