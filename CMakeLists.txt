cmake_minimum_required(VERSION 3.25)
project(foxglove_sdk)

include(FetchContent)

# Download Foxglove SDK
# FetchContent_Declare(
#   foxglove
#   DOWNLOAD_EXTRACT_TIMESTAMP TRUE
#   URL https://github.com/foxglove/foxglove-sdk/releases/download/sdk%2Fv0.9.1/foxglove-v0.9.1-cpp-x86_64-unknown-linux-gnu.zip
#   URL_HASH SHA256=7a9463270fac3cdf99ca5d41e597cae41e2c8b59a48b9eafec7a9ad6a70b11b1
# )
FetchContent_Declare(
  foxglove
  DOWNLOAD_EXTRACT_TIMESTAMP TRUE
  URL https://github.com/jlack1987/foxglove-sdk.git
  GIT_TAG SHA256=4418b1b815ae352f1402fc4500e8e2c3a497e31e
)
FetchContent_MakeAvailable(foxglove)

# Gather headers and sources
file(GLOB_RECURSE FOXGLOVE_HEADERS "${foxglove_SOURCE_DIR}/include/*.h")
file(GLOB_RECURSE FOXGLOVE_SOURCES
    "${foxglove_SOURCE_DIR}/src/*.cpp"
    "${foxglove_SOURCE_DIR}/src/server/*.cpp"
)

# Define a library target
add_library(foxglove_sdk STATIC ${FOXGLOVE_SOURCES} ${FOXGLOVE_HEADERS})
target_include_directories(foxglove_sdk
  PUBLIC
    $<BUILD_INTERFACE:${foxglove_SOURCE_DIR}/include>
    $<BUILD_INTERFACE:${foxglove_SOURCE_DIR}/include/foxglove>
    $<INSTALL_INTERFACE:include>
)
target_link_libraries(foxglove_sdk PUBLIC ${foxglove_SOURCE_DIR}/lib/libfoxglove.a)

# Set export info
add_library(foxglove_sdk::foxglove_sdk ALIAS foxglove_sdk)

# Install rules
include(GNUInstallDirs)

install(TARGETS foxglove_sdk EXPORT foxglove_sdkTargets)
install(DIRECTORY ${foxglove_SOURCE_DIR}/include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/foxglove)

# Export the target
install(EXPORT foxglove_sdkTargets
  FILE foxglove_sdkTargets.cmake
  NAMESPACE foxglove_sdk::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/foxglove_sdk
)

# Create and install config file
configure_file(
  ${CMAKE_CURRENT_SOURCE_DIR}/cmake/foxglove_sdkConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/foxglove_sdkConfig.cmake
  @ONLY
)
install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/foxglove_sdkConfig.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/foxglove_sdk
)
