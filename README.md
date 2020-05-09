# A usage example for creating a Rack plugin with [conan-vcvrack-sdk](https://github.com/qno/conan-vcvrack-sdk).

This example follows the [VCV Rack Plugin Development Tutorial](https://vcvrack.com/manual/PluginDevelopmentTutorial).

## Setup development environment

It is not necessary to setup the development environment for Rack, like described in the [tutorial](https://vcvrack.com/manual/Building#setting-up-your-development-environment), as this is handled by Conan package manager.
Besides the fact that you need to have a C++ compiler installed on Linux and MacOS.

### Follow the usage instructions from [conan-vcvrack-sdk](https://github.com/qno/conan-vcvrack-sdk#usage)

### additional setup for Windows

Rack plugins for Windows are compiled with MinGW GCC.

***Note:*** There is no need to have MSYS2 and MinGW toolchain installed. It will be completely handled by Conan!

* Create a [Conan profile](https://docs.conan.io/en/latest/reference/profiles.html) for MinGW under `<USER HOME>/.conan/profiles`, called e.g. `mingw`, with the following content:
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
* Execute the command `conan install ..\vcvrack-sdk-plugin-example` (on Windows execute `conan install -pr mingw ..\vcvrack-sdk-plugin-example`)

This will setup the development environment and install all required dependencies.

Now load the [generated virtual environment by Conan](https://docs.conan.io/en/latest/mastering/virtualenv.html?#virtualenv-generator):
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

### Using CMake for development
* In the sources folder add a main `CMakeLists.txt` file with the following content:
```
cmake_minimum_required(VERSION 3.10)

# set this to the plugin slug!
set(PLUGIN_NAME MyPlugin)

project(VCVRack${PLUGIN_NAME}Plugin)

set(CMAKE_CXX_STANDARD 14)

if ("${PLUGIN_SLUG}" STREQUAL "")
  message(WARNING "Plugin Slug is missing! Add -DPLUGIN_SLUG=<SLUG> to the cmake call.")
endif ()

if (NOT "${PLUGIN_NAME}" STREQUAL "${PLUGIN_SLUG}")
  message(WARNING "Plugin Slug '${PLUGIN_SLUG}' doesn't match defined PLUGIN_NAME variable '${PLUGIN_NAME}'")
endif ()

message(STATUS "Plugin Slug: '${PLUGIN_SLUG}'")

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
  * On Windows execute `cmake -G Ninja -DPLUGIN_SLUG=MyPlugin -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="%HOMEPATH%"\Documents\Rack\plugins-v1 ..\vcvrack-sdk-plugin-example`
  * On MacOS execute `cmake -G Ninja -DPLUGIN_SLUG=MyPlugin -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$HOME/Documents/Rack/plugins-v1 ../vcvrack-sdk-plugin-example`
  * On Linux execute `cmake -G Ninja -DPLUGIN_SLUG=MyPlugin -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$HOME/.Rack/plugins-v1 ../vcvrack-sdk-plugin-example`
* Build with `cmake --build . --target install` (or `ninja install`)

For testing start the Rack application and load the plugin.
