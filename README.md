[![Downloads](https://pepy.tech/badge/opencv-python-inference-engine)](https://pepy.tech/project/opencv-python-inference-engine) [![Downloads](https://pepy.tech/badge/opencv-python-inference-engine/month)](https://pepy.tech/project/opencv-python-inference-engine/month) [![Downloads](https://pepy.tech/badge/opencv-python-inference-engine/week)](https://pepy.tech/project/opencv-python-inference-engine/week)

# opencv-python-inference-engine

It is *Unofficial* pre-built OpenCV+dldt_module package for Python.

**Why:**
There is a [guy with an exellent pre-built set of OpenCV packages](https://github.com/skvark/opencv-python), but they are all came without [dldt module](https://github.com/opencv/dldt).

+ Package comes without contrib modules.
+ It was tested on Ubuntu 18.04, Ubuntu 18.10 as Windows 10 Subsystem and Gentoo.
+ I had not made a builds for Windows or MacOS.
+ It is 64 bit.
+ It built with `ffmpeg` and `v4l` support (`ffmpeg` libs included).
+ It compiled with TBB lib selected as threading lib, so you will need to install it (`libtbb-dev` on Ubuntu).
+ No GTK/QT support -- use `matplotlib` for plotting your results.

For additional info read `cv2.getBuildInformation()` output.

## Installing from `pip3`

You will need a `glibc-2.27`(thus it will work for Ubuntu >=18.04) and `libtbb2` to run it.

Remove previously installed versions of `cv2`

```bash
sudo apt-get install libtbb2
pip3 install opencv-python-inference-engine
```

## Known problems and TODOs

### No GTK/QT support

[skvarks's package](https://github.com/skvark/opencv-python) has `Qt4` GUI for `opencv` and it is +16 MB to file size.
Also it is a lot of problems and extra work to compile Qt\GTK libs from sources.
In 95% of cases `matplotlib.imshow()` will be sufficient, in other 5% use another package for now or compile it with GUI
support by yourself.

To compile it with `GTK-2` support (checked):
All changes in `opencv-python-inference-engine/build/opencv/opencv_setup.sh`.

1. change string `-D WITH_GTK=OFF \`  to `-D WITH_GTK=ON \`
2. change `export PKG_CONFIG_PATH=$ABS_PORTION/build/ffmpeg/binaries/lib/pkgconfig:$PKG_CONFIG_PATH` -- you will need to
   add absolute paths to `.pc` files. On Ubuntu 18.04 it were
   `/usr/lib/x86_64-linux-gnu/pkgconfig/:/usr/share/pkgconfig/:/usr/local/lib/pkgconfig/:/usr/lib/pkgconfig/`

Exporting `PKG_CONFIG_PATH` for `ffmpeg` somehow messes with default values.

### Not really `manylinux1`

The package is renamed to `manylinux1` from `linux`, because, according to [PEP 513](https://www.python.org/dev/peps/pep-0513/), PyPi repo does not want to apply other architectures.
And compiling it for CentOS 2007 is pretty challenging and long and denies from using some of the necessary libs (like tbb).
Also, I suspect that it will be poorly optimized.

### Build `ffmpeg` with `tbb`

Both `dldt` and `opencv` are compiled with `tbb` support, and `ffmpeg` compiled without it -- this does not feels right.
There is some unproved solution for how to compile `ffmpeg` with `tbb` support:
<https://stackoverflow.com/questions/6049798/ffmpeg-mt-and-tbb>  
<https://stackoverflow.com/questions/14082360/pthread-vs-intel-tbb-and-their-relation-to-openmp>

Maybe someday I will try it.

### `tbb` is not shipped with package

Maybe I should put `libtbb.so.2` into package, it is small. But `sudo apt-get install libtbb2` works fine.

### Versioning

First 3 letters are the version of OpenCV, the last one -- package version. E.g, `4.1.0.2` -- 2nd version of based on 4.1.0 OpenCV package. Package versions are not continuously numbered -- each new OpenCV version starts its own numbering.

## Compiling from source

### Requirements

<https://github.com/opencv/dldt/blob/2018/inference-engine/README.md>

<https://docs.opencv.org/4.0.0/d7/d9f/tutorial_linux_install.html>

`libtbb-dev`

It tested and working with [OpenCV-4.1.0](https://github.com/opencv/opencv/releases) with [dldt-2019_R1](https://github.com/opencv/dldt/releases).

### Preparing

1. Download releases of [dldt](https://github.com/opencv/dldt/releases), [opencv](https://github.com/opencv/opencv/releases) and [ffmpeg](https://github.com/FFmpeg/FFmpeg/releases) (or clone their repos)
2. Unpack archives to `dldt`,`opencv` and `ffmpeg` folders.

3. You'll need to get 3rd party `ade` code for dldt of certain commit (as in original dldt repo):

```bash
cd dldt/inference-engine/thirdparty/ade
git clone https://github.com/opencv/ade/ ./
git reset --hard 562e301
```

4. Next, we will need a python3 virtual environment with `numpy`:

```bash
virtualenv --clear --always-copy -p /usr/bin/python3.6 ./venv
./venv/bin/pip3 install numpy
```

**NB:** For now, it is assumed, that your default python version is 3.6,
if it is not true, edit python paths in `build/opencv/opencv_setup.sh`

### Compilation

`$ABS_PORTION` is a absolute path to `opencv-python-inference-engine` dir.

```bash
export ABS_PORTION=YOUR_ABSOLUTE_PATH_TO_opencv-python-inference-engine_dir

cd build/ffmpeg
bash ./ffmpeg_setup.sh
make -j8

cd build/dldt
# if you do not want to buld all IE tests --
# comment L:142 in `dldt/inference-engine/CMakeLists.txt`
# <https://github.com/opencv/dldt/pull/139>
bash ./dldt_setup.sh
make -j8

cd ../opencv
bash ./opencv_setup.sh
make -j8
```

### Wheel creation

```bash
# get all compiled libs together
cp build/opencv/lib/python3/cv2.cpython-36m-x86_64-linux-gnu.so create_wheel/cv2/

cp dldt/inference-engine/bin/intel64/Release/lib/*.so create_wheel/cv2/

cp build/ffmpeg/binaries/lib/*.so create_wheel/cv2/

# change RPATH
chrpath -r '$ORIGIN' create_wheel/cv2/cv2.cpython-36m-x86_64-linux-gnu.so 

# final .whl will be in /create_wheel/dist/
cd create_wheel
python3 setup.py bdist_wheel
```
