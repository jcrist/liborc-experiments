cmake_minimum_required(VERSION 2.8)

include(ExternalProject)

# Include liborc
set (ORC_PREFIX "${CMAKE_BINARY_DIR}/libs/orc_ep")
set (ORC_CMAKE_ARGS -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
                    -DCMAKE_INSTALL_PREFIX=${ORC_PREFIX}
                    -DBUILD_LIBHDFSPP=OFF
                    -DBUILD_JAVA=OFF)

ExternalProject_Add (orc_ep
  GIT_REPOSITORY "https://github.com/jcrist/orc.git"
  GIT_TAG "fix-install-includes"
  LOG_DOWNLOAD 1
  LOG_CONFIGURE 1
  LOG_BUILD 1
  LOG_INSTALL 1
  CMAKE_ARGS ${ORC_CMAKE_ARGS})

add_library (liborc STATIC IMPORTED)
set_target_properties (liborc PROPERTIES IMPORTED_LOCATION "${ORC_PREFIX}/lib/liborc.a")
add_dependencies (liborc orc_ep)
include_directories (SYSTEM "${ORC_PREFIX}/include")
# All the built libs needed to link with to use liborc
set (LIBORC_LIBS
     liborc
     "${ORC_PREFIX}/lib/liblz4.a"
     "${ORC_PREFIX}/lib/libprotobuf.a"
     "${ORC_PREFIX}/lib/libsnappy.a"
     "${ORC_PREFIX}/lib/libz.a")


# -- Compiler Flags --
set(CMAKE_CXX_FLAGS_DEBUG "-O0 -g")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O3 -g -DNDEBUG")
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -DNDEBUG")
message(STATUS "compiler ${CMAKE_CXX_COMPILER_ID} version ${CMAKE_CXX_COMPILER_VERSION}")
if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  set (CXX11_FLAGS "-std=c++11")
  set (WARN_FLAGS "-Weverything -Wno-c++98-compat -Wno-missing-prototypes")
  set (WARN_FLAGS "${WARN_FLAGS} -Wno-c++98-compat-pedantic -Wno-padded")
  set (WARN_FLAGS "${WARN_FLAGS} -Wno-covered-switch-default")
  set (WARN_FLAGS "${WARN_FLAGS} -Wno-missing-noreturn -Wno-unknown-pragmas")
  set (WARN_FLAGS "${WARN_FLAGS} -Wno-gnu-zero-variadic-macro-arguments")
  set (WARN_FLAGS "${WARN_FLAGS} -Wconversion -Werror")
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  set (WARN_FLAGS "-Wall -Wno-unknown-pragmas -Wconversion -Werror")
  if (CMAKE_CXX_COMPILER_VERSION STREQUAL "" OR
      CMAKE_CXX_COMPILER_VERSION VERSION_LESS "4.7")
    set (CXX11_FLAGS "-std=c++0x")
  else ()
    set (CXX11_FLAGS "-std=c++11")
  endif ()
elseif (MSVC)
  add_definitions (-D_SCL_SECURE_NO_WARNINGS)
  add_definitions (-D_CRT_SECURE_NO_WARNINGS)
  # TODO: We assume MSVC debug mode. In the future, set these flags
  # appropriately for all build mode.
  set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
  set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} /MTd")
  set(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /NODEFAULTLIB:libc.lib")
  set(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /NODEFAULTLIB:libcmt.lib")
  set(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /NODEFAULTLIB:msvcrt.lib")
  set(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /NODEFAULTLIB:libcd.lib")

  set(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS_DEBUG} /NODEFAULTLIB:msvcrtd.lib")
endif ()

set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CXX11_FLAGS} ${WARN_FLAGS}")

add_subdirectory(pyorc)
