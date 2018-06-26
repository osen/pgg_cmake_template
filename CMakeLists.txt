cmake_minimum_required(VERSION 2.6)
project(TRIANGLE)

#
# Obtain the "contrib" path. For example 'contrib/gnu-windows-x86'.
# This will be scanned for dependencies.
#
string(TOLOWER
  contrib/${CMAKE_CXX_COMPILER_ID}-${CMAKE_SYSTEM_NAME}-x86
  CONTRIB_PATH
)

#
# Recursively collect list of all directories within the contrib directory.
#
file(GLOB_RECURSE CONTRIB_PATHS LIST_DIRECTORIES true ${CONTRIB_PATH})

#
# Recursively collect a list of all shared objects within the contrib
# directory ready to be copied across as part of the post build task.
#
file(GLOB_RECURSE CONTRIB_SHARED_LIBRARIES
  ${CONTRIB_PATH}/*.so
  ${CONTRIB_PATH}/*.dll
)

#
# Set additional include directories to include all folders in contrib
# directory. This is a bit sloppy but easy. Using '*/include$' would be
# cleaner.
#
include_directories(
  ${CONTRIB_PATHS}
)

#
# Scan contrib paths for libraries that match the specified names. These
# will be used for linking.
#
find_library(LIBSDL2 SDL2 PATHS ${CONTRIB_PATHS})
find_library(LIBSDL2MAIN SDL2main PATHS ${CONTRIB_PATHS})
find_library(LIBSDL2IMAGE SDL2_image PATHS ${CONTRIB_PATHS})
find_library(LIBGLEW glew32 glew PATHS ${CONTRIB_PATHS})
find_library(LIBGL GL)

#
# Windows platform doesn't have a "standard path" so just hard code this one.
#
if(WIN32)
  set(LIBGL opengl32)
endif()

#
# Create an executable from the following source units.
#
add_executable(triangle
  src/main.cpp
)

#
# Link the executable against its dependent libraries.
#
target_link_libraries(triangle
  ${LIBSDL2} ${LIBSDL2MAIN} ${LIBSDL2IMAGE} ${LIBGLEW} ${LIBGL}
)

#
# Copy across all shared libraries from the contrib directory.
#
add_custom_command(TARGET triangle POST_BUILD
  COMMAND ${CMAKE_COMMAND} -E copy_if_different
    ${CONTRIB_SHARED_LIBRARIES}
    $<TARGET_FILE_DIR:triangle>
)
