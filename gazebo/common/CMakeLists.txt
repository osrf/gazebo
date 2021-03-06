include (${gazebo_cmake_dir}/GazeboUtils.cmake)

# tiny obj loader
include_directories(SYSTEM ${CMAKE_SOURCE_DIR}/deps/tinyobjloader)

if (HAVE_OPENAL)
  include_directories(${OPENAL_INCLUDE_DIR})
endif()

if (HAVE_GTS)
  include_directories(${gts_INCLUDE_DIRS})
  link_directories(${gts_LIBRARY_DIRS})
  add_definitions(${gts_CFLAGS})
endif()

if (HAVE_GDAL)
  include_directories(${GDAL_INCLUDE_DIR})
endif()

include_directories(${tinyxml_INCLUDE_DIRS})
link_directories(${tinyxml_LIBRARY_DIRS})

set (sources
  Animation.cc
  Assert.cc
  AudioDecoder.cc
  Battery.cc
  Base64.cc
  BVHLoader.cc
  ColladaExporter.cc
  ColladaLoader.cc
  CommonIface.cc
  Console.cc
  Dem.cc
  Event.cc
  Events.cc
  Exception.cc
  FuelModelDatabase.cc
  HeightmapData.cc
  Image.cc
  ImageHeightmap.cc
  KeyEvent.cc
  KeyFrame.cc
  Material.cc
  MaterialDensity.cc
  Mesh.cc
  MeshExporter.cc
  MeshLoader.cc
  MeshManager.cc
  ModelDatabase.cc
  MouseEvent.cc
  OBJLoader.cc
  PID.cc
  SdfFrameSemantics.cc
  SemanticVersion.cc
  SkeletonAnimation.cc
  Skeleton.cc
  SphericalCoordinates.cc
  STLLoader.cc
  SystemPaths.cc
  SVGLoader.cc
  Time.cc
  Timer.cc
  URI.cc
  Video.cc
  VideoEncoder.cc
  ffmpeg_inc.cc
)

if (NOT USE_EXTERNAL_TINYXML)
  include_directories (${CMAKE_SOURCE_DIR}/deps/win/tinyxml)
  add_library (tinyxml_internal STATIC
    ${CMAKE_SOURCE_DIR}/deps/win/tinyxml/tinystr.cpp
    ${CMAKE_SOURCE_DIR}/deps/win/tinyxml/tinyxml.cpp
    ${CMAKE_SOURCE_DIR}/deps/win/tinyxml/tinyxmlerror.cpp
    ${CMAKE_SOURCE_DIR}/deps/win/tinyxml/tinyxmlparser.cpp)
  set (tinyxml_LIBRARIES tinyxml_internal)
endif()

set (headers
  Animation.hh
  Assert.hh
  AudioDecoder.hh
  Battery.hh
  Base64.hh
  BVHLoader.hh
  ColladaLoader.hh
  CommonIface.hh
  CommonTypes.hh
  Console.hh
  Dem.hh
  EnumIface.hh
  Event.hh
  Events.hh
  Exception.hh
  FuelModelDatabase.hh
  MovingWindowFilter.hh
  HeightmapData.hh
  Image.hh
  ImageHeightmap.hh
  KeyEvent.hh
  KeyFrame.hh
  Material.hh
  MaterialDensity.hh
  Mesh.hh
  MeshLoader.hh
  MeshManager.hh
  ModelDatabase.hh
  MouseEvent.hh
  OBJLoader.hh
  PID.hh
  Plugin.hh
  SdfFrameSemantics.hh
  SemanticVersion.hh
  SkeletonAnimation.hh
  Skeleton.hh
  SingletonT.hh
  SphericalCoordinates.hh
  STLLoader.hh
  SystemPaths.hh
  SVGLoader.hh
  Time.hh
  Timer.hh
  UpdateInfo.hh
  URI.hh
  Video.hh
  VideoEncoder.hh
  WeakBind.hh
  ffmpeg_inc.h
 )

set (gtest_sources
  Animation_TEST.cc
  Battery_TEST.cc
  ColladaExporter_TEST.cc
  ColladaLoader_TEST.cc
  CommonIface_TEST.cc
  Console_TEST.cc
  Dem_TEST.cc
  EnumIface_TEST.cc
  Exception_TEST.cc
  Event_TEST.cc
  FuelModelDatabase_TEST.cc
  HeightmapData_TEST.cc
  Image_TEST.cc
  ImageHeightmap_TEST.cc
  Material_TEST.cc
  MaterialDensity_TEST.cc
  Mesh_TEST.cc
  MeshManager_TEST.cc
  MouseEvent_TEST.cc
  MovingWindowFilter_TEST.cc
  OBJLoader_TEST.cc
  Plugin_TEST.cc
  SemanticVersion_TEST.cc
  SphericalCoordinates_TEST.cc
  SystemPaths_TEST.cc
  SVGLoader_TEST.cc
  Time_TEST.cc
  URI_TEST.cc
  VideoEncoder_TEST.cc
  WeakBind_TEST.cc
)

