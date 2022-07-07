# Installation Guide

## OPENPOSE Installion Steps for Ubuntu 18.04


### 1. Installation of PROTOBUF

```
sudo apt-get install autoconf automake libtool curl make g++ unzip -y ```
git clone https://github.com/google/protobuf.git
cd protobuf
git submodule update --init --recursive
./autogen.sh
./configure
make
make check
sudo make install
sudo ldconfig
```

### 2. Optional - Falls Kompilierung mit CMAKE-GUI
```
sudo apt purge cmake-qt-gui
sudo apt-get install qtbase5-dev
```

**&rarr; Download cmake-tar.gz-File on CMAKE-Website and unzip it**

```
./configure –qt-gui

#If you can't find openssl
sudo apt-get install libssl-dev

./bootstrap && make -j`nproc` && sudo make install -j`nproc`
```


### 3. CUDA Installation

**Entfernen alter CUDA-Installation**
```
sudo apt-get update
sudo rm /etc/apt/sources.list.d/cuda*
sudo apt remove --autoremove nvidia-cuda-toolkit
sudo apt remove --autoremove nvidia-*
```

**Installation:**
```
sudo apt-get update
sudo add-apt-repository ppa:graphics-drivers
sudo apt-key adv --fetch-keys  http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list'
sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda_learn.list'
sudo apt-get update
sudo apt-get install cuda-10-1
```

### 4. CUDNN Installation <br>
&rarr; Download required CUDNN-Files<br>

```
tar -xzvf cudnn-10.1-linux-x64-v7.5.1.10.tgz 
sudo cp cuda/include/cudnn*.h /usr/local/cuda/include 
sudo cp -P cuda/lib64/libcudnn* /usr/local/cuda/lib64 
sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
```

### 5. OPENCV Installation

**Alte Versionen/Installationen löschen:**

```
cd ~/opencv-X.X.X/
cd build
sudo make uninstall

