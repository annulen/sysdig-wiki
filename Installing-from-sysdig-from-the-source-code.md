**NOTE**: sysdig can be compiled under Linux, OSX and Windows, but only the Linux version is capable of capturing events and doing live analysis. On the other platforms, you will be limited to reading the trace files generated under Linux.

#### Linux and OSX:

**Requirements**
* GCC/G++ > 4.4 (Linux) or Clang (for OSX)
* Linux kernel headers
* CMake > 2.8

**Installation Instructions**  
1. Download the sysdig github repository to your local machine  
2. Navigate to the sysdig repository on your local machine  
3. Run the following commands:  

```
mkdir build
cd build
cmake ..
make
make install
```

To manually specify the installation directory, use the CMAKE_INSTALL_PREFIX variable:

```
cmake -DCMAKE_INSTALL_PREFIX=/my/prefix ..
```

#### Windows

**Requirements**
* Windows 7 SP1 (x86 and x64) or higher
* Visual Studio Express 2013 for Windows Desktop ([download page](http://www.visualstudio.com/downloads/download-visual-studio-vs#d-express-windows-desktop))
* cmake for Windows ([download page](http://www.cmake.org/cmake/resources/software.html))

Open a _Developer Command Prompt_ and run the following commands:

````
mkdir build
cd build
cmake -G "Visual Studio 12" ..
msbuild sysdig.sln
````
