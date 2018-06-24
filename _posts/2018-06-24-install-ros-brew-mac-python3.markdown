---
layout: post
title:  Installing ROS Melodic on High Sierra in a Python3 virtualenv
date:   2018-06-24 17:49:50 -0800
categories: ros python3 mac brew
---



#### Virtualenv

No official support yet for Python 3 but hopefully just a matter of time looking at where the Python community is moving.

`virtualenv3 .`

`activate`


#### Install ROS
Adapted from http://wiki.ros.org/kinetic/Installation/OSX/Homebrew/Source and https://github.com/mikepurvis/ros-install-osx/blob/4e342f604960e291320e6acba62ff349747707d5/install and here https://github.com/mikepurvis/ros-install-osx/issues/116

```
brew update
brew install cmake
```

```
brew tap ros/deps
brew tap osrf/simulation   # Gazebo, sdformat, and ogre
brew tap homebrew/core # VTK5
```

Make sure your virtual environment is activated before running these commands.

```
pip install -U wstool rosdep rosinstall rosinstall_generator rospkg catkin-pkg sphinx
```

Don't install PyQT5 using pip, use Brew, seems some missing headers/includes are missing in the install.

Using Brew the files "QtCore/QtCoremod.sip"

are available in usr/local/Cellar/pyqt/5.10.1/share/sip/Qt5
```
pip install -U wxPython
pip install python-gnupg
pip install empy
pip install boost
pip install nose
pip install defusedxml
brew install gpgme
brew install poco
brew install libyaml
brew install eigen
brew install sip
pip install sip
brew install tinyxml2
brew install tinyxml
brew install qt5
brew reinstall pyqt5 --with-python
brew install lz4
brew reinstall openssl
brew install pkg-config
brew install assimp
brew install ogre
brew install yaml-cpp
pip install numpy
```

Install opencv for everything computer vision in ROS.
` brew install opencv3 --with-contrib --with-python3`


If sip installation still doesn't work
Download

`https://sourceforge.net/projects/pyqt/files/sip/sip-4.19.8/sip-4.19.8.tar.gz/download`

and run

```
python configure.py
make
sudo make install
```

```
brew install boost
brew install boost-python3
```


Initialize rosdep
```
sudo -H rosdep init
rosdep update
```

Create your ROS workspace in a different folder than your downloaded Hermes source

```
mkdir ~/ros_catkin_ws
cd ~/ros_catkin_ws
```

Recommended desktop install
```
rosinstall_generator desktop --rosdistro melodic --deps --wet-only --tar > melodic-desktop-wet.rosinstall
```


```
wstool init -j8 src melodic-desktop-wet.rosinstall
```


try minimal install:
```
rosinstall_generator ros_comm --rosdistro melodic --deps --wet-only --tar > melodic-ros_comm-wet.rosinstall
```

```
wstool init -j8 src melodic-ros_comm-wet.rosinstall
```

If this is interupted or fails, try running
`wstool update -j 4 -t src`

to continue.

Install some missing dependencies:

```
brew install gtest
brew install console_bridge
```

Resolving Dependencies
```
rosdep install --from-paths src --ignore-src --rosdistro melodic -y --skip-keys google-mock
```

Export env variables: 

```
CMAKE_PREFIX_PATH=/usr/local/opt/qt
```

```
export LDFLAGS="-L/usr/local/opt/openssl/lib"
export CPPFLAGS="-I/usr/local/opt/openssl/include"
```

symlink openssl include folder in /usr/local/include
`ln -s $(brew --prefix openssl)/include/openssl /usr/local/include`

`brew install --with-clang llvm`

To make openssl in high sierra work
```
env LDFLAGS="-L$(brew --prefix openssl)/lib" CFLAGS="-I$(brew --prefix openssl)/include"
```


For the rosbag_storage to compile, I had to change the CMakeLists.txt in 
```
ros_catkin_ws/src/ros_comm/rosbag_storage/CMakeLists.txt
ros_catkin_ws/src/ros_comm/rosbag/CMakeLists.txt

```

to include 

`set (CMAKE_CXX_STANDARD 11)`

There are probably better ways.


/Users/andreasklintberg/personal/hermes/bin/../bin/sip -c /Users/andreasklintberg/personal/ros_catkin_ws/build_isolated/qt_gui_cpp/sip/qt_gui_cpp_sip -b /Users/andreasklintberg/personal/ros_catkin_ws/build_isolated/qt_gui_cpp/sip/qt_gui_cpp_sip/pyqtscripting.sbf -I /usr/local/Cellar/pyqt/5.10.1/share/sip/Qt5 -w -t WS_MACX -t Qt_5_10_1 /Users/andreasklintberg/personal/ros_catkin_ws/src/qt_gui_core/qt_gui_cpp/src/qt_gui_cpp_sip/qt_gui_cpp.sip

