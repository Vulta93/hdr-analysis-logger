cmake_minimum_required(VERSION 3.20)
project(hdr_analysis_logger LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
# Static runtime, so users don't need the Visual C++ redistributable.
set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

add_library(hdr_analysis_logger MODULE src/addon.cpp)
target_include_directories(hdr_analysis_logger SYSTEM PRIVATE
  ${CMAKE_CURRENT_SOURCE_DIR}/third_party/reshade
  ${CMAKE_CURRENT_SOURCE_DIR}/third_party/nlohmann)
target_compile_definitions(hdr_analysis_logger PRIVATE NOMINMAX)
if(MSVC)
  target_compile_options(hdr_analysis_logger PRIVATE /utf-8 /EHsc)
endif()

# ReShade loads add-ons with these extensions.
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  set(ADDON_SUFFIX ".addon64")
else()
  set(ADDON_SUFFIX ".addon32")
endif()
set_target_properties(hdr_analysis_logger PROPERTIES
  PREFIX ""
  OUTPUT_NAME "HDRAnalysisLogger"
  SUFFIX "${ADDON_SUFFIX}")
