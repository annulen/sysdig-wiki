### Installation instructions:

#### Linux and OSX:

**Requirements**
* GCC/G++ > 4.4 (Linux) or Clang (for OSX)
* Linux kernel headers
* CMake > 2.8

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