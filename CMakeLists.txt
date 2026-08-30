cmake_minimum_required(VERSION 3.20)

project(keycastow
    VERSION 2.0.2.5
    DESCRIPTION "Keystroke visualizer for Windows"
    LANGUAGES CXX)

if(NOT WIN32)
    message(FATAL_ERROR "KeyCastOW is a Win32 application and only builds on Windows.")
endif()

if(NOT MSVC)
    message(FATAL_ERROR
        "KeyCastOW needs an MSVC-compatible toolchain (cl or clang-cl); "
        "the UTF-16 .rc file and #pragma comment(lib, ...) are MSVC-only.")
endif()

# Sources mirror keycastow.vcxproj: two translation units plus the resource script.
add_executable(keycastow WIN32
    keycast.cpp
    keylog.cpp
    keycastow.rc
    keycast.h
    keylog.h
    resource.h
    timer.h)

# <CharacterSet>Unicode</CharacterSet>
target_compile_definitions(keycastow PRIVATE UNICODE _UNICODE)

# DbgHelp is already pulled in by #pragma comment(lib, ...) in keycast.cpp, but
# naming it here keeps the link line self-describing.
target_link_libraries(keycastow PRIVATE gdiplus Comctl32 DbgHelp)

# Release used the static runtime so the .exe stays portable (see README).
set_property(TARGET keycastow PROPERTY
    MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

# The .vcxproj embedded keycastow.exe.manifest via the Manifest item; without it
# the common-controls v6 dependency is lost and the UI falls back to the old look.
target_link_options(keycastow PRIVATE
    "/MANIFEST:EMBED"
    "/MANIFESTINPUT:${CMAKE_CURRENT_SOURCE_DIR}/keycastow.exe.manifest")

# Debug info in both configurations, as the original project did.
target_link_options(keycastow PRIVATE "/DEBUG")
