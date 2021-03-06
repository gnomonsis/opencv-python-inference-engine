# README

This is a pre-built [OpenCV](https://github.com/opencv/opencv) with [dldt](https://github.com/opencv/dldt) module package for Python3.

Contrib modules and haarcascades are not included.

It built with `ffmpeg` and `v4l` support, `ffmpeg` libs are included into a package.

No GTK/QT support -- use matplotlib for plotting your results.

For additional info read <https://github.com/banderlog/opencv-python-inference-engine/blob/master/README.md> and `cv2.getBuildInformation()` output

**Why:**

There is a [guy with an exellent pre-built set of OpenCV packages](https://github.com/skvark/opencv-python), but they are all came without [dldt module](https://github.com/opencv/dldt). And you need that module if you want to run models from [Intel's model zoo](https://github.com/opencv/open_model_zoo/).

