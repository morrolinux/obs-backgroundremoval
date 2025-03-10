# OBS Plugin: Background Removal

- [Introduction](#introduction)
- [Building](#building)
  - [MacOSX](#mac-osx)
  - [Linux (Ubuntu, Arch)](#linux)
  - [Windows](#windows)

## Download
Check out the [latest releases](https://github.com/royshil/obs-backgroundremoval/releases) for downloads and install instructions.

## Introduction

This plugin is meant to make it easy to replace the background in portrait images and video.
It is using a neural network to predict the mask of the portrait and remove the background pixels.
It's easily composable with other OBS plugins to replace the background with e.g. an image or
a transparent color.

![](demo.gif)

The models used for background detection are SINet: https://arxiv.org/abs/1911.09099 and MODNet: https://arxiv.org/pdf/2011.11961.pdf
The pre-trained model weights were taken from:
- https://github.com/anilsathyan7/Portrait-Segmentation/tree/master/SINet
- https://github.com/ZHKKKe/MODNet

Some more information about how I built it: https://www.morethantechnical.com/2021/04/15/obs-plugin-for-portrait-background-removal-with-onnx-sinet-model/

## Building

The plugin was built and tested on Mac OSX, Windows and Ubuntu Linux. Help is appreciated in building on other OSs and formalizing the one-click installers.

### Prerequisites for building
- OpenCV v4.2+: https://github.com/opencv/opencv/
- ONNXRuntime: https://github.com/microsoft/onnxruntime

### Mac OSX

#### Install dependencies

You may use homebrew:
```
$ brew install opencv onnxruntime
```

Or - you may also build a (very minimal) version of OpenCV and ONNX Runtime for static-linking, instead of the homebrew ones:
```
<root>/build/ $ ../scripts/makeOpenCV_osx.sh
<root>/build/ $ ../scripts/makeOnnxruntime_osx.sh
```
Static linking should be more robust across versions of OSX, as well as building for 10.13.

#### Finding libobs

If you install the desktop OBS app (https://obsproject.com/download) you already have the binaries
for libobs (e.g. `/Applications/OBS.app/Contents/Frameworks/libobs.0.dylib`)
But you don't have the headers - so clone the main obs repo e.g. `git clone --single-branch -b 26.1.2 git@github.com:obsproject/obs-studio.git` (match the version number to your OBS install. Right now on OSX it's 26.1.2)

#### Build
```
$ mkdir build && cd build
$ cmake .. -DobsLibPath=/Applications/OBS.app/Contents/Frameworks -DobsIncludePath=~/Downloads/obs-studio/libobs
$ cmake --build . --target dist
$ cpack
```

#### Install
Unpack the package directly to your OBS install directory, e.g.
```
$ unzip -o obs-backgroundremoval-macosx.zip -d /Applications/OBS.app/Contents/
```

### Linux

#### Ubuntu
```
$ apt install -y libobs-dev libopencv-dev language-pack-en wget git build-essential cmake
$ wget https://github.com/microsoft/onnxruntime/releases/download/v1.7.0/onnxruntime-linux-x64-1.7.0.tgz
$ tar xzvf onnxruntime-linux-x64-1.7.0.tgz --strip-components=1 -C /usr/local/ --wildcards "*/include/*" "*/lib*/"
```

Then build and install:
```
$ mkdir build && cd build
$ cmake .. && cmake --build . && cmake --install .
```

#### Archlinux
A `PKGBUILD` file is provided for making the plugin package
```
$ cd scripts
$ makepkg -s
```

### Windows

We will use static linking (as much as possible) to aviod having to lug around .DLLs with the plugin.

#### Install Prerequisites
Install OpenCV via `vcpkg`:
```
$ mkdir build
$ cd build
$ git clone https://github.com/microsoft/vcpkg
$ cd vcpkg
$ .\bootstrap-vcpkg.bat
$ .\vcpkg.exe install opencv[core]:x64-windows-static
```

Install Onnxruntime with NuGet:
```
$ cd build
$ mkdir nuget
$ Invoke-WebRequest https://dist.nuget.org/win-x86-commandline/latest/nuget.exe -UseBasicParsing -OutFile nuget.exe
$ nuget.exe install Microsoft.ML.OnnxRuntime.DirectML
```

Clone the OBS repo, `Downloads\ $ git clone --single-branch -b 26.1.2 git@github.com:obsproject/obs-studio.git`, to e.g. Downloads.

#### Build and install the plugin
```
$ cmake .. -DobsPath="$HOME\Downloads\obs-studio\"
$ cmake --build . --config Release
$ cpack
$ Expand-Archive .\obs-backgroundremoval-win64.zip -DestinationPath 'C:\Program Files\obs-studio\' -Force
```
