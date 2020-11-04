cmake_minimum_required(VERSION 3.16...3.21)

project(obs-asio VERSION 3.1.0)
add_library(obs-asio MODULE)

set(PLUGIN_AUTHOR "Andersama & pkv")

include(FetchContent)
FetchContent_Declare(
  BASS_ASIO
  URL https://www.un4seen.com/files/bassasio14.zip
  URL_HASH MD5=825182f314d1b925f1cc42647b4ec117)

FetchContent_MakeAvailable(BASS_ASIO)

find_package(libobs REQUIRED)
find_package(obs-frontend-api REQUIRED)
include(cmake/ObsPluginHelpers.cmake)
find_qt(COMPONENTS Widgets Core)

# ##################################################################################################
# QT support                                                                                       #
# ##################################################################################################
set(CMAKE_PREFIX_PATH "${QTDIR}")
set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)

# ##################################################################################################
# set architecture                                                                                 #
# ##################################################################################################
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(ARCH 64)
else()
  set(ARCH 32)
endif()

if(ARCH EQUAL 64)
  set(OBS_ARCH_NAME "64bit")
  set(OBS_BUILDDIR_ARCH "build64")
else()
  set(OBS_ARCH_NAME "32bit")
  set(OBS_BUILDDIR_ARCH "build32")
endif()

target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE ${bass_asio_SOURCE_DIR}/c)

target_sources(${CMAKE_PROJECT_NAME} PRIVATE src/asio-input.cpp)

target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE OBS::libobs OBS::obs-frontend-api Qt::Core
                                                    Qt::Widgets)

if(OS_WINDOWS)
  if(ARCH EQUAL 64)
    target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE ${bass_asio_SOURCE_DIR}/c/x64/bassasio.lib)
  else()
    target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE ${bass_asio_SOURCE_DIR}/c/bassasio.lib)
  endif()
endif()

if(MSVC)
  target_include_directories(${CMAKE_PROJECT_NAME}
                             PRIVATE ${CMAKE_SOURCE_DIR}/../obs-studio/deps/w32-pthreads/)
  target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE OBS::w32-pthreads)
endif()

# --- Windows-specific build settings and tasks ---
if(OS_WINDOWS)
  configure_file(cmake/bundle/windows/installer-Windows.iss.in
                 ${CMAKE_BINARY_DIR}/installer-Windows.generated.iss)

  configure_file(cmake/bundle/windows/resource.rc.in ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.rc)
  target_sources(${CMAKE_PROJECT_NAME} PRIVATE ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.rc)

  if(MSVC)
    target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE /WX-)
  endif()
  # --- End of section ---
endif()

set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES FOLDER "plugins")
setup_plugin_target(${CMAKE_PROJECT_NAME})