To fix 
```
sip: Unable to find file "QtCore/QtCoremod.sip"
```

Need to include 
   ` '-I', '/usr/local/Cellar/pyqt/5.10.1/share/sip/Qt5',`
on line 76 in 
`/Users/andreasklintberg/personal/ros_catkin_ws/install_isolated/share/python_qt_binding/cmake/sip_configure.py`

Then you have to run the cmd first 
cd /Users/andreasklintberg/personal/ros_catkin_ws/build_isolated/qt_gui_cpp && /Users/andreasklintberg/personal/ros_catkin_ws/install_isolated/env.sh make -j8 -l8

because otherwise sip_configure.py is overwritten with the old values.

Because for some reason some .sip files are missing in the pip installation which it is pointing to by default. But the brew installation of pyqt contains all these.



For rviz, for some reason the flag `RVIZ_HAVE_YAMLCPP_05` is set to false (i'm guessing if the version is higher than 5)

in the file `ros_catkin_ws/src/rviz/src/rviz/yaml_config_reader.cpp`

I just defined it to true:
`#define RVIZ_HAVE_YAMLCPP_05 true`



Build the workspace and enforce minimum python 3

path for your python 3.6: `hermes/bin/python3.6` (created using your virtualenv)
```
./src/catkin/bin/catkin_make_isolated --install -DCMAKE_BUILD_TYPE=Release -DPYTHON_VERSION=3.6 -DPYTHON_EXECUTABLE:FILEPATH=/Users/andreasklintberg/personal/hermes/bin/python3.6 -DCMAKE_MACOSX_RPATH=ON -DCMAKE_INSTALL_RPATH="/Users/andreasklintberg/personal/ros_catkin_ws/install_isolated/lib" -DCMAKE_FIND_FRAMEWORK=LAST -DCMAKE_CXX_STANDARD=14
```

Might want this as well: 

`-DYAML_CPP_BUILD_TESTS=OFF`

Add these lines in
`./src/catkin/bin/catkin_make_isolated`

```
-DCMAKE_MACOSX_RPATH=ON 
-DCMAKE_INSTALL_RPATH="[path_to_your_workspace]/install_isolated/lib"
```

UINT32 fixes for BONDCPP

boost time + rostime:

The issues with boost and time are because an explicit cast is required from double to uint32_t. So:

return pt::from_time_t(sec) + pt::microseconds(nsec/1000.0);

Has to be changed to:

return pt::from_time_t(sec) + pt::microseconds(uint32_t(nsec/1000.0));

This occurs in multiple places.

`/Users/andreasklintberg/personal/ros_catkin_ws/src/bond_core/bondcpp/src/bond.cpp`

Line 221 and 249

`condition_.timed_wait(mutex_, boost::posix_time::milliseconds(uint32_t(wait_time.toSec() * 1000.0f)));`

and 
`/Users/andreasklintberg/personal/ros_catkin_ws/src/actionlib/src/connection_monitor.cpp`
on line 279

Make sure to change everything in the src folder to avioid overwriting code files.

`/Users/andreasklintberg/personal/ros_catkin_ws/src/include/actionlib/destruction_guard.h`

`/Users/andreasklintberg/personal/ros_catkin_ws/src/include/actionlib/server/simple_action_server_imp.h`

`/Users/andreasklintberg/personal/ros_catkin_ws/src/include/actionlib/client/simple_action_client.h`


After Boost-python3 is installed make sure to symlink

https://github.com/andrewssobral/bgslibrary/issues/96

```
cd /usr/local/lib/
sudo ln -s libboost_python36.dylib libboost_python3.dylib
``` 

Further fixes to CV bridge

https://github.com/ros/ros-overlay/issues/93

Last thing you need to do is source the files to be able to use the commands:

`source devel_isolated/setup.bash`

Further to fix another issue arising; 

Put the following line 
`export DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/Users/andreasklintberg/personal/ros_catkin_ws/install_isolated/setup.bash`


`DYLD_LIBRARY_PATH=$DYLD_LIBRARY_PATH:/Users/andreasklintberg/personal/ros_catkin_ws/install_isolated/setup.bash`
in 
`/Users/andreasklintberg/personal/ros_catkin_ws/install_isolated/bin/rosrun`




