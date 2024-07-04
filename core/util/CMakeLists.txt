set(CMAKE_OSX_ARCHITECTURES arm64)
include(src/CMakeLists.txt)
add_library(utils ${util_srcs})
