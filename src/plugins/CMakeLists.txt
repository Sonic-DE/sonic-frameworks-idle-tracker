if(APPLE)
    add_subdirectory(osx)
endif()

if(WIN32)
    add_subdirectory(windows)
endif()

# Linux/FreeBSD: XSync is required.
if(CMAKE_SYSTEM_NAME STREQUAL "Linux" OR CMAKE_SYSTEM_NAME STREQUAL "FreeBSD")
    add_subdirectory(xsync)
endif()
