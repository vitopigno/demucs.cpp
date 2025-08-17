cmake_minimum_required(VERSION 3.20)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

if (WIN32)
    set(CMAKE_CXX_FLAGS "/MD")

    set(CMAKE_CXX_FLAGS_DEBUG "/Zi")
    set(CMAKE_CXX_FLAGS_RELEASE "/O2 /fp:fast")
else()
    set(CMAKE_CXX_FLAGS "-Wall -Wextra")

    set(CMAKE_CXX_FLAGS_DEBUG "-g -DEIGEN_FAST_MATH=0 -O0")
    set(CMAKE_CXX_FLAGS_RELEASE "-Ofast -march=native -fno-unsafe-math-optimizations -freciprocal-math -fno-signed-zeros")
endif()

# define a macro NDEBUG for Eigen3 release builds
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -DNDEBUG")

project(demucs.cpp)

# set C++ standard to C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# add openmp support to regular eigen
find_package(OpenMP REQUIRED)
if(OPENMP_FOUND)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
    include_directories(${OpenMP_CXX_INCLUDE_DIRS})
endif()

if(OPENMP_FOUND)
    set(LIBRARIES_TO_LINK ${OpenMP_CXX_LIBRARIES})
endif()

# include src/*.cpp and src/*.c as source files
file(GLOB SOURCES "src/*.cpp")

add_library(demucs.cpp.lib ${SOURCES})

# add library Eigen3
# expose source files to project linking to this one
target_include_directories(demucs.cpp.lib PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/vendor/eigen
    ${CMAKE_CURRENT_SOURCE_DIR}/src
)

target_link_libraries(demucs.cpp.lib ${LIBRARIES_TO_LINK})

# compile vendored submodule libnyquist for driver programs
set(LIBNYQUIST_BUILD_EXAMPLE OFF CACHE BOOL "Disable libnyquist example" FORCE)
add_subdirectory(vendor/libnyquist)

add_executable(demucs_mt.cpp.main "cli-apps/demucs_mt.cpp")
target_include_directories(demucs_mt.cpp.main PRIVATE vendor/libnyquist/include)
target_include_directories(demucs_mt.cpp.main PRIVATE cli-apps)
target_link_libraries(demucs_mt.cpp.main demucs.cpp.lib libnyquist)
