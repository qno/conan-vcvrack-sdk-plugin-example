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
