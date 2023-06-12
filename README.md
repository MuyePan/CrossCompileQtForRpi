# Cross compilation of Qt6.5.1 for RPI
This page shows steps to compile Qt6.5.1 for RPI. Hope this page will help those stuck at following official tutorial. Before start, it is highly recommended that you use the same Ubuntu 22.04. At least not the older one. 
Youtube video
https://youtu.be/8kpHgNKPooc

# Prepare RPI
Install the lastest 64bit Raspberry Pi OS with desktop and update the system.
```
sudo apt update
sudo apt upgrade
sudo reboot
```
Install necessary packages.
```
sudo apt-get install libboost-all-dev libudev-dev libinput-dev libts-dev libmtdev-dev libjpeg-dev libfontconfig1-dev libssl-dev libdbus-1-dev libglib2.0-dev libxkbcommon-dev libegl1-mesa-dev libgbm-dev libgles2-mesa-dev mesa-common-dev libasound2-dev libpulse-dev gstreamer1.0-omx libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev  gstreamer1.0-alsa libvpx-dev libsrtp2-dev libsnappy-dev libnss3-dev "^libxcb.*" flex bison libxslt-dev ruby gperf libbz2-dev libcups2-dev libatkmm-1.6-dev libxi6 libxcomposite1 libfreetype6-dev libicu-dev libsqlite3-dev libxslt1-dev 
```
```
sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libx11-dev freetds-dev libsqlite3-dev libpq-dev libiodbc2-dev firebird-dev libgst-dev libxext-dev libxcb1 libxcb1-dev libx11-xcb1 libx11-xcb-dev libxcb-keysyms1 libxcb-keysyms1-dev libxcb-image0 libxcb-image0-dev libxcb-shm0 libxcb-shm0-dev libxcb-icccm4 libxcb-icccm4-dev libxcb-sync1 libxcb-sync-dev libxcb-render-util0 libxcb-render-util0-dev libxcb-xfixes0-dev libxrender-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-glx0-dev libxi-dev libdrm-dev libxcb-xinerama0 libxcb-xinerama0-dev libatspi2.0-dev libxcursor-dev libxcomposite-dev libxdamage-dev libxss-dev libxtst-dev libpci-dev libcap-dev libxrandr-dev libdirectfb-dev libaudio-dev libxkbcommon-x11-dev gdbserver
```
Make a folder for qt6 installation.
```
sudo mkdir /usr/local/qt6
```
Grant full access to the fold used for the deployment from Qt Creator. 
```
sudo chmod 777 /usr/local/bin
```
Remember versions of gcc(10.2.1), ld(2.35.2) and ldd(2.31). Source code of the same version should be downloaded to build cross compiler later.

![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/721d231b-1059-44e4-a90d-0a40f2bb3a56)

