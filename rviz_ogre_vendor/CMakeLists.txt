cmake_minimum_required(VERSION 3.10)
project(rviz_ogre_vendor)

find_package(ament_cmake REQUIRED)
find_package(ament_cmake_vendor_package REQUIRED)

if(WIN32)
  set(NOT_WIN32 FALSE)
else()
  set(NOT_WIN32 TRUE)
endif()

ament_vendor(freetype_vendor
  SATISFIED ${NOT_WIN32}
  VCS_TYPE tar
  VCS_URL https://git.savannah.gnu.org/cgit/freetype/freetype2.git/snapshot/freetype2-VER-2-6-5.tar.gz
  VCS_VERSION freetype2-VER-2-6-5
  CMAKE_ARGS
    -Wno-dev
    -DWITH_ZLIB=OFF
    -DWITH_BZip2=OFF
    -DWITH_PNG=OFF
    -DWITH_HarfBuzz=OFF
    -DBUILD_SHARED_LIBS:BOOL=ON
)

ament_vendor(zlib_vendor
  SATISFIED ${NOT_WIN32}
  VCS_TYPE tar
  VCS_URL https://www.zlib.net/fossils/zlib-1.2.11.tar.gz
  VCS_VERSION zlib-1.2.11
  CMAKE_ARGS
    -Wno-dev
    -UCMAKE_CXX_COMPILER
    # zlib doesn't use CMAKE_INSTALL_PREFIX correctly, so we need to override
    -DINSTALL_BIN_DIR=<INSTALL_DIR>/bin
    -DINSTALL_LIB_DIR=<INSTALL_DIR>/lib
    -DINSTALL_INC_DIR=<INSTALL_DIR>/include
    -DINSTALL_MAN_DIR=<INSTALL_DIR>/share/man
    -DINSTALL_PKGCONFIG_DIR=<INSTALL_DIR>/share/pkgconfig
)

set(OGRE_CXX_FLAGS ${CMAKE_CXX_FLAGS})
set(OGRE_CXX_FLAGS "-Wno-array-compare ${OGRE_CXX_FLAGS}")
set(OGRE_CXX_FLAGS "-Wno-dangling-pointer ${OGRE_CXX_FLAGS}")
set(OGRE_CXX_FLAGS "-Wno-deprecated-declarations ${OGRE_CXX_FLAGS}")
set(OGRE_CXX_FLAGS "-Wno-format-overflow ${OGRE_CXX_FLAGS}")
set(OGRE_CXX_FLAGS "-Wno-maybe-uninitialized ${OGRE_CXX_FLAGS}")
set(OGRE_CXX_FLAGS "-Wno-mismatched-new-delete ${OGRE_CXX_FLAGS}")
set(OGRE_CXX_FLAGS "-Wno-range-loop-construct ${OGRE_CXX_FLAGS}")
set(OGRE_CXX_FLAGS "-Wno-undef ${OGRE_CXX_FLAGS}")

ament_vendor(ogre_vendor
  VCS_TYPE zip
  VCS_URL https://github.com/OGRECave/ogre/archive/v1.12.10.zip
  VCS_VERSION ogre-1.12.10
  CMAKE_ARGS
    -DOGRE_CONFIG_ENABLE_ZIP:BOOL=ON
    -DOGRE_STATIC:BOOL=OFF
    -DOGRE_INSTALL_PDB:BOOL=OFF
    -DOGRE_BUILD_DEPENDENCIES:BOOL=OFF
    -DOGRE_BUILD_TESTS:BOOL=OFF
    -DOGRE_BUILD_SAMPLES:BOOL=FALSE
    -DOGRE_INSTALL_SAMPLES:BOOL=FALSE
    -DOGRE_INSTALL_SAMPLES_SOURCE:BOOL=FALSE
    -DOGRE_CONFIG_THREADS:STRING=0
    -DOGRE_RESOURCEMANAGER_STRICT:STRING=2
    -DCMAKE_SKIP_INSTALL_RPATH:BOOL=ON
    -DOGRE_BUILD_LIBS_AS_FRAMEWORKS:BOOL=OFF
    -DOGRE_BUILD_COMPONENT_PYTHON:BOOL=FALSE
    -DOGRE_BUILD_COMPONENT_JAVA:BOOL=FALSE
    -DOGRE_BUILD_COMPONENT_CSHARP:BOOL=FALSE
    -DOGRE_BUILD_COMPONENT_BITES:BOOL=FALSE
    -DOGRE_BUILD_PLUGIN_DOT_SCENE:BOOL=FALSE
    -DOGRE_BUILD_TOOLS:BOOL=FALSE
    -DCMAKE_POLICY_DEFAULT_CMP0074=NEW
    "-DCMAKE_CXX_FLAGS=${OGRE_CXX_FLAGS}"
    -Wno-dev
  PATCHES
    pragma-patch.diff
    relocatable.patch
    C4267.patch
)

if(TARGET ogre_vendor)
  if(TARGET freetype_vendor)
    ExternalProject_Add_StepDependencies(ogre_vendor configure freetype_vendor)
  endif()

  if(TARGET zlib_vendor)
    ExternalProject_Add_StepDependencies(ogre_vendor configure zlib_vendor)
  endif()
endif()

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package(
  CONFIG_EXTRAS "rviz_ogre_vendor-extras.cmake.in"
)
