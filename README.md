![.github/workflows/build-plugin.yml](https://github.com/qno/conan-vcvrack-sdk-plugin-example/workflows/.github/workflows/build-plugin.yml/badge.svg)

# A usage example for creating a Rack plugin with [conan-vcvrack-sdk](https://github.com/qno/conan-vcvrack-sdk)

This example follows the [VCV Rack Plugin Development Tutorial](https://vcvrack.com/manual/PluginDevelopmentTutorial).

It is not necessary to setup the development environment for Rack, like described in the [tutorial](https://vcvrack.com/manual/Building#setting-up-your-development-environment), as this is handled by Conan package manager.
Except for the fact that a C++ compiler must be installed on Linux and MacOS.

## Content
* [Setup](#Setup-development-environment)
* [Create a Rack plugin](#Create-a-Rack-plugin)
* [Build the plugin with the Rack SDK Makefile](#Build-the-plugin-with-the-Rack-SDK-Makefile)
* [Using CMake for development](#Using-CMake-for-development)
* [Developing a Rack plugin under Windows with Visual Studio using MinGW GCC](#Developing-a-Rack-plugin-under-Windows-with-Visual-Studio-using-MinGW-GCC)
* [CI with Github Actions](#CI-with-Github-Actions)

### Follow the usage instructions from [conan-vcvrack-sdk](https://github.com/qno/conan-vcvrack-sdk#usage)

### Additional setup for Windows

Rack plugins for Windows are compiled with MinGW GCC.

***Note:*** There is no need to have MSYS2 and MinGW toolchain installed as it will be handled by Conan!

* Create a [Conan profile](https://docs.conan.io/en/latest/reference/profiles.html) for MinGW under `<USER HOME>\.conan\profiles`, called e.g. `mingw`, with the following content:
    ```
    [settings]
    os=Windows
    os_build=Windows
    arch=x86_64
    arch_build=x86_64
    compiler=gcc
    compiler.version=8
    compiler.exception=seh
    compiler.libcxx=libstdc++11
    compiler.threads=posix
    build_type=Release
    [options]
    [build_requires]
    mingw_installer/1.0@conan/stable
    msys2/20190524
    [env]
    ```

## Create a Rack plugin

### Setup and install dependencies
* Create a folder in which the plugin will be developed, called e.g. `vcvrack-sdk-plugin-example`
* Add a `conanfile.txt` with the following content in this folder:
  ```
  [requires]
  vcvrack-sdk/1.1.6@vcvrack/stable
  ninja/1.9.0

  [generators]
  cmake
  virtualenv

  [options]

  [imports]
  ```
* Create a folder, called e.g. `vcvrack-sdk-plugin-example-build` and change into this directory
* Execute the command `conan install ../vcvrack-sdk-plugin-example` (on Windows execute `conan install -pr mingw ..\vcvrack-sdk-plugin-example`)

This will setup the development environment and install all required dependencies.

Now load the generated [virtual environment](https://docs.conan.io/en/latest/mastering/virtualenv.html?#virtualenv-generator):
* on Windows call `activate.bat`
* on Linux and MacOS call `source ./activate.sh`

***Note:*** To unload the virtual environment, call `deactivate.bat` on Windows or `source ./deactivate.sh` on Linux and MacOS.

### Create plugin with Rack SDK helper script
You can skip this step and use the [sources](https://github.com/qno/conan-vcvrack-sdk-plugin-example/archive/master.zip) from this repository.
* Change into the sources directory, e.g. `vcvrack-sdk-plugin-example` (make sure the virtual environment is still activated!)
* Execute the command `helper.py MyPlugin` and follow the instructions
* Copy the content from the generated `MyPlugin` folder one level up and delete the `MyPlugin` folder
* Download the [MyModule.svg](https://vcvrack.com/manual/_static/MyModule.svg) and put it into `res` folder
* Execute the command `helper.py createmodule MyModule res/MyModule.svg src/MyModule.cpp` and follow the instructions
* Edit the `src/MyModule.cpp` as described in the [tutorial](https://vcvrack.com/manual/PluginDevelopmentTutorial#implementing-the-dsp-kernel)
* Edit `src/plugin.hpp` and uncomment the line `extern Model* modelMyModule;`
* Edit `src/plugin.cpp` and uncomment the line `p->addModel(modelMyModule);`

### Build the plugin with the Rack SDK Makefile
Make sure the virtual environment is still activated and you are still in the sources directory!
* Execute `make` to build the plugin
* Execute `make install` to install the plugin

For testing start the Rack application and load the plugin.

It maybe seems to be a bit strange to have a build folder created which is not used when building the plugin with `make`,
but it is needed so Conan can install the dependencies and create the necessary scripts for the environment. Also,
by using the Rack SDK Makefile, it is only possible to build a plugin within the source folder.

### Using CMake for development

The advantage of using CMake for development is that a lot of C++ IDE's (Clion, QtCreator, VSCode etc.) have builtin
support for CMake and can directly open generated CMake projects.

* In the sources folder add a `CMakeLists.txt` file with the following content
(or just use the [sources](https://github.com/qno/conan-vcvrack-sdk-plugin-example/archive/master.zip) from this repository):
```
cmake_minimum_required(VERSION 3.10)

# set this to the plugin slug!
set(PLUGIN_NAME MyPlugin)

project(VCVRack${PLUGIN_NAME}Plugin)

set(CMAKE_CXX_STANDARD 14)

# Do not change the LIB_NAME!
set(LIB_NAME plugin)

include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
conan_basic_setup()

add_subdirectory(src)

file(GLOB LICENSE LICENSE*)
file(INSTALL plugin.json res ${LICENSE} DESTINATION dist/${PLUGIN_NAME})
install(TARGETS ${LIB_NAME} LIBRARY DESTINATION ${PROJECT_BINARY_DIR}/dist/${PLUGIN_NAME} OPTIONAL)
install(DIRECTORY ${PROJECT_BINARY_DIR}/dist/${PLUGIN_NAME}/ DESTINATION ${PLUGIN_NAME})
```
* In the src folder add a `CMakeLists.txt` file with the following content:
```
set(SOURCES plugin.cpp MyModule.cpp)

add_library(${LIB_NAME} MODULE ${SOURCES})

target_compile_options(${LIB_NAME} PRIVATE "-fuse-ld=gold" "-flto")
target_include_directories(${LIB_NAME} PRIVATE ${PROJECT_SOURCE_DIR}/include)
target_link_libraries(${LIB_NAME} ${CONAN_LIBS})

if (CMAKE_CXX_COMPILER_ID STREQUAL "AppleClang")
  set_target_properties(${LIB_NAME} PROPERTIES SUFFIX ".dylib")
endif ()

set_target_properties(${LIB_NAME} PROPERTIES PREFIX "")
```

### Build the plugin with CMake
* Change into the build directory, e.g. `vcvrack-sdk-plugin-example-build` (make sure the virtual environment is still activated!)
* Generate the project with CMake
  * On Windows execute `cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="%HOMEPATH%"\Documents\Rack\plugins-v1 ..\vcvrack-sdk-plugin-example`
  * On MacOS execute `cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$HOME/Documents/Rack/plugins-v1 ../vcvrack-sdk-plugin-example`
  * On Linux execute `cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$HOME/.Rack/plugins-v1 ../vcvrack-sdk-plugin-example`
* Build with `cmake --build . --target install` (or `ninja install`)

For testing start the Rack application and load the plugin.

***Note:*** If a plugin should be published in the [VCV Rack Library](https://library.vcvrack.com), it will be always
build with the VCV Rack Makefile based process. To make this work when using CMake, the Makefile of the plugin just
has to be updated as well by adding the required source files and include path setup.

## Developing a Rack plugin under Windows with Visual Studio using MinGW GCC

It is possible to use the MinGW GCC toolchain in Visual Studio for development by using the feature
[Open Folder](https://docs.microsoft.com/en-us/visualstudio/ide/develop-code-in-visual-studio-without-projects-or-solutions?view=vs-2019).
It requires the installation of a recent [Visual Studio community](https://visualstudio.microsoft.com).

### Create a CMakeSettings.json for Visual Studio

The conan-vcvrack-sdk recipe is capable of generating the required `CMakeSettings.json` automatically
by adding the `VSCMakeSettings` generators section to the `conanfile.txt`:
```
...
[generators]
cmake
virtualenv
VSCMakeSettings
...
```
As next it is required to extend the project `CMakeLists.txt` with a workaround, because Visual Studio is using a built-in
CMake program and therefor won't be able to create and read the required information from the`conanbuildinfo.cmake` file.

Replace the lines containing:
```
...
include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
conan_basic_setup()
...
```
with:
```
if (EXISTS ${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
  message(STATUS "conanbuildinfo.cmake detected, configure project with Conan")
  include(${CMAKE_BINARY_DIR}/conanbuildinfo.cmake)
  conan_basic_setup()
else ()
  message(STATUS "conanbuildinfo.cmake not detected, try to configure project with RACK_SDK variable")
  if (${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
    if ("${RACK_SDK}" STREQUAL "")
      message(FATAL_ERROR "Path to Rack SDK missing! Add -DRACK_SDK=<path to Rack SDK> to the cmake call.")
    else ()
      message(STATUS "Use Rack SDK: ${RACK_SDK}")
      include_directories("${RACK_SDK}/include" "${RACK_SDK}/dep/include")
      link_directories(${RACK_SDK})
      add_definitions(-DARCH_WIN -D_USE_MATH_DEFINES -march=nocona -funsafe-math-optimizations -fPIC
                      -Wsuggest-override -Wall -Wextra -Wno-unused-parameter)
      set (CONAN_LIBS "Rack")
      set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
    endif ()
  else ()
    message(FATAL_ERROR "Configure project with RACK_SDK variable is only supported on Windows platform!")
  endif ()
endif ()
```

### Install the project with Conan
* Change into the build directory, e.g. `vcvrack-sdk-plugin-example-build`
* Execute the command `conan install -pr mingw ..\vcvrack-sdk-plugin-example`
* Copy the generated `CMakeSettings.json` file into the sources folder

Now start Visual Studio IDE and open project by **Open Folder** and navigate to your plugin sources.

## CI with Github Actions

This repository contains a Github workflow definition to build the plugin.
On each push to the repo the plugin gets build for the Linux, MacOS and Windows platform.
The build result will be attached as artifact and can be accessed via the [Actions](https://github.com/qno/conan-vcvrack-sdk-plugin-example/actions) tab.