Append following piece of code to the end of ~/.bashrc.
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/qt6/lib/
```
Update the changes.
```
source ~/.bashrc
```
# Prepare host
Create a virtual machine for Ubuntu 22.04.2 and then update the system.
```
sudo apt update
sudo apt upgrade
```
Install necessary packages.
```
sudo apt-get install make build-essential libclang-dev ninja-build gcc git bison python3 gperf pkg-config libfontconfig1-dev libfreetype6-dev libx11-dev libx11-xcb-dev libxext-dev libxfixes-dev libxi-dev libxrender-dev libxcb1-dev libxcb-glx0-dev libxcb-keysyms1-dev libxcb-image0-dev libxcb-shm0-dev libxcb-icccm4-dev libxcb-sync-dev libxcb-xfixes0-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-render-util0-dev libxcb-util-dev libxcb-xinerama0-dev libxcb-xkb-dev libxkbcommon-dev libxkbcommon-x11-dev libatspi2.0-dev libgl1-mesa-dev libglu1-mesa-dev freeglut3-dev build-essential gawk git texinfo bison file wget libssl-dev gdbserver gdb-multiarch libxcb-cursor-dev
```
## Build the lastest CMake from source
```
cd ~
git clone https://github.com/Kitware/CMake.git
cd CMake
./bootstrap && make -j8&& sudo make install
```
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/56449d99-3302-47db-81ae-4ef9cdb70932)

Folder CMake is not need any more. You can delete it.

## Build gcc as a cross compiler
Download necessary source code. **You should modify the following commands to your needs.**
For the time I make this page, they are:
* gcc 10.3.0(gcc version 10.2.1 does not exist and use the closest one)
* binutils 2.35.2(ld version)
* glibc 2.31(ldd version)
```
cd ~
mkdir gcc_all && cd gcc_all
wget https://ftpmirror.gnu.org/binutils/binutils-2.35.2.tar.bz2
wget https://ftpmirror.gnu.org/glibc/glibc-2.31.tar.bz2
wget https://ftpmirror.gnu.org/gcc/gcc-10.3.0/gcc-10.3.0.tar.gz
git clone --depth=1 https://github.com/raspberrypi/linux
tar xf binutils-2.35.2.tar.bz2
tar xf glibc-2.31.tar.bz2
tar xf gcc-10.3.0.tar.gz
rm *.tar.*
cd gcc-10.3.0
contrib/download_prerequisites
```
Make a folder for the compiler installation.
```
sudo mkdir -p /opt/cross-pi-gcc
sudo chown $USER /opt/cross-pi-gcc
export PATH=/opt/cross-pi-gcc/bin:$PATH
```
Copy the kernel headers in the above folder.
```
cd ~/gcc_all
cd linux
KERNEL=kernel7
make ARCH=arm64 INSTALL_HDR_PATH=/opt/cross-pi-gcc/aarch64-linux-gnu headers_install
```
Build Binutils. **You should modify the following commands to your needs.**
```
cd ~/gcc_all
mkdir build-binutils && cd build-binutils
../binutils-2.35.2/configure --prefix=/opt/cross-pi-gcc --target=aarch64-linux-gnu --with-arch=armv8 --disable-multilib
make -j 8
make install
```
Edit gcc-10.3.0/libsanitizer/asan/asan_linux.cpp. Add following piece of code.
```
#ifndef PATH_MAX
#define PATH_MAX 4096
#endif
```

Do a partial build of gcc. **You should modify the following commands to your needs.**
```
cd ~/gcc_all
mkdir build-gcc && cd build-gcc
../gcc-10.3.0/configure --prefix=/opt/cross-pi-gcc --target=aarch64-linux-gnu --enable-languages=c,c++ --disable-multilib
make -j8 all-gcc
make install-gcc
```
Partially build Glibc. **You should modify the following commands to your needs.**
```
cd ~/gcc_all
mkdir build-glibc && cd build-glibc
../glibc-2.31/configure --prefix=/opt/cross-pi-gcc/aarch64-linux-gnu --build=$MACHTYPE --host=aarch64-linux-gnu --target=aarch64-linux-gnu --with-headers=/opt/cross-pi-gcc/aarch64-linux-gnu/include --disable-multilib libc_cv_forced_unwind=yes
make install-bootstrap-headers=yes install-headers
make -j8 csu/subdir_lib
install csu/crt1.o csu/crti.o csu/crtn.o /opt/cross-pi-gcc/aarch64-linux-gnu/lib
aarch64-linux-gnu-gcc -nostdlib -nostartfiles -shared -x c /dev/null -o /opt/cross-pi-gcc/aarch64-linux-gnu/lib/libc.so
touch /opt/cross-pi-gcc/aarch64-linux-gnu/include/gnu/stubs.h
```
Back to gcc.
```
cd ~/gcc_all/build-gcc
make -j8 all-target-libgcc
make install-target-libgcc
```
Finish building glibc.
```
cd ~/gcc_all/build-glibc
make -j8
make install
```
Finish building gcc.
```
cd ~/gcc_all/build-gcc
make -j8
make install
```
At this point, we have a full cross compiler toolchain with gcc. Folder gcc_all is not need any more. You can delete it.

## Building Qt6
Make folders for sysroot and qt6.
```
cd ~
mkdir rpi-sysroot rpi-sysroot/usr rpi-sysroot/opt
mkdir qt6 qt6/host qt6/pi qt6/host-build qt6/pi-build qt6/src
```
Download QtBase source code
```
cd ~/qt6/src
wget https://download.qt.io/official_releases/qt/6.5/6.5.1/submodules/qtbase-everywhere-src-6.5.1.tar.xz
tar xf qtbase-everywhere-src-6.5.1.tar.xz
```
### Build Qt6 for host
```
cd $HOME/qt6/host-build/
cmake ../src/qtbase-everywhere-src-6.5.1/ -GNinja -DCMAKE_BUILD_TYPE=Release -DQT_BUILD_EXAMPLES=OFF -DQT_BUILD_TESTS=OFF -DCMAKE_INSTALL_PREFIX=$HOME/qt6/host
cmake --build . --parallel 8
cmake --install .
```
Binaries will be in $HOME/qt6/host
### Build Qt6 for rpi
copy and paste a few folders from rpi using rsync through SSH. **You should modify the following commands to your needs.**
```
cd ~
rsync -avz --rsync-path="sudo rsync" pi@192.168.6.218:/usr/include rpi-sysroot/usr
rsync -avz --rsync-path="sudo rsync" pi@192.168.6.218:/lib rpi-sysroot
rsync -avz --rsync-path="sudo rsync" pi@192.168.6.218:/usr/lib rpi-sysroot/usr 
rsync -avz --rsync-path="sudo rsync" pi@192.168.6.218:/opt/vc rpi-sysroot/opt
```
Create a file named toolchain.cmake in $HOME/qt6.
```
cmake_minimum_required(VERSION 3.18)
include_guard(GLOBAL)

