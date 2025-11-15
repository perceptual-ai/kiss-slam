cmake_minimum_required(VERSION 3.16...3.26)
project(kiss_slam VERSION 0.0.2 LANGUAGES CXX)

option(USE_SYSTEM_EIGEN3 "Use system pre-installed Eigen" ON)
option(USE_SYSTEM_TSL-ROBIN-MAP "Use system pre-installed tsl_robin" ON)
option(USE_SYSTEM_G2O "Use a pre-installed version of g2o" ON)

set(CMAKE_BUILD_TYPE Release)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

include(3rdparty/find_dependencies.cmake)
include(cmake/CompilerOptions.cmake)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/voxel_map)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/pgo)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/occupancy_mapper)

pybind11_add_module(kiss_slam_pybind MODULE kiss_slam_pybind.cpp)
target_link_libraries(kiss_slam_pybind PRIVATE pgo voxel_map Eigen3::Eigen occupancy_mapper)
target_compile_features(kiss_slam_pybind PUBLIC cxx_std_20)

# Set RPATH to look for libraries in the same directory as the module
set_target_properties(kiss_slam_pybind PROPERTIES
    INSTALL_RPATH "$ORIGIN"
    BUILD_RPATH "${CMAKE_BINARY_DIR}/_deps/g2o-build/lib"
)

# Install the compiled extension into the Python package source directory
install(TARGETS kiss_slam_pybind 
        DESTINATION ${CMAKE_SOURCE_DIR}/kiss_slam/kiss_slam_pybind)

# Copy g2o libraries to the install location
install(DIRECTORY ${CMAKE_BINARY_DIR}/_deps/g2o-build/lib/
    DESTINATION ${CMAKE_SOURCE_DIR}/kiss_slam/kiss_slam_pybind
    FILES_MATCHING PATTERN "libg2o*.so*"
)
