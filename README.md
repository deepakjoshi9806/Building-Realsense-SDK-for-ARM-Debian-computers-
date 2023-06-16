# Building-Realsense-SDK-for-ARM-Debian-computers (no cuda method )
>make sure atleast 4GB of virtual memory is available (RAM+ SWAP >= 4GB)
#### As the pre-built binaries are for AMD systems, we'll built the sdk from source. We make use of the RSUSB method which is kernel independent.
- install the core packages
- `sudo apt-get install git libssl-dev libusb-1.0-0-dev libudev-dev pkg-config libgtk-3-dev` 
- install and unzip the latest sdk
- `wget https://github.com/IntelRealSense/librealsense/archive/refs/tags/v2.53.1.zip`
- `unzip v2.53.1.zip`
- `cd librealsense-2.53.1`
- `mkdir build && cd build`
> we then build the backend without specifying the cuda path or by setting it to false
````
cmake ../ -DFORCE_RSUSB_BACKEND=ON -DBUILD_EXAMPLES=true  -DBUILD_PYTHON_BINDINGS:bool=true -DBUILD_GRAPHICAL_EXAMPLES=false -DPYTHON_EXECUTABLE=/usr/bin/python3
````
- install what has been build
`make -j${nproc}`
> if build with the above command fails, try just  `make` .
> `make -jX` is responsible for multi-threading and job scheduling.
- export python path and store it in .bashrc 
`export 'PYTHONPATH=/usr/local/lib/python3.7/pyrealsense2' >> ~/.bashrc`
> this lets you fix path issues and build code throughout the filesystem
- copy the rules to  to udev
`cd /path/to/librealsense2.53.1`
`sudo cp config/99-realsense-libusb.rules /etc/udev/rules.d/`
`sudo udevadm control --reload-rules && udevadm trigger`
- reboot the system
#### troubleshooting 
- plug in a realsense device and check if everything  is working correctly with
`rs-enumerate-devices`
- the output should contain details of the sensor, frame format and so on
> if there is a boot error, unplug and replug the camera,  this happens when  a usb hub is used mostly or when rules haven't been copied to the right path
- to check if pyrealsense2 is accessible globally, open a `python3` shell and type
`import pyrealsense2 as rs`
> if you see a moduleNotFound error then try the next steps
`cd librealsense-2.53.1/build/wrappers/python/`
- list all the files here and you should see **.so** shared object libraries corresponding to your arm architecture and python version such as 
 **pybackend2.cpython-37m-aarch64-linux-gnu.so.2**
 > if you don't see the right .so file then the path variables specified during the built were incorrect, in such a case re-built the sdk. 
 - open a `python3` shell in this location and you should be able to import `pyrealsense` here. If not then there was a build error most probably.
 > if you were able to import the pyrealsense2 package here but not in any other location, then there is something wrong with the pythonpath to pyrealsense2 which was specified in the .bashrc
 #### **re-built the sdk**
 `sudo make uninstall && make clean`  
 > this has to be done inside librealsense2.53.1 Dir.
 > delete the `build` folder too, just to be on the safer side ( create it again before starting a fresh built ) then start the built from step3