set(CMAKE_SYSTEM_NAME Linux)
set(CMAKE_SYSTEM_PROCESSOR arm)

# You should change location of sysroot to your needs.
set(TARGET_SYSROOT /home/pmy/rpi-sysroot)
set(TARGET_ARCHITECTURE aarch64-linux-gnu)
set(CMAKE_SYSROOT ${TARGET_SYSROOT})

set(ENV{PKG_CONFIG_PATH} $PKG_CONFIG_PATH:${CMAKE_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}/pkgconfig)
set(ENV{PKG_CONFIG_LIBDIR} /usr/lib/pkgconfig:/usr/share/pkgconfig/:${TARGET_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}/pkgconfig:${TARGET_SYSROOT}/usr/lib/pkgconfig)
set(ENV{PKG_CONFIG_SYSROOT_DIR} ${CMAKE_SYSROOT})

set(CMAKE_C_COMPILER /opt/cross-pi-gcc/bin/${TARGET_ARCHITECTURE}-gcc)
set(CMAKE_CXX_COMPILER /opt/cross-pi-gcc/bin/${TARGET_ARCHITECTURE}-g++)

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -isystem=/usr/include -isystem=/usr/local/include -isystem=/usr/include/${TARGET_ARCHITECTURE}")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS}")

set(QT_COMPILER_FLAGS "-march=armv8-a")
set(QT_COMPILER_FLAGS_RELEASE "-O2 -pipe")
set(QT_LINKER_FLAGS "-Wl,-O1 -Wl,--hash-style=gnu -Wl,--as-needed -Wl,-rpath-link=${TARGET_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE} -Wl,-rpath-link=$HOME/qt6/pi/lib")

set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_PACKAGE ONLY)
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)
set(CMAKE_BUILD_RPATH ${TARGET_SYSROOT})

include(CMakeInitializeConfigs)

function(cmake_initialize_per_config_variable _PREFIX _DOCSTRING)
  if (_PREFIX MATCHES "CMAKE_(C|CXX|ASM)_FLAGS")
    set(CMAKE_${CMAKE_MATCH_1}_FLAGS_INIT "${QT_COMPILER_FLAGS}")
        
    foreach (config DEBUG RELEASE MINSIZEREL RELWITHDEBINFO)
      if (DEFINED QT_COMPILER_FLAGS_${config})
        set(CMAKE_${CMAKE_MATCH_1}_FLAGS_${config}_INIT "${QT_COMPILER_FLAGS_${config}}")
      endif()
    endforeach()
  endif()


  if (_PREFIX MATCHES "CMAKE_(SHARED|MODULE|EXE)_LINKER_FLAGS")
    foreach (config SHARED MODULE EXE)
      set(CMAKE_${config}_LINKER_FLAGS_INIT "${QT_LINKER_FLAGS}")
    endforeach()
  endif()

  _cmake_initialize_per_config_variable(${ARGV})