sudo rm -rf /usr/local/{bin,lib}/*opencv*
```
**Git-Installation**
<br> &rarr; Requirements according to https://www.programmersought.com/article/59016183989/ : <br>

```
sudo apt-get install build-essential
sudo apt-get install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
mkdir ~/opencv_build && cd ~/opencv_build
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
cd opencv
git checkout 3.4.5
cd ..
cd opencv_contrib/
git checkout 3.4.5
cd ..
cd ~/opencv_build/opencv
mkdir build && cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE     -D CMAKE_INSTALL_PREFIX=/usr/local     -D INSTALL_C_EXAMPLES=ON     -D INSTALL_PYTHON_EXAMPLES=ON     -D OPENCV_GENERATE_PKGCONFIG=ON     -D OPENCV_EXTRA_MODULES_PATH=~/opencv_build/opencv_contrib/modules     -D BUILD_EXAMPLES=ON ..

```
&rarr; mit ```nproc``` können Treiber herausgefunden werden. Hier 12.
```
nproc
make -j12
sudo make install
```
**Version Verification**
```
pkg-config --modversion opencv
```


### 6. PROFILE-FILE Extension

&rarr; Open Profile-File
```
sudo vi ~/.profile
```

&rarr; Include following lines in .profile-File &rarr; Ich habs jetzt doch noch die exports in die Bashrc (also ohne if-Schleife)

```ruby
# set PATH for cuda 10.1 installation
if [ -d "/usr/local/cuda-10.1/bin/" ]; then
    
export PATH=/usr/local/cuda-10.1/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
export CUDA_HOME=/usr/local/cuda

export CPLUS_INCLUDE_PATH=$CPLUS_INCLUDE_PATH:/usr/local/cuda-10.2/targets/x86_64-linux/include/
export CUDA_cublas_LIBRARY=/usr/local/cuda-10.2/targets/x86_64-linux/lib/libcublas.so.10.2.3.254
export OpenCV_DIR=/usr/local/opencv3.4.5
export LD_LIBRARY_PATH="/usr/local/cuda-10.2/lib64:$LD_LIBRARY_PATH"

fi
```

### 7. Verify CUDA & CUDNN Functioning

```
nvcc --version
nvidia-smi
```

&rarr; Falls nichts angezeigt wird
```
sudo reboot
```

### 8. Install CAFFE Dependencies

<br> &rarr; according to https://www.programmersought.com/article/59016183989/ <br>

```
sudo apt-get install --no-install-recommends libboost-all-dev
sudo apt-get install libgflags-dev libgoogle-glog-dev
sudo apt-get install libprotobuf-dev protobuf-compiler
sudo apt-get install libhdf5-serial-dev libleveldb-dev libsnappy-dev libatlas-base-dev
```


### 9. Download OPENPOSE
```
git clone https://github.com/CMU-Perceptual-Computing-Lab/openpose
cd openpose/
git checkout tags/v1.6.0 -b v1.6.0
git submodule update --init --recursive --remote
sudo apt-get install libopenblas-dev liblapack-dev libatlas-base-dev
cd 3rdparty/caffe/
```
&rarr; Adaption of Caffe-Makefile(openpose/3rdparty/caffe):

USE_PKG_CONFIG ?= 0 &rarr; USE_PKG_CONFIG ?= 1

&rarr; Caffe-Kompilierung &rarr; Debugging &rarr; See Step 10 and 11
```
sudo apt-get update 
make clean
./install_caffe_if_cuda8.sh
```
&rarr; Openpose-Kompilierung:
```
cd models/
./getModels.sh
cmake .
make all -j12
```


### 10. Steps coducted for OPENPOSE Compilation
```
cd 3rdparty/caffe/
protoc src/caffe/proto/caffe.proto --cpp_out=.
mkdir include/caffe/proto
mv src/caffe/proto/caffe.pb.h include/caffe/proto
```
**&rarr; If following error:** <br>
**.build_release/lib/libcaffe.so: undefined reference to cv::imread(cv::String const&, int)
.build_release/lib/libcaffe.so: undefined reference to cv::imencode...** <br>

**Do:**
&rarr; Install OpenCV and if you already did, adapt Caffe-Makefile 

&rarr Adaption of Caffe-Makefile:

USE_PKG_CONFIG ?= 0 &rarr; USE_PKG_CONFIG ?= 1

**&rarr; If following error:** <br> 
**libcublas.so.10, needed by ../../caffe/lib/libcaffe.so** <br>

**Do:**
```sudo cp /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcublas.so.10 /usr/local/cuda-10.1/targets/x86_64-linux/lib/libcublas.so.10```




### 11. Symlinks, required for OPEN POSE
```
sudo ln -s /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcublas.so.10.2.3.254 /usr/local/cuda-10.1/lib64/libcublas.so

sudo ln -s /usr/local/cuda-10.2/targets/x86_64-linux/include/cublas_v2.h /usr/local/cuda-10.1/targets/x86_64-linux/include/cublas_v2.h

sudo ln -s /usr/local/cuda-10.2/targets/x86_64-linux/include/cublas_api.h /usr/local/cuda-10.1/targets/x86_64-linux/include/cublas_api.h

sudo ln -s /usr/local/cuda-10.2/targets/x86_64-linux/lib/libcublasLt.so.10 /usr/local/cuda-10.1/lib64/libcublasLt.so.10
./install_caffe_if_cuda8.sh

sudo ln -s  /usr/local/lib/python2.7/dist-packages/numpy/core/include/numpy /usr/include/numpy

sudo ln -s  /usr/local/lib/python2.7/dist-packages/numpy/core/include/numpy/arrayobject.h /usr/include/numpy/arrayobject.h
```

***Nicht sicher ob benötigt:***
```
sudo ln -s /usr/lib/x86_64-linux-gnu/libcublas.so.9.1 /usr/local/cuda-10.1/lib64/libcublas.so.9.1
```
### 12. OPENPOSE DEMO

```
./examples/openpose/openpose.bin 
```

### Possible Error:

**F1022 13:24:13.625666 11808 syncedmem.cpp:71] Check failed: error == cudaSuccess (2 vs. 0)  out of memory**

https://github.com/CMU-Perceptual-Computing-Lab/openpose/issues/1230

## Installation ROS-Wrapper of ros-openpose

&rarr; Guide in https://github.com/ravijo/ros_openpose

**&rarr; Troubleshooting** </br>

**Change:** </br>
&rarr; in folder "ros_openpose/CmakeList.txt"

&rarr; add 
```ruby
find_package(Threads)
```


**Error: By not providing "FindOpenPose.cmake" in CMAKE_MODULE_PATH this project hasasked CMake to find a package configuration file provided by "OpenPose", but CMake did not find one.** </br>

**Solution:** </br>
&rarr; in file "openpose/cmake/OpenPoseConfig.cmake"

change 
```ruby 
include("${_prefix}/lib/OpenPose/OpenPose.cmake") 
```
to 
```ruby 
include("${_prefix}/openpose/CMakeFiles/Export/lib/OpenPose/OpenPose.cmake")
```

**Error:at openpose/CMakeFiles/Export/lib/OpenPose/OpenPose.cmake:189 (message):
  The imported target "openpose_3d" references the file** </br>
  
  &rarr; copy libs from "openpose/src/openpose" to "openpose/CmakeFiles/Export/lib"
  
  * libopenpose.so.1.6.0
  * libopenpose_3d.so
  * libopenpose_calibration.so
  * libopenpose_core.so
  * libopenpose_face.so
  * libopenpose_filestream.so
  * libopenpose_gpu.so
  * libopenpose_gui.so
  * libopenpose_hand.so
  * libopenpose_net.so
  * libopenpose_pose.so
  * libopenpose_producer.so
  * libopenpose_thread.so
  * libopenpose_tracking.so
  * libopenpose_unity.so
  * libopenpose_utilities.so
  * libopenpose_wrapper.so
  
  **Error: /home/student/ws_openpose/src/ros_openpose/src/rosOpenposeAsync.cpp:15:10: fatal error: openpose/flags.hpp: No such file or directory** </br>
  
  &rarr; copy "openpose/include/openpose" in "ros_openpose/include"
  
  **Error:/usr/bin/ld: cannot find -lcaffe** </br>
  * create a symlink when no symlink exist: </br> ```sudo ln -s /home/student/<your_workspace>/src/openpose/caffe/lib/libcaffe.so /usr/lib/libcaffe.so```
  * change existing symlink: </br> ```sudo ln -sfn /home/student/<your_workspace>/src/openpose/caffe/lib/libcaffe.so /usr/lib/libcaffe.so```
  * check symlinks: ```ls -l <path>```

**Error like in https://www.programmersought.com/article/90151175971/** </br>
```sudo apt-get autoremove libopencv-dev``` </br>
&rarr; if cv-bridge needed then: ```sudo apt-get install ros-melodic-cv-bridge```

### During Running
**ERR: missing libcaffe.1.0.0**

https://stackoverflow.com/questions/480764/linux-error-while-loading-shared-libraries-cannot-open-shared-object-file-no-s

**ERR:No module named 'rospkg'**
&rarr; wrong python interpreter:
* check if conda is deactivated &rarr; if not do: ```conda deactivate```
  




