# ----------------------------------------------------------------------------
#  CMake file for Intel ITT API. See root CMakeLists.txt
#
# ----------------------------------------------------------------------------

if(NOT ITT_LIBRARY)
  set(ITT_LIBRARY "ittnotify")
endif()
project(${ITT_LIBRARY} C)

if(NOT WIN32)
  include(CheckLibraryExists)
  if(COMMAND CHECK_LIBRARY_EXISTS)
    CHECK_LIBRARY_EXISTS(dl dlerror "" HAVE_DL_LIBRARY)
  endif()
endif()

ocv_warnings_disable(CMAKE_C_FLAGS -Wimplicit-fallthrough)

ocv_include_directories("${CMAKE_CURRENT_SOURCE_DIR}/include")
set(ITT_INCLUDE_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include")

set(ITT_PUBLIC_HDRS
    include/ittnotify.h
    include/jitprofiling.h
    include/libittnotify.h
)
set(ITT_PRIVATE_HDRS
    src/ittnotify/disable_warnings.h
    src/ittnotify/ittnotify_config.h
    src/ittnotify/ittnotify_static.h
    src/ittnotify/ittnotify_types.h
)
set(ITT_SRCS
    src/ittnotify/ittnotify_static.c
    src/ittnotify/jitprofiling.c
)

add_library(${ITT_LIBRARY} STATIC ${OPENCV_3RDPARTY_EXCLUDE_FROM_ALL} ${ITT_SRCS} ${ITT_PUBLIC_HDRS} ${ITT_PRIVATE_HDRS})

file(STRINGS "src/ittnotify/ittnotify_config.h" API_VERSION_NUM REGEX "#define\[ \t]+API_VERSION_NUM[ \t]+([0-9\.]+)")
if(API_VERSION_NUM MATCHES "#define\[ \t]+API_VERSION_NUM[ \t]+([0-9\.]*)")
  set(ITTNOTIFY_VERSION "${CMAKE_MATCH_1}"  CACHE INTERNAL "" FORCE)
endif()

if(NOT WIN32)
  if(HAVE_DL_LIBRARY)
    target_link_libraries(${ITT_LIBRARY} dl)
  endif()
endif()

set_target_properties(${ITT_LIBRARY} PROPERTIES
        OUTPUT_NAME ${ITT_LIBRARY}
        DEBUG_POSTFIX "${OPENCV_DEBUG_POSTFIX}"
        COMPILE_PDB_NAME ${ITT_LIBRARY}
        COMPILE_PDB_NAME_DEBUG "${ITT_LIBRARY}${OPENCV_DEBUG_POSTFIX}"
        ARCHIVE_OUTPUT_DIRECTORY ${3P_LIBRARY_OUTPUT_PATH}
    )

ocv_warnings_disable(CMAKE_C_FLAGS -Wundef -Wsign-compare)
ocv_warnings_disable(CMAKE_C_FLAGS -Wstrict-prototypes) # clang15

if(ENABLE_SOLUTION_FOLDERS)
  set_target_properties(${ITT_LIBRARY} PROPERTIES FOLDER "3rdparty")
endif()

if(NOT BUILD_SHARED_LIBS)
  ocv_install_target(${ITT_LIBRARY} EXPORT OpenCVModules ARCHIVE DESTINATION ${OPENCV_3P_LIB_INSTALL_PATH} COMPONENT dev OPTIONAL)
endif()

ocv_install_3rdparty_licenses(ittnotify src/ittnotify/BSD-3-Clause.txt src/ittnotify/GPL-2.0-only.txt)