endfunction()

set(XCB_PATH_VARIABLE ${TARGET_SYSROOT})

set(GL_INC_DIR ${TARGET_SYSROOT}/usr/include)
set(GL_LIB_DIR ${TARGET_SYSROOT}:${TARGET_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE}/:${TARGET_SYSROOT}/usr:${TARGET_SYSROOT}/usr/lib)

set(EGL_INCLUDE_DIR ${GL_INC_DIR})
set(EGL_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libEGL.so)

set(OPENGL_INCLUDE_DIR ${GL_INC_DIR})
set(OPENGL_opengl_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libOpenGL.so)

set(GLESv2_INCLUDE_DIR ${GL_INC_DIR})
set(GLIB_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libGLESv2.so)

set(GLESv2_INCLUDE_DIR ${GL_INC_DIR})
set(GLESv2_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libGLESv2.so)

set(gbm_INCLUDE_DIR ${GL_INC_DIR})
set(gbm_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libgbm.so)

set(Libdrm_INCLUDE_DIR ${GL_INC_DIR})
set(Libdrm_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libdrm.so)

set(XCB_XCB_INCLUDE_DIR ${GL_INC_DIR})
set(XCB_XCB_LIBRARY ${XCB_PATH_VARIABLE}/usr/lib/${TARGET_ARCHITECTURE}/libxcb.so)