# Timer test fails on OSX
# See issue #1626
if (NOT APPLE)
  set (gtest_sources
    ${gtest_sources}
    Timer_TEST.cc
  )
endif()

if (HAVE_GTS)
set (sources
  ${sources}
  GTSMeshUtils.cc
  MeshCSG.cc
)
set (headers
  ${headers}
  GTSMeshUtils.hh
  MeshCSG.hh
)

set (gtest_sources
  ${gtest_sources}
  GTSMeshUtils_TEST.cc
)
endif()

if (HAVE_FFMPEG)
  set (gtest_sources ${gtest_sources}
                     AudioDecoder_TEST.cc)
endif()

set (headers_install ${headers})

gz_build_tests(${gtest_sources} EXTRA_LIBS gazebo_common ${tinyxml_LIBRARIES})

set (common_headers "")
foreach (hdr ${headers_install})
  set (common_headers "${common_headers}#include \"gazebo/common/${hdr}\"\n")
endforeach()
configure_file (${CMAKE_CURRENT_SOURCE_DIR}/common.hh.in ${CMAKE_CURRENT_BINARY_DIR}/common.hh )

if (WIN32)
  # 0x601 = Windows 7.  TBB needs to know this.
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -D_WIN32_WINNT=0x601")
endif ()

gz_add_library(gazebo_common ${sources})

set_property(
  SOURCE SystemPaths.cc SystemPaths_TEST.cc ModelDatabase.cc Console.cc
  PROPERTY COMPILE_DEFINITIONS
  GAZEBO_PLUGIN_PATH="${GAZEBO_PLUGIN_PATH}"
  GAZEBO_MODEL_PATH="${GAZEBO_MODEL_PATH}"
  GAZEBO_RESOURCE_PATH="${GAZEBO_RESOURCE_PATH}"
  GAZEBO_MODEL_DATABASE_URI="${GAZEBO_MODEL_DATABASE_URI}"
  GAZEBO_DEFAULT_MASTER_PORT=${GAZEBO_DEFAULT_MASTER_PORT}
  OGRE_RESOURCE_PATH="${OGRE_RESOURCE_PATH}"
)

target_compile_definitions(gazebo_common
  PRIVATE BUILDING_DLL_GZ_COMMON
)

target_link_libraries(gazebo_common
  PUBLIC
    ${Boost_LIBRARIES}
    ${freeimage_LIBRARIES}
    ignition-common3::graphics
    ${IGNITION-FUEL_TOOLS_LIBRARIES}
    ${IGNITION-MATH_LIBRARIES}
    ${IGNITION-COMMON_LIBRARIES}
    ${libdl_library}
    ${libtar_LIBRARIES}
    ${SDFormat_LIBRARIES}
    ${tinyxml_LIBRARIES}
  PRIVATE
    CURL::libcurl
    ${libtool_library}
    ${libavcodec_LIBRARIES}
    ${libavformat_LIBRARIES}
    ${libavutil_LIBRARIES}
    ${libavdevice_LIBRARIES}
    ${libswscale_LIBRARIES}
    ${TBB_LIBRARIES}
)

if (UNIX)
  target_link_libraries(gazebo_common PRIVATE pthread)
endif()

if (UNIX AND NOT APPLE)
  # rt is used for clock_gettime, which is not available on apple or windows
  target_link_libraries(gazebo_common PRIVATE rt)
endif()

if (HAVE_GTS)
  target_link_libraries(gazebo_common PRIVATE ${gts_LIBRARIES})
endif()

if (HAVE_OPENAL)
  target_link_libraries(gazebo_common PRIVATE ${OPENAL_LIBRARY})
endif()

if (HAVE_GDAL)
  target_link_libraries(gazebo_common PRIVATE ${GDAL_LIBRARY})
endif()

if (USE_PCH)
  set(MORE_FLAGS "-pthread" ${Boost_PKGCONFIG_CFLAGS})
  add_pch(gazebo_common common_pch.hh ${MORE_FLAGS})
endif()


gz_install_library(gazebo_common)
gz_install_includes("common" ${headers_install} ${CMAKE_CURRENT_BINARY_DIR}/common.hh)