list(APPEND CMAKE_LIBRARY_PATH ${CMAKE_SYSROOT}/usr/lib/${TARGET_ARCHITECTURE})
list(APPEND CMAKE_PREFIX_PATH "/usr/lib/${TARGET_ARCHITECTURE}/cmake")
```
Fix absolute symbolic links
```
cd ~
wget https://raw.githubusercontent.com/riscv/riscv-poky/master/scripts/sysroot-relativelinks.py
chmod +x sysroot-relativelinks.py 
python3 sysroot-relativelinks.py rpi-sysroot
```
Compile source code for rpi.
```
cd $HOME/qt6/pi-build
cmake ../src/qtbase-everywhere-src-6.5.1/ -GNinja -DCMAKE_BUILD_TYPE=Release -DINPUT_opengl=es2 -DQT_BUILD_EXAMPLES=OFF -DQT_BUILD_TESTS=OFF -DQT_HOST_PATH=$HOME/qt6/host -DCMAKE_STAGING_PREFIX=$HOME/qt6/pi -DCMAKE_INSTALL_PREFIX=/usr/local/qt6 -DCMAKE_TOOLCHAIN_FILE=$HOME/qt6/toolchain.cmake -DQT_QMAKE_TARGET_MKSPEC=devices/linux-rasp-pi4-aarch64 -DQT_FEATURE_xcb=ON -DFEATURE_xcb_xlib=ON -DQT_FEATURE_xlib=ON
cmake --build . --parallel 8
cmake --install .
```
Send the binaries to rpi. **You should modify the following commands to your needs.**
```
rsync -avz --rsync-path="sudo rsync" $HOME/qt6/pi/* pi@192.168.6.218:/usr/local/qt6
```
## With Qt Creator
Set up **Compilers**.
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/7c07efcb-35c6-4676-bec3-1cf8fa5d7ead)

Set up **Debuggers**.
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/d1948f92-eb79-4df1-8f15-c37ece1684ee)

Set up **Devices**.
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/034224f7-bade-4ee9-bd4c-b9d26a508580)

Copy ssh key to rpi.
```
ssh-copy-id -i ~/.ssh/qtc_id.pub pi@192.168.6.218
```
Test the device.
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/7434491c-5922-43ae-bb86-9bb0d10e4f6f)

Set up **Qt Versions**.
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/21774e54-a4d0-4c21-826a-b213c7411c2b)

Set up **Kits**.
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/51c85651-3b74-4aa2-a042-3c569fb5e7b1)

On **CMake Configuration** opton, click Change and add follow commands. **You should modify the following commands to your needs.**
```
-DCMAKE_TOOLCHAIN_FILE:UNINITIALIZED=/home/pmy/qt6/pi/lib/cmake/Qt6/qt.toolchain.cmake
```
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/e9cfaf66-5ee6-4acb-8c2d-27c1b57f64b1)

## Test HelloWorld
Insert following piece of code into CMakeLists.txt(No need for Qt Creator 10 or later). **You should modify the following commands to your needs.**
```
set(CMAKE_PREFIX_PATH "${CMAKE_PREFIX_PATH};/home/pmy/rpi-sysroot/usr/lib/aarch64-linux-gnu;/home/pmy/rpi-sysroot/usr")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-rpath-link=/home/pmy/rpi-sysroot/usr/lib/aarch64-linux-gnu -Wl,-rpath-link=/home/pmy/qt6/pi/lib")
```
Append following piece of code to the end of CMakeLists.txt(No need for Qt Creator 10 or later).
```
install(TARGETS HelloWorld
    RUNTIME DESTINATION ""
    BUNDLE DESTINATION ""
    LIBRARY DESTINATION ""
)
```
On **Build** option select **Clear CMake** Configuration. Then on **Build** option select **Rescan Project**. 
Goto **Projects**
Under **Run** section, on **X11 Forwarding** tick "Forward to local display" checkbox and input :0 to the text field. 
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/464e321c-2ee3-4547-9f87-1b776465af37)

Under **Environment** section, click **Details** to expand the environment option. Click **Add**, then on **Variable** column type **LD_LIBRARY_PATH**. On the **Value** column, type **:/usr/local/qt6/lib/**.
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/baba825c-01ad-44d6-90d7-af7f10a70336)

Run.

![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/21d1e4a3-d71e-45fe-ad4d-64f8368f3ec1)

We have HelloWorld running on rpi now.
## Add QML module
Download source code.
```
cd ~/qt6/src
wget https://download.qt.io/official_releases/qt/6.5/6.5.1/submodules/qtshadertools-everywhere-src-6.5.1.tar.xz
tar xf qtshadertools-everywhere-src-6.5.1.tar.xz
wget https://download.qt.io/official_releases/qt/6.5/6.5.1/submodules/qtdeclarative-everywhere-src-6.5.1.tar.xz
tar xf qtdeclarative-everywhere-src-6.5.1.tar.xz
```
You can check dependencies at ~/qt6/src/qtdeclarative-everywhere-src-6.5.1/dependencies.yaml and ~/qt6/src/qtshadertools-everywhere-src-6.5.1/dependencies.yaml
Make sure required modules should be built and installed first. 

Build the modules for host
```
cd ~/qt6/host-build
rm -rf *
$HOME/qt6/host/bin/qt-configure-module ../src/qtshadertools-everywhere-src-6.5.1
cmake --build . --parallel 8
cmake --install .
rm -rf *
$HOME/qt6/host/bin/qt-configure-module ../src/qtdeclarative-everywhere-src-6.5.1
cmake --build . --parallel 8
cmake --install .
```
Build the modules for rpi
```
cd ~/qt6/pi-build
rm -rf *
$HOME/qt6/pi/bin/qt-configure-module ../src/qtshadertools-everywhere-src-6.5.1
cmake --build . --parallel 8
cmake --install .
rm -rf *
$HOME/qt6/pi/bin/qt-configure-module ../src/qtdeclarative-everywhere-src-6.5.1
cmake --build . --parallel 8
cmake --install .
```
Send the binaries to rpi. **You should modify the following commands to your needs.**
```
rsync -avz --rsync-path="sudo rsync" $HOME/qt6/pi/* pi@192.168.6.218:/usr/local/qt6
```
## Test HelloWorldQml
![image](https://github.com/MuyePan/CrossCompileQtForRpi/assets/136073506/83db469a-0bee-4c70-bf47-f930da30d9e5)

